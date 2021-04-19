---
title: "Hosting the exact same Hugo blog on Github and Gitlab"
date: 2021-04-19
layout: post
author: Evan Goad
URL: 2021/04/19/hugo-on-github-and-gitlab
tags:
    - Hugo
    - Gitlab
    - Github
---

I decided that it would be a fun exercise to update my blog from scratch.  I had a Jekyll blog hosted on Github pages, but I haven't touched the thing in years. I wanted to see what the current state of static site generators, and see what the alternatives are 

Hugo is another static site generator written in Go.  It provides all of the same features I had been using Jekyll, and since it's written in Go, it builds super fast.

install the hugo tool and build a new site:
```shell
$ brew install hugo
$ hugo new site . --force
```
Pick a theme:
```shell
$ git submodule add https://github.com/zhaohuabing/hugo-theme-cleanwhite.git themes/hugo-theme-cleanwhite
```
Generate a post:
```shell
$ mkdir post
$ hugo new post/my-first-post.md
```
Run the server locally:
```shell
$ hugo server -D
```

Then I will talk about configuration changes in config.toml, adding static images, and some of the parameters I can specify on my post.
