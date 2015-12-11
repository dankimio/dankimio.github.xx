---
layout: post
title: Mutual Friendship in Rails
description: Add Facebook-like friendships to your Rails app.
---

There're many ways of implementing a friendship model in Rails. Creating Twitter-like subscription model, where distinction between following and followers is clear, is easy enough. Mutual friendship (Facebook) model requires a bit more code. In this case friends are added only after user's approval:

1. `Foo` sends a friend request to `Bar`.
2. `Bar` accepts the friend request from `Foo`.
3. `Foo` and `Bar` are now friends.

We'll have two join models: `FriendRequest` and `Friendship`, each will have its own controller with RESTful routes.

## Friend requests
First, we'll create `FriendRequest` resource.

```bash
$ rails generate resource FriendRequest user:references friend:references
$ rake db:migrate
```

Then, we'll establish `has_many :through` association between users. You can read more about associations [here](http://guides.rubyonrails.org/association_basics.html#the-has-one-through-association).

```ruby
# app/models/user.rb:
class User < ActiveRecord::Base
  has_many :friend_requests, dependent: :destroy
  has_many :pending_friends, through: :friend_requests, source: :friend
end
```

We're using `source: :friend` because we've changed the association name (should've been `friends`, but it is reserved for "original" friends).

Because the FriendRequest model has self-referential association, we have to specify the class name:

```ruby
class FriendRequest < ActiveRecord::Base
  belongs_to :user
  belongs_to :friend, class_name: 'User'
end
```

The corresponding controller will have four actions:

1. `index` — view incoming and outgoing friend requests
2. `create` — send a friend request to another user
3. `update` — to accept friend requests
4. `destroy` — to decline friend requests

##### Sending friend requests
To send a request we'll create a new `FriendRequest` record with our current user as a friend:

```ruby
# app/controllers/friend_requests_controller.rb
class FriendRequestsController < ApplicationController
  before_action :set_friend_request, except: [:index, :create]

  def create
    friend = User.find(params[:friend_id])
    @friend_request = current_user.friend_requests.new(friend: friend)

    if @friend_request.save
      render :show, status: :created, location: @friend_request
    else
      render json: @friend_request.errors, status: :unprocessable_entity
    end
  end
  ...
  private

  def set_friend_request
    @friend_request = FriendRequest.find(params[:id])
  end
end
```

##### Gettings requests
Here, we're simply grabbing all incoming and outgoing requests.

```ruby
def index
  @incoming = FriendRequest.where(friend: current_user)
  @outgoing = current_user.friend_requests
end
...
```

##### Canceling requests
Canceling a request is equivalent to destroying a friend request record.

```ruby
def destroy
  @friend_request.destroy
  head :no_content
end
```

## Friendships
Now let's move on to another join model: `Friendship`. We'll be using this association to get and destroy friends.

```bash
$ rails g model Friendship user:references friend:references
$ rails g controller Friends index destroy
$ rake db:migrate
```

First, we must set up an association:

```ruby
# app/models/friendship.rb
class User < ActiveRecord::Base
  ...
  has_many :friendships, dependent: :destroy
  has_many :friends, through: :friendships
end
```

Friendship model looks like this:

```ruby
class Friendship < ActiveRecord::Base
  after_create :create_inverse_relationship
  after_destroy :destroy_inverse_relationship

  belongs_to :user
  belongs_to :friend, class_name: 'User'

  private

  def create_inverse_relationship
    friend.friendships.create(friend: user)
  end

  def destroy_inverse_relationship
    friendship = friend.friendships.find_by(friend: user)
    friendship.destroy if friendship
  end
end
```

After it, we'll be able to query user records for friends:

```ruby
User.first.friends # => [Users]
```

##### Accepting requests
Next, we'll set up some ancillary methods in `FriendRequest` model:

```ruby
# app/models/friend_request.rb
class FriendRequest < ActiveRecord::Base
  ...
  # This method will build the actual association and destroy the request
  def accept
    user.friends << friend
    destroy
  end
end
```

Thanks to `has_many :through` association we're now able to add friends using `append (<<)` method.

Now it takes just one line of controller code to accept a friend request:

```ruby
# app/controllers/friend_requests
def update
  @friend_request.accept
  head :no_content
end
```

##### Listing all friends
The code for `Friends` controller is straightforward:

```ruby
# app/controllers/friends_controller.rb
class FriendsController < ApplicationController
  before_action :set_friend, only: :destroy

  def index
    @friends = current_user.friends
  end
  ...
  private

  def set_friend
    @friend = current_user.friends.find(params[:id])
  end
end
```

##### Removing friends
Unfriending users is a bit complicated since we have to remove friendships from the both sides of association. We'll write a separate method that will take this responsibility.

```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  ...
  def remove_friend(friend)
    current_user.friends.destroy(friend)
  end
end
```

Now we're able to use this method in our controller:

```ruby
# app/controllers/friends_controller.rb
def destroy
  current_user.remove_friend(@friend)
  head :no_content
end
```

## Final steps
You might want to add some validations to the models (uniqueness and presence):

```ruby
validates :user, presence: true
validates :friend, presence: true, uniqueness: { scope: :user }
```

It's also a good idea to restrict self associations, so the user won't be able to befriend himself. This validation should go into both classes (`Friendship` and `FriendRequest`).

```ruby
validate :not_self

private

def not_self
  errors.add(:friend, "can't be equal to user") if user == friend
end
```

And users shouldn't be able to send friend requests if it already exists or if they're already friends:

```ruby
# app/models/friend_request
class FriendRequest < ActiveRecord::Base
  ...
  validate :not_friends
  validate :not_pending
  ...
  private

  def not_friends
    errors.add(:friend, 'is already added') if user.friends.include?(friend)
  end

  def not_pending
    errors.add(:friend, 'already requested friendship') if friend.pending_friends.include?(user)
  end
end
```
