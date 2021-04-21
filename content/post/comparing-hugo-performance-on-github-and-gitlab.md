---
title: "Comparing Hugo performance on Github and Gitlab"
date: 2021-04-21
layout: post
author: Evan Goad
URL: 2021/04/21/comparing-hugo-performance-on-github-and-gitlab
image: /img/20210421/flowerpair.jpg
tags:
    - Hugo
    - Gitlab
    - Github
---

# A Little History: 

A lot has changed since GitHub Pages [first released in
2008](https://github.blog/2008-12-18-github-pages/).  It began as a simple and
free way to host a personal blog, and documentation for your open source
projects.  Now Github Pages is used by huge companies like [Twitter, Facebook,
and Netflix](https://github.com/collections/github-pages-examples), and hosts
large open source projects like [electronjs.org](electronjs.org) and
[jekyllrb.com](jekyllrb.com). 

Jekyll, to this day, is still the static site generator (SSG) recommeded and
documented for github pages.  They have always supported static html content,
so technically any SSG was supported from the beginning. But only Jekyll would
automatically perform the build step in a behind the scenes CI/CD process.
This was very appealing to many people without much CI/CD experience like me.
You could do extra work if you wanted to replicate this behavior with other
SSGs.  There are some older blogs out there about using git hooks, or Travis CI
credits for your Github Pages deploy.

Fast forward to 2015 and [Gitlab
announces](https://about.gitlab.com/releases/2015/12/22/gitlab-8-3-released/)
it's enterprise edition would be getting a feature called Gitlab Pages, and by
the next year it was available to everyone.  In their [introductory blog
post](https://about.gitlab.com/blog/2016/04/07/gitlab-pages-setup/#gitlab-ci),
they lay out specific instructions not just for Jekyll, but for Hexo as well.
They leverage their in house CI/CD system in order to support a flexible build
system for SSGs.  Github would follow this up shortly with [Github Actions in
2018](https://github.blog/2018-10-17-action-demos/), which really opened doors
for SSG convenience.  Now you could share open source CI/CD configuration in
Github, making all other SSGs just as convenient as the Jekyll option.

With all these advancements in SSGs, I began wondering what significant
differences there were between Github Pages and Gitlab Pages, or if there were
any.

# Chosing My Next SSG

I started my [evangoad.github.io](evangoad.github.io) project using Jekyll in
2015, as my first experience trying out an SSG.  Now I'm intersted in looking
at other SSGs, one that fits best with my goal of comparing Github Pages and
Gitlab Pages performance.  My criteria for this SSG was:

1. Builds fast
2. Low effort to setup on the host platform
3. Hosted site recieves good grades in web performance tools 

So I went to [jamstack.org/generators](jamstack.org/generators) to see what the
popular SSGs are at the moment:

![Image of Generators listed at jamstack.org listed in order of popularity:
Next.js, Hugo, Gatsby, Jekyll, Nuxt,
Hexo](/img/20210421/jamstack-generators.png)

[Hugo](https://gohugo.io/) immediately stuck out to me in the sea of
javascript, ruby, and python solutions.  Everybody tells me this Go language is
crazy fast, and Hugo is self described as "The world’s fastest framework for
building websites" in it's [Github project](https://github.com/gohugoio/hugo).
It must build fast!

But is Hugo going to be low effort to setup on Github and Gitlab?  Turns out
that Hugo has fantastic documentation site, and official documentation for both
[Github Pages](https://gohugo.io/hosting-and-deployment/hosting-on-github/) and
[Gitlab Pages](https://gohugo.io/hosting-and-deployment/hosting-on-gitlab/).   

![Screenshot of both the Github and Gitlab documentation
pages](/img/20210421/hugo-docs-support.png)

This is great!  I have met the criteria for #1 and #2 so far.  I won't be able
to test the performance until I am hosted on Github and Gitlab, so we'll forget
about criteria #3 for now.

# Switching from Jekyll to Hugo

I don't have any advice here when it comes to preserving content.  If you have
been relying on simple markdown for your content, it theoretically should be
easy to port your existing site over.  I, on the other hand, had no content on
previous blog that I wished to preserve.  So instead, I opted to delete the
project in [this commit](https://github.com/evangoad/evangoad.github.io/commit/40632008af8265d8e82bf9e087b84e5be1eb9779), so that I may start completely new. Then I followed Hugo's official [Quick
Start](https://gohugo.io/getting-started/quick-start/) document as I built my
new site in the existing evangoad.github.io repository.

First I install the hugo tool with brew:  
```shell
$ brew install hugo
```
Then create a new site. I had to use the `--force` option because this wasn't an empty directory.
```shell
$ hugo new site . --force
```

I looked for a theme on [themes.gohugo.io](https://themes.gohugo.io/), and I
really like this
[hugo-theme-cleanwhite](content/post/building-the-exact-same-hugo-blog-on-gitlab-and-github.md)
created for the personal blog https://zhaohuabing.com

![Screenshot of the example website](/img/20210421/theme-example.png)

Looks great!  I had the option of copying the theme's files directly to my repository, or using a submodule.  I went with the submodule:

```shell
$ git submodule add https://github.com/zhaohuabing/hugo-theme-cleanwhite.git themes/hugo-theme-cleanwhite
```

Referencing the `config.toml` from the theme's example site, I was able to
create my own file:

```toml
baseURL = "example.org"
languageCode = "en-us"
title = "Evan Goad"
theme = "hugo-theme-cleanwhite"
paginate = 5 #frontpage pagination
preserveTaxonomyNames = true

[outputs]
home = ["HTML", "RSS"]

[params]
  header_image = "img/flowers.jpg"
  about_me = true
  sidebar_about_description = "Software Developer, Full Stack and Site Reliability"
  sidebar_avatar = "img/profile.jpg"
  featured_tags = false
  showtoc = false

[params.social]
  linkedin = "https://www.linkedin.com/in/evangoad"
  github = "https://github.com/evangoad"

[[params.addtional_menus]]
  title =  "ABOUT"
  href =  "/top/about/"

[markup]
  [markup.highlight]
    style = "dracula"
```

There's a lot in here, but basically I'm using the configuration provided by
both Hugo and the theme.

Next I generated a post:
```shell
$ mkdir post
$ hugo new post/building-the-exact-same-hugo-blog-on-gitlab-and-github.md
```
I filled that post with a little placeholder markdown content. I also uploaded
some images to the `static/img` directory, and created a small about me page
also with markdown content. 

To test all of these changes locally, I run the server in draft mode:
```shell
$ hugo server -D
```

Looks great to me locally! I decide I'm ready to setup the Github Actions
pipeline.

# Setting up Github Actions

Reading the official Hugo documentation, I realize that all I'm going to have
to do is create a single workflow file:


```yaml
name: github pages

on:
  push:
    branches:
      - hugo_transition  # Set a branch to deploy

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          # extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

And that's it! I'm ready to test a Github Actions deployment.  The only thing I
changed from their example was the build branch `hugo_transition`.  Apparently
`secrets.GITHUB_TOKEN` is always available in Github Actions.  This token
is used to commit a built version of the Hugo site in a branch called
`gh-pages`.  In the repository settings page on Github, I changed the branch
Github Pages was looking at from `main` to `gh-pages`.

At this point, I [commit my
changes](https://github.com/evangoad/evangoad.github.io/commit/0dadfc37fefb60161d2554f3e6bc3a56f958b9ba#diff-28043ff911f28a5cb5742f7638363546311225a63eabc365af5356c70d4deb77)
and I push them to my repository in order to test the deployment.

![Screenshot of my firt Github Action run failing](/img/20210421/github-action-failure.png)

Oh no! It failed! I'm still not sure why, but the build is not working on this
specific part of the theme.  Luckily for me, I selected a theme where the
author specifically states he "tried to make every part of the template as a
replaceable partial html".  This means I can copy just copy the footer template
and comment out the `async` calls that were bugging out. I pushed that change
in [this
commit](https://github.com/evangoad/evangoad.github.io/commit/3df21db92dec56ae520b8c64977b1d3f2e490c78)

![Screenshot of my first successful Github Action run](/img/20210421/github-action-success.png)

Success!  And look at how short that deploy action was, just 12 seconds.  I
checked evangoad.github.io and my static website was up and running. I quickly
realiezed I had to remove a [reference to
example.org](https://github.com/evangoad/evangoad.github.io/commit/cbe9a8bd7975098d4a96ad0dd9cbeadfc5646629)
in my config.toml file so my links could work (did you catch that earlier in
the blog?)  

After that, I merged my changes into the `main` branch and I was done with my
first provider

# Setting up Gitlab

I actually had to sign up with Gitlab and do the whole ssh key song and dance
first.  Once I created a repo, I copied all the files from my Github project
except for the `.github/workflows/gh-pages.yml`.  Did a quick test with `hugo
server -D` to make sure I copied everything I needed, and then [committed those
changes](https://gitlab.com/evangoad/evangoad.gitlab.io/-/commit/4cf3dd9e15d38a93a53f599596b0cb649cffb1ce).

Now it's time to see what the Gitlab CI/CD config is like. According to the
[official Hugo
documentation](https://gohugo.io/hosting-and-deployment/hosting-on-gitlab/) I'm
going to have to... copy an even smaller config file!!

```yaml
image: registry.gitlab.com/pages/hugo:latest

variables:
  GIT_SUBMODULE_STRATEGY: recursive

pages:
  script:
  - hugo
  artifacts:
    paths:
    - public
  only:
  - copy-hugo
```

I was really impressed with how simple and straightforward this config is.  I
was able to get a working build on evangoad.gitlab.io with [this
commit](https://gitlab.com/evangoad/evangoad.gitlab.io/-/commit/b6956f6bc1fa8003009f58d16d7b934e4ebd1655).

![Screenshot of successful deploy on Gitlab](/img/20210421/gitlab-success.png)

Another very fast build at 21 seconds.  I am very happy to discover that Hugo
builds extremely fast on both Github and Gitlab. They have both exceeded my
expectations for criteria #1 and #2

For clean up, I had to change the deploy branch to `main` from `copy-hugo` as
well as a reference to my github site in [this
commit](https://gitlab.com/evangoad/evangoad.gitlab.io/-/commit/a900d237a03ef4c6c782625725da26d4cd48472a).
I added the `--minify` arg in a [later
commit](https://gitlab.com/evangoad/evangoad.gitlab.io/-/commit/cbda056c66c35385b6cc00559d78c7738da472f3)
to make it identical to the Github config.

# Performance testing

The Hugo blog is up and running on https://evangoad.github.io and
https://evangoad.gitlab.io. and I can finally do some performance testing! I
did not know what the best tools were for testing static website performance,
so I did a little spelunking.  Here's the tools I settled on:

1. [Yellowlab.tools](https://yellowlab.tools)
2. [KeyCDN Tools](https://tools.keycdn.com)
3. [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/)

PageSpeed Insights is the super dominant tool when it comes to perfomance
testing, so I feel a bit obligated to include it here.  Yellowlab.tools has
been maintained by Gaël Métais since 2014, and just saw the release of [version
2.0 in
February](https://letstalkaboutwebperf.com/en/yellow-lab-tools-v2-is-out/).  He
describes it as a complimentary tool to Insights as they don't audit the same
issues.  I think KeyCDN is a good addition as well, since this gives me global
web performance averages, as well as a convenient speed test tool.

## Yellowlab.tools

I start with Yellowlab.tools since this an analysis of assets themselves, and
not how they are transported or cached.  I wanted to make sure that the content
I was serving was a similar as possible, so I could isolate the differnces
between the two services.  First, I check the grade for the original theme
(https://zhaohuabing.com/) to set a baseline of expectations:

![Screentshot of zhaohuabing.com's score, a 95 out of 100](/img/20210421/theme-grade.png)

95 out of 100 criteria are met!  That screenshot is only a small portion of the information we
recieve. In the rest of the report there are only a handful issues about
jquery, old css prefixes, and color complexity.

So now lets run the same command for Github:

![Screenshot of my github website's score, a 94 out of 100](/img/20210421/github-grade.png)

94 out of 100!  The only regression here is related to an unused font-awesome
asset, so I am going to ignore it for now.

Now let's see how the Gitlab site is fairing:

![Screenshot of my gitlab website's score, a 91 out of 100](/img/20210421/gitlab-bad-grade.png)

Yikes! There is a clear difference here when it comes to compression of
assets. I searched for guidance around compression and Gitlab Pages, and came
across this article called ["Support Compression in GitLab
Pages"](https://blog.ideotec.es/support-compression-gitlab-pages/).
Apparently, Gitlab is not taking this compression step for us, while Github is
adding either gzip or brotli compression automatically.  The blog describes a
way we can add a compression step to the Gitlab CI build script, so now the
build script looks like this:

```yaml
image:
  name: klakegg/hugo:alpine
  entrypoint: [""]

before_script:
 - apk update
 - apk add brotli

variables:
  GIT_SUBMODULE_STRATEGY: recursive

pages:
  script:
  - hugo --minify
  - gzip -k -9 $(find public -iname '*.html' -o -iname '*.css' -o -iname '*.js' -o -iname '*.xml')
  - brotli -Z $(find public -iname '*.html' -o -iname '*.css' -o -iname '*.js' -o -iname '*.xml')
  artifacts:
    paths:
    - public
  only:
  - main
```
Always happy to switch to an alpine image!  [The deploy](https://gitlab.com/evangoad/evangoad.gitlab.io/-/pipelines/290102416/builds) only took 29 seconds,
and after running Yellowlab.tools again:

![Screen shot of gitlab website's score, 94 out of 100](/img/20210421/gitlab-better-grade.png)

Hooray! Now Github and Gitlab have identical scores.  I am satisfied with how
my content is generated, now it's time to do some speed tests.

## KeyCDN

The KeyCDN Tools page has a lot of useful way for testing speed.  The first I
want to look at is their [Performance
Test](https://tools.keycdn.com/performance).  This allows you to test DNS,
Connect, TLS, and even TTFB from multiple locations.  This is particularly
helpful for understanding the general latency that is going to be present, and
how it deviates per region.  Here are the results for both the github site and
the gilab site: 

![](/img/20210421/keycdn-github-perf.png)
![](/img/20210421/keycdn-gitlab-perf.png)

As you can see, Github is outperforming Gitlab in every category here.  I'm not
sure why this is the case, but it definitely shows that Github has won round 1
of the speed test.  The Performance test is only measuring initial network
responses, so we have no idea how fast the assets are moving at this point.

In order to do that, we'll take a look at some old fashioned speed tests!  This
[Website Speed Test](https://tools.keycdn.com/speed) measures the total amount
of time it takes for a web page to finish loading, assets included.  It has a
sample size of 15, so you are really testing the speed of the page when it
should be cached.

![Screenshot of Github speed, around 500ms on average](/img/20210421/keycdn-github-speed.png)
![Screenshot of Gitlab speed, around 1s on average](/img/20210421/keycdn-gitlab-speed.png)

The Gitlab website is twice as slow as the Github website.  I was noticing this
too when I was browsing my sites in firefox.  The Github hosted one felt
noticeably snappier, so I took a look at the network tab.  And what I found is
that Gitlab is basically never caching the assets!  No wonder it is taking so
much long than Github, which is again, doing something automatically for us.

![Screenshot demonstrating that Gitlab was not caching](/img/20210421/asset-cache-comparison.png)

Another thing I want to note is that the size of the Gitlab website is 11.2 KB
smaller than the Github one.  I'm guessing this is because the Github website
is only using Gzip compression and not Brotli.  I was able to confirm Github
doesn't automatically add Brotli compression with KeyCDN's [Brotli Test](https://tools.keycdn.com/brotli-test) tool:

![Screenshot of Brotli test](/img/20210421/github-brotli-test.png)

## PageSpeed Insights

Last but not least, we have PageSpeed Insights.  I performed an analysis on the
mobile versions of the Github and Gitlab site:

![](/img/20210421/pagespeed-insights.png)
![](/img/20210421/pagespeed-insights-lab-data.png)

Turns out, Google doesn't rate these too differently. I thought the large speed
differences I found in KeyCDN would have reflected greater in the PageSpeed
Insights results.  I have to admit to not having any experience with PageSpeed
Insights before, so I don't really know what goes into a "Speed Index" or a
"First Contentful Paint".  But I do know that we are testing beyond just
response times now.  I imagine that a "paint" requires the html, then the
assets, and then a browser to render the asset.  The Github website
consistently scores higher than Gitlab in this test, showing a strong
indication that the user experience is going to be best on Github.

# Conclusion

I am so happy that I have switched to Hugo.  The build times are blazing fast
on my local machine, as well as Github/Gitlab CI.  When it comes to hosting,
Github clearly has a bit of an edge with the automatic compression and caching
of assets.  However, both take very little time to setup, and aren't too
different by some performance measures.

Going forward I'm interested in learning more about the Hugo theme I'm using,
and ways it could be optimized for performance.  It might be fun to test Github
Pages against another competitor like Netlify in the future.
