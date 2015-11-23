---
layout: post
title: Sync Sublime Text settings using Dropbox
categories: blog
description: Use Sublime Text across multiple devices.
---

It becomes useful to sync your preferences and packages across multiple machines. Instead of recreating your workflow every time you reinstall your operating system or buy a new machine it's easier to pull your configuration from Dropbox.

![Sublime Text config](/images/2014/08/sublime-config.png)

This is an excerpt from [Package Control documentation](https://sublime.wbond.net/docs/syncing#dropbox-osx) (visit the link for Windows and Linux).

1. Open Terminal.app
2. Go to your Sublime Text folder: `cd ~/Library/Application\ Support/Sublime\ Text\ 3/Packages/`
3. Create Sublime Text folder in your Dropbox: `mkdir ~/Dropbox/Sublime`
4. Move your settings from local machine to Dropbox folder: `mv User ~/Dropbox/Sublime/`
5. Create a link to the Dropbox folder we created: `ln -s ~/Dropbox/Sublime/User`

Then on the other machine:

1. Go to your Sublime Text folder: `cd ~/Library/Application\ Support/Sublime\ Text\ 3/Packages/`
2. Remove User folder: `rm -r User `
3. Create a link to Dropbox backup: `ln -s ~/Dropbox/Sublime/User`
