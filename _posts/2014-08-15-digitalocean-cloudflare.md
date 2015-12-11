---
layout: post
title: Set up CloudFlare with DigitalOcean
description: Use CloudFlare to manage your domains.
---

DigitalOcean tutorials recommend using their own DNS management system, but it's not necessary. It is possible to attach domain to your droplet using your own DNS provider without changing NS servers to DigitalOcean's. I'm using CloudFlare, but you can use your own DNS provider.

1. First of all, you need to point your domain to your DNS provider. (You can point them to CloudFlare or use your provider's DNS system)
2. In your DNS settings create an A record for the root domain and a wildcard. It should look like this: ![](/images/2014/08/digitalocean-cloudflare.png)
3. Save your settings and wait some time  until your changes are propagated.

Now you can use your droplet without using DigitalOcean's DNS. You have to disable CloudFlare protection (cloud icon in DNS settings) in order to use SSH and other protocols.
