---
layout: post
title: Jekyll Local Install and GitHub Pages Deployment
---
## Environment
* Bash on Windows Ubuntu 16.04


## Jekyll Install
First install jekyll:
```
sudo apt-get install ruby ruby-dev
sudo apt-get install gcc make
sudo -H gem install jekyll bundler
jekyll new [username].github.io
cd [username].github.io
bundle exec jekyll serve
```
You may come across some dependency issues during `sudo -H gem install jekyll bundler`, 

I.E., `zlib`, `sudo apt-get install zlib1g-dev` will do.

if `bundle exec jekyll serve` runs successfully, you will have a local jekyll blog.

Head to [http://127.0.0.1:4000/](http://127.0.0.1:4000/) to check it out.

Next, we will deploy it to GitHub Pages.

## Create GitHub Pages Repository
Head over to GitHub and create a new repository named `[username].github.io`, 
where username is your username on GitHub.

Just use the default settings, no need to make any changes here.

## Deploy to GitHub Pages
To deploy our local jekyll blog to GitHub Pages, just add it to our newly created [username].github.io.

Inside [username].github.io
```
git init
git add .
git config user.email "[email]"
git commit -m "[commit message]"
git remote add origin https://github.com/[username]/[username].github.io
git push origin master
```
Head over to [[username].github.io]([username].github.io) to check out your jekyll blog on GitHub Pages.

## To Do
* Change theme;
* Minima theme add table style.

## Reference
* [https://pages.github.com/](https://pages.github.com/)
* [https://jekyllrb.com/docs/quickstart/](https://jekyllrb.com/docs/quickstart/)
* [https://jekyllrb.com/docs/installation/#requirements]( https://jekyllrb.com/docs/installation/#requirements)
* [https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/](https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/)
