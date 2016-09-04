---
layout: post
title: How to set up admin area in Rails with Devise and Bootstrap
description: Build an admin area from scratch.
---

While there are a lot of solutions to this problem already ([RailsAdmin](https://github.com/sferik/rails_admin) and [ActiveAdmin](https://github.com/activeadmin/activeadmin)), they offer too much: theming, exporting, history and etc. They also introduce a lot of dependencies.

## What features should it have
Our goal is to build a simple and good-looking admin dashboard without making our structure and design too complex.

The basic features that our dashboard will have:

1. Single-user system: no registrations, new admins can be added via admin interface
2. Bootstrap views: Bootstrap adds simplicity and look & feel to the interface
3. Pagination: show only a limited number of records and with page switcher in the bottom

## Setting up a project
We will build a blog app that will have only two resources: posts and categories.

```bash
$ rails new blog
$ cd blog
```

### Generating models
Next, we'll have to scaffold our models and run migrations:

```
$ rails generate scaffold category name
$ rails generate scaffold post title content:text category:belongs_to
$ rake db:migrate
```

I will add some basic validations:

```ruby
# app/models/post.rb
class Post < ActiveRecord::Base
  belongs_to :category

  validates :title, presence: true
  validates :content, presence: true
  validates :category, presence: true
end
```

```ruby
# app/models/category.rb
class Category < ActiveRecord::Base
  validates :name, presence: true
end
```

## Devise

### Installation
Then, we will add Devise:

```ruby
# Gemfile
gem 'devise'
```

And install it:

```bash
$ bundle install
```

```bash
$ rails generate devise:install
```

Let's route our root path to posts:

```ruby
# config/routes.rb
root 'posts#index'
```

And then generate `Admin` model and run migrations:

```bash
$ rails generate devise Admin
$ rake db:migrate
```

This will create `admins` table in our database. You can also modify the columns in the migration that was created by this generator.

### Configuration
If you open admin model, you'll see all Devise modules that are enabled by default:

```ruby
class Admin < ActiveRecord::Base
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable
end
```

All we need to do here is to remove `:registerable` module from the class to forbid all unwanted sign ups.

```ruby
class Admin < ActiveRecord::Base
  devise :database_authenticatable, :recoverable, :rememberable, :trackable, :validatable
end
```

From now on, Devise will hide all sign up views and routes.

Because we will have resources in both user area and admin area, we'll have to namespace admin area. To do this, we need to pass additional params to devise route:

```ruby
# config/routes.rb
devise_for :admins, path: 'admin', skip: :registrations
```

Admin area will be namespaced in URLs: `/admin/sign_in`, `/admin/posts`. This is how our sign in page looks like:

![Admin sign in page](/images/2015/11/Admin-sign-in.png)

### Admin resources
Now we can add categories and posts to admin dashboard. To do that we'll create namespaced controllers, they will be separate from those that were created by scaffold command.

We can use the default generator. I prefer to skip JSON formatters in controllers for admin area and to do that we just need to comment out `jbuilder` gem from the Gemfile, and we can uncomment it later.

```ruby
# gem 'jbuilder', '~> 2.0'
```

```bash
$ bundle install
```

We already have models for posts and categories, and we can skip models when generating resources by using scaffold controller which will generate only controllers and views.
```bash
$ rails generate scaffold_controller admin/posts
```

This is the output of scaffold controller. We can see, that there were no models and migrations created:

```bash
create  app/controllers/admin/posts_controller.rb
invoke  erb
create    app/views/admin/posts
create    app/views/admin/posts/index.html.erb
create    app/views/admin/posts/edit.html.erb
create    app/views/admin/posts/show.html.erb
create    app/views/admin/posts/new.html.erb
create    app/views/admin/posts/_form.html.erb
```

The are a few things that we need to change in these controllers:

1. Remove namespace for models: `Admin::Post` to `Post` and the same for categories
2. Change redirect statements: `redirect_to [:admin, @admin_post]`, without it admins would have been redirected outside dashboard
3. Don't forget to update strong parameters:

```ruby
def admin_post_params
  params.require(:post).permit(:title, :content, :category_id)
end
```

Now we can add admin resources to routes and map `/admin` path to some page like posts index.

```ruby
# config/routes.rb
namespace :admin do
  resources :posts
  resources :categories
end

get 'admin' => 'admin/posts#index'
```

Let's also generate views and controllers for categories:

```bash
$ rails g scaffold_controller admin/categories
```

At this moment all admin views will look the same as previously generated ones. It's time to run our server and see all the changes we've made:

```bash
$ rails server
```

![List of posts](/images/2015/11/List-of-posts.png)

## Authentication
Currently, everyone can visit admin area, so let's enforce authentication for dashboard. It's a good idea to create `AdminController` that all other admin controllers will inherit:

```ruby
# app/controllers/admin/admin_controller.rb
class Admin::AdminController < ActionController::Base
  before_action :authenticate_admin!
  layout 'admin'
end
```

Inherit all admin controllers from `Admin::AdminController`:

```ruby
class Admin::PostsController < Admin::AdminController
end

class Admin::CategoriesController < Admin::AdminController
end
```

Now all actions in admin controllers will require authentication. `authenticate_admin!` is a Devise method, that will check if the current user is logged in and redirect to sign in page otherwise.

![Sign in page](/images/2015/11/Admin-sign-in-1.png)

## Views
New post form looks like this:

![New post form](/images/2015/11/Screen-Shot-2015-11-01-at-5-13-59-PM.png)

We'll add fields to our form and make sure they look good with Bootstrap.

### Bootstrap
Add Bootstrap to the Gemfile and run `bundle install`.

```ruby
# Gemfile
gem 'bootstrap-sass'
```

Next, we need to apply Bootstrap to our layouts. Bootstrap should be imported before all other CSS assets.

```bash
$ cd app/assets/stylesheets
$ mv application.css application.scss
```

We will remove `require self` and `require_tree .` statements and import Bootstrap via SCSS's `@import` statement. This is why we removed `require`:

> Do not use \*= require in Sass or your other stylesheets will not be able to access the Bootstrap mixins or variables.

```scss
// app/assets/stylesheets/application.scss
@import 'bootstrap-sprockets';
@import 'bootstrap';
```

Now let's import Bootstrap in Javascript, it should be imported after jQuery:

```js
// app/assets/javascripts/application.js
//= require jquery
//= require jquery_ujs
//= require bootstrap-sprockets
//= require turbolinks
//= require_tree .
```

We can verify that everything was installed correctly by visiting our admin posts page again. To see the changes we need to restart the web server.

![Admin sign in page with Bootstrap](/images/2015/11/Screen-Shot-2015-11-01-at-5-32-34-PM.png)

Let's create the first admin to seeds, so that it exists after we set up the database:

```ruby
# db/seeds.rb
Admin.create!(email: 'foo@bar.com', password: 'password')
```

Now, seed the database:

```bash
$ rake db:seed
```

We can now log in to the admin area with this admin account:

![Signed in successfully](/images/2015/11/Screen-Shot-2015-11-01-at-5-39-59-PM.png)

We want to keep our admin and application layouts separated. Admin dashboard will have its own design and will not be related to the main layout. We will create a layout file and tell `AdminController` to use the new layout.

```html
<#% app/views/layouts/admin.erb %>
<!DOCTYPE html>
<html>
<head>
  <title>Blog Admin</title>

  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>
</head>
<body>

<%= render 'layouts/header' %>

<div class="container">
  <% flash.each do |key, value| %>
    <div class="alert alert-<%= key %>">
      <%= value %>
    </div>
  <% end %>

  <%= yield %>
</div>

<%= render 'layouts/footer' %>

</body>
</html>
```

Header and footer partials:

```html
<%# app/views/layouts/_header.erb %>
<nav class="navbar navbar-default navbar-static-top">
  <div class="container-fluid">
    <div class="navbar-header">
      <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
        <span class="sr-only">Toggle navigation</span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
      </button>
      <%= link_to 'Admin', admin_path, class: 'navbar-brand' %>
    </div>

    <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
      <ul class="nav navbar-nav">
        <li class="dropdown">
          <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">Sections <span class="caret"></span></a>
          <ul class="dropdown-menu">
            <li><%= link_to 'Posts', admin_posts_path %></li>
            <li><%= link_to 'Categories', admin_categories_path %></li>
          </ul>
        </li>
      </ul>
      <ul class="nav navbar-nav navbar-right">
        <% if admin_signed_in? %>
          <li><%= link_to 'Sign out', destroy_admin_session_path, method: :delete %></li>
        <% else %>
          <li><%= link_to 'Sign in', new_admin_session_path %></li>
        <% end %>
      </ul>
    </div>
  </div>
</nav>
```

```html
<%# app/views/layouts/_footer.erb %>
<footer class="footer">
  <div class="container">
    <p class="text-muted">&copy; 2015 Awesome Blog</p>
  </div>
</footer>
```

This layout includes meta tags required by Bootstrap to preserve compatibility and to enable responsiveness on mobile devices. It is also important to have a container for flash messages, this container uses `alert` Bootstrap class. Footer requires additional CSS code and to accomplish this we will create a separate stylesheet file that will be imported in `application.scss` file. Notice that I have also added code for flash alerts to match Bootstrap classes. For example, notice alert will have `alert-info` class.

```scss
// app/assets/stylesheets/custom.scss
// Sticky footer styles
html {
  position: relative;
  min-height: 100%;
}

body {
  // Margin bottom by footer height
  margin-bottom: 60px;
}

.footer {
  position: absolute;
  bottom: 0;
  width: 100%;
  // Set the fixed height of the footer here
  height: 60px;
  background-color: #f5f5f5;
}

// Custom page CSS. Not required for template or sticky footer method.
.container .text-muted {
  margin: 20px 0;
}

// Alerts
.alert-notice {
  @extend .alert-info !optional;
}

.alert-alert {
  @extend .alert-danger !optional;
}
```

Import new styles in the main file:

```scss
// app/assets/stylesheets/application.scss
@import 'bootstrap-sprockets';
@import 'bootstrap';
@import 'custom';
```

Now we can tell `ApplicationController` to use the new layout if in admin area:

```ruby
class ApplicationController < ActionController::Base
  layout :set_layout

  private

  def set_layout
    devise_controller? ? 'admin' : false
  end
end
```

This is how admin area looks at the moment:

![Admin dashboard with Bootstrap](/images/2015/11/Screen-Shot-2015-11-01-at-6-25-19-PM.png)

### Bootstrap for Devise views
Dashboard looks better now, but Devise views still don't have Bootstrap styles. We could override Devise views and customize them as we want to, but there's [devise-bootstrap-views](https://github.com/hisea/devise-bootstrap-views) gem that does the same thing.

```ruby
# Gemfile
gem 'devise-bootstrap-views'
```

```bash
$ bundle install
```

Then we need to import devise views styles. At the very bottom of `application.scss` before importing `custom`:

```scss
@import 'bootstrap-sprockets';
@import 'bootstrap';
@import 'devise_bootstrap_views';
@import 'custom';
```

That's it! This is how the sign in form looks like now:

![Sign in form with Bootstrap views](/images/2015/11/Screen-Shot-2015-11-01-at-6-41-44-PM.png)

### Customizing index
First, we need to remove `<%= notice %>` code from all index pages, because it is already included in the layout.

```html
<%# app/views/posts/index.html.erb %>
<h1>Posts</h1>

<div class="form-group">
  <%= link_to 'New Post', new_admin_post_path, class: 'btn btn-default' %>
</div>

<table class="table">
  <thead>
    <tr>
      <th>Title</th>
      <th>Content</th>
      <th>Category</th>
      <th></th>
    </tr>
  </thead>

  <tbody>
    <% @admin_posts.each do |post| %>
      <tr>
        <td><%= link_to post.title, [:admin, post] %></td>
        <td><%= post.content %></td>
        <td><%= post.category.name %></td>
        <td>
          <%= link_to 'Edit', edit_admin_post_path(post), class: 'btn btn-default' %>
          <%= link_to 'Delete', [:admin, post], method: :delete, data: { confirm: 'Are you sure?' }, class: 'btn btn-danger' %>
        </td>
      </tr>
    <% end %>
  </tbody>
</table>
```

The new index page:
![The new index page](/images/2015/11/Screen-Shot-2015-11-01-at-6-54-48-PM.png)

Next, we'll update the form. Notice that the argument for `form_for` method is an array, `:admin` specifies the namespace of the controller that will receive the form data.

```html
<%# app/views/posts/_form.html.erb %>
<%= form_for([:admin, @admin_post]) do |f| %>
  <% if @admin_post.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@admin_post.errors.count, "error") %> prohibited this user from being saved:</h2>

      <ul>
      <% @admin_post.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

  <div class="form-group">
    <%= f.label :title %>
    <%= f.text_field :title, class: 'form-control' %>
  </div>

  <div class="form-group">
    <%= f.label :content %>
    <%= f.text_area :content, class: 'form-control' %>
  </div>

  <div class="form-group">
    <%= f.label :category %>
    <%= f.collection_select(:id, Category.all, :id, :name, {}, class: 'form-control') %>
  </div>

  <div class="form-group">
    <%= f.submit class: 'btn btn-default' %>
  </div>
<% end %>
```

Now let's update and clean up `new.html.erb`, `edit.html.erb` and `show.html.erb`. I prefer not to have `Edit` and `Back` buttons that were generated by Rails:

```html
<%# app/views/posts/new.html.erb %>
<h1>New Post</h1>

<%= render 'form' %>
```

```html
<%# app/views/posts/edit.html.erb %>
<h1>Editing Post</h1>

<%= render 'form' %>
```

```html
<%# app/views/posts/show.html.erb %>
<div class="form-group">
  <strong>Title:</strong> <%= @admin_post.title %>
</div>

<div class="form-group">
  <strong>Content:</strong> <%= @admin_post.content %>
</div>

<div class="form-group">
  <strong>Category:</strong> <%= @admin_post.category.name %>
</div>
```

![The new show page](/images/2015/11/Screen-Shot-2015-11-01-at-7-20-55-PM.png)

The steps for categories resource are the same. Dashboard looks much better now.

## Pagination
It's time for pagination, I will use Kaminari for this purpose, but you can use `will_paginate` as well. Both gems have Bootstrap styles that can be installed with just one generator.

### Seeds
For testing purposes I'll create a lot of posts in Rails console.

```ruby
100.times { |n| Post.create(title: "Post #{n}", content: "Lorem ipsum dolor sit amet", category: Category.first) }
```

### Install and set up Kaminari
```ruby
# Gemfile
gem 'kaminari'
```

```bash
$ bundle install
```

Then, we need to add append `.page()` method to our ActiveRecord relations:
```ruby
# app/controllers/admin/posts_controller.rb
def index
  @admin_posts = Post.all.page(params[:page])
end
```

In our views we'll add a block for pagination. This will render Kaminari views.

```html
<%# app/views/posts/index.html.erb %>
<%= paginate @admin_posts %>
```

Kaminari comes without styles. But Kaminari has themes (including one for Bootstrap) that are available via generator. You can find the list of all themes [here](https://github.com/amatsuda/kaminari_themes).

```bash
$ rails g kaminari:views bootstrap3
```

Pagination now has Bootstrap styles applied:

![Pagination with Bootstrap](/images/2015/11/Screen-Shot-2015-11-01-at-7-36-56-PM.png)

### Other possible features
1. i18n
2. Search
3. Extracting to gem
