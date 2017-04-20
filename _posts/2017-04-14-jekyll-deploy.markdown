---
layout: post
title:  "Deploying jekyll with custom plugins to github pages"
date:   2017-04-14
categories: jekyll update github deploy
tags: jekyll github deploy
---

In the process of building this site and [www.kimchichingu.com](http://www.kimchichingu,com)
I had to figure out many things, one of the most important being of course, deploying.
When deciding to deploy a site one of the most important things to decide is where
to host it, I chose to use github pages, like any cheap-skate developer with a
github account would. This seemed like it was going to work out great, just make changes
and github will build and deploy to github pages for me! What could go wrong?

## Plugins

Custom plugins and any other ruby gems. Github doesn't allow for the use
of non-standard jekyll in their github pages if you want them to build it for you.
They list on their site that it's due to security reasons, which is fine, but what's
a developer to do? Well, you can build it yourself and deploy it to a /docs subfolder
in your project, or checkout master in a gh-pages branch and then remove everything
that isn't part of your generated site. Both of those are messy code wise and
git history wise. I've found a better way.  

## git subtree

```
git push origin :gh-pages
git subtree push --prefix _site/ origin gh-pages
```  

Instead, delete the remote branch, and push your compiled site as an orphan branch!
This keeps your history clean and prevents you from checking in un-needed files, but
not all is well yet. First you still have to change your baseurl if you use a
different one locally, and also you have to be sure that the build folder is not
in gitignore. This means you have to be sure to not accidentally check your built
site in and also to not check in your incorrect config.

The alternative I've found is having a \_config.yml and a \_config.yml.example and
only checking in the example file to git. In my case as Kimchu Chingu is open source
I want anyone to easily clone and build the repo, so a master config is important.
I also have a .gitignore-publish file that I use when I deploy. This requires you
to rename files, then once pushed, rename them back which is messy and time consuming
as well as error prone. To minimalize this, I made a simple script that does it
all and cleans up to be sure that nothing weird gets checked in.

```
mv .gitignore .gitignore-local
mv .gitignore-publish .gitignore
mv _config.yml _config-local.yml
mv _config.yml.example _config.yml
bundle exec jekyll build
git add -A
git commit -m "publish"
git push origin :gh-pages
git subtree push --prefix _site/ origin gh-pages
mv .gitignore .gitignore-publish
mv .gitignore-local .gitignore
mv _config.yml _config.yml.example
mv _config-local.yml _config.yml
git reset HEAD~1
git stash
```

Since I use this script in two places and didn't want to copy it include any changes
I made a [repo](https://github.com/samuraiseoul/JekyllDeploy) for the script alone. Then I add it to jekyll projects that deploy
to github pages using

```
git submodule add git@github.com:samuraiseoul/JekyllDeploy.git
```

This prevents code reuse and allows easy updating in the future. Once you have the
module pulled in, just run ```sh JekyllDeploy/publish.sh``` and your site should
build and be deployed!

Happy deving!
