---
layout: post
comments: true
title:  "First things first, 5 minutes read on setting up this blog."
date:   2017-05-04 23:31:51 -0400
categories: blog
---

**Welcome to the Cloud Native Blog**, where I will try to keep publishing some cool technologies I am working on and some personal opinions about new tendencies I am seeing in the marketplace.

In this first post I will explain how I was able to create this blog in less than 20 minutes using Jekyll and Google Storage to serve my static pages. I wanted something that was very easy and I could write using markdown to format my page. I thought about writing my own engine using Rails or use Wordpress, but at the end of the day I just wanted to be able to publish something fast and easy, and Jekyll seemed to be a nice solution. Not saying I am never moving to another platform, but this is good for now.

Here is a simple guide to get up and running within minutes; Jekyll is a ruby gem which has a great engine to generate static webpages, where you can write your posts in markdown and publish very simple. GitHub pages uses Jekyll as their engine and you can actually very simple create a blog at GitHub.

First you need to install Jekyll by simply doing:

```ruby
gem install jekyll bundler
```

This will install the gem locally in your computeer. After that you can go ahead and create your new blog by typing

```ruby
jekyll new the-name-of-your-blog
cd the-name-of-your-blog
```

That's it, the structure and layout of your blog should be created now and you will be able to create new posts very easily.

A basic Jekyll site usually looks something like this:

```
.
├── _config.yml
├── _data
|   └── members.yml
├── _drafts
|   ├── begin-with-the-crazy-ideas.md
|   └── on-simplicity-in-technology.md
├── _includes
|   ├── footer.html
|   └── header.html
├── _layouts
|   ├── default.html
|   └── post.html
├── _posts
|   ├── 2007-10-29-why-every-programmer-should-play-nethack.md
|   └── 2009-04-26-barcamp-boston-4-roundup.md
├── _sass
|   ├── _base.scss
|   └── _layout.scss
├── _site
├── .jekyll-metadata
└── index.html # can also be an 'index.md' with valid YAML Frontmatter
```

Now you can just create a new post in the folder posts by following the name convention. It is extremelly important follow the convention otherwise your post won't be created. After you create the post you can go ahead and type

```ruby
jekyll serve
```
This will enable a local web server and you can go ahead and take a look at your page by accessing localhost:4000

```
$ jekyll serve
Configuration file: /Users/cnblog_production/_config.yml
Configuration file: /Users/cnblog_production/_config.yml
            Source: /Users/cnblog_production
       Destination: /Users/cnblog_production/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 0.23 seconds.
 Auto-regeneration: enabled for '/Users/cnblog_production'
Configuration file: /Users/cnblog_production/_config.yml
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```
You can get all the steps and how to customize and improve your blog by reading the jekyll website.

#[Go to Jekyll](https://jekyllrb.com/)

After doing that you can go to your folder and check that Jekyll created another folder called sites where your static webpages live. This is the folder you will have to upload to Google Cloud Storage in order to make your blog available online. I am managing my domain using Google Domains and I am pointing the domain to Google Cloud

Lastly I used Disqus for the comments in the blog. After doing some research, Disqus seemed to be the easiest solution to create the comments section in the blog and it has a rteally good UI to moderate comments.

All the details to post your Jekyll blog to Google Cloud Storage can be found [here](https://little418.com/2015/07/jekyll-google-cloud-storage.html)

For information about Disqus you can take a look at their website and simply creating an account and following the configuration steps. It is quite simple.
