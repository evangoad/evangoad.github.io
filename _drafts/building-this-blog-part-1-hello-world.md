---
title: "Building this Blog Part 1: Hello World"
permalink: blog/buliding-this-blog-part-1-hello-world
layout: post
---

When I went on my job hunt recently, I wanted to create a simple personal
website to house a blog andlist my contact information.  I wanted a hosting
solution that would cost me no money, and I wanted the source of my website to
be open source.  

After a bit of research online, I decided that the static website hosting
service called [Github
Pages](https://help.github.com/articles/what-is-github-pages/) is the perfect
tool for my needs.  If you own a Github account and are familiar with the
[basics of version control](www.google.com), then you are ready to setup a
static website with Github Pages just like i did.  Here's how I would recommend
going about it:

## Create the repository

The repository for this static website is what Github calls a "User Pages
site", as documented
[here](https://help.github.com/articles/user-organization-and-project-pages/#user-and-organization-pages-sites).
The first step is creating a new github repository named
`$GITHUB_USER.github.io`.  So in my case thats evangoad.github.io.  Visit
[github.com/new](https://github.com/new)

![Image of me creating evangoad.github.io repository](/images/github-create-repository-evangoad-github-io.png)

As you can see, I have already created this repository.  I can't recommend
making this repository private since the static site's contents are going to be
publicly available anyway.  Making your personal static website an open source
project allows it to be versioned and hosted for free on github pages, which is
the most appealing aspect to me.

## Test your deployed site

Yes, by simply having a repository with this name Github Pages kicks into to
gear and reserves the `$GITHUB_USER.github.io` subdomain for you.  Create a
file called `index.html` in the root of your project:

```html
<!-- index.html -->
<h1>Hello World</h1>
```

Commit that to the master branch, and then wait a little while for github to
finish building your webite.  Visit your unique github.io subdomain and see
your html deployed.

If you followed along with this blog, then pat yourself on the back!  You've
taken a first important step to having your own public blog hosted for free.
If you're just looking for free hosting of public html content, then feel free
to start adding other html pages, css, and the works.  However, there is
another advantage to using Github Pages that I haven't mentioned yet.  Look
forward to that in part 2! 

