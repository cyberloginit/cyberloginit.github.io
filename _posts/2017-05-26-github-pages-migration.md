---
layout: post
title: GitHub Pages Migration
---

Getting bored with Ubuntu 16.04's sluggish Unity GUI, I decided to have a Ubuntu GNOME 16.04 fresh install.
After that, I had no idea on how to setup my writing environment for GitHub Pages for a while.

I preferred Visual Studio Code, and wrote all of my posts locally in the repository.

There is a full backup of the repo in my Dropbox, and it is synced. So I just open the directory, `git pull origin master`, and copy the private key I used
for the GitHub account into `~/.ssh/`.

That's it. Just open your favorite editor and write a new post, `git push` works like a charm!

## Reference
* https://www.smashingmagazine.com/2014/08/build-blog-jekyll-github-pages/