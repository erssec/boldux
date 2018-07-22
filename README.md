# BoldUX - A Jekyll Theme
[![forthebadge](https://forthebadge.com/images/badges/built-with-love.svg)](https://forthebadge.com)

A jekyll theme inspired by BraveUX's [agency website](https://github.com/braveux/braveux) and based on [braveux-jekyll](https://github.com/GlitchWitchIO/braveux-jekyll).

* Responsive design
* Dynamic SEO tags
* Support for Disqus Comments
* Compatible with GitHub Pages
* Simple build process
* Easily customizable

> Demo: [boldux.glitchwitch.io](https://boldux.glitchwitch.io/) and [glitchwitch.io](https://glitchwitch.io/)

[![BoldUX Home Page](https://github.com/GlitchWitchIO/boldux/blob/master/screenshot.png?raw=true)](https://boldux.glitchwitch.io/)

## Table of Contents

1. [Usage](#usage)
2. [Structure](#structure)
3. [Site configuration](#site-configuration)
4. [Meta and Branding](#meta-and-branding)
5. [Customizing text](#customizing-text)
6. [Footer Icons](#footer-icons)
7. [Comments](#comments-via-disqus)
8. [Google Analytics](#google-analytics)
9. [Layout: Post](#layout-post)

## Usage

In command line, do the following steps:
```command
# Clone repo to your local machine
$ git clone https://github.com/GlitchWitchIO/boldux.git
```
```command
# Move into the downloaded repo
$ cd boldux
```
```command
# Build and serve
$ jekyll serve
```
## Structure
```bash
.boldux
├── assets
│   ├── css                       # Main CSS
│   ├── fonts                     # Fonts
│   ├── img                       # Images
│   └── js                        # Javascript
├── _data                         # Navigation data yaml
├── _draft                        # Unpublished drafts
├── _includes                     # Theme includes
├── _layouts                      # Layouts (see below for details)
├── pages
│   ├── about.md                  # About example page
│   ├── blog.md                   # Blog listing page
│   └── contact.md                # Contact page
├── _posts                        # Blog posts
├── _sass                         # Sass partials
├── 404.md                        # 404 Page Not Found
├── browserconfig.xml             # Info for MSTiles
├── CNAME                         # Custom domain to be used for GitHub Pages
├── _config.yml                   # Site configuration
├── feed.xml                      # RSS Feed
├── index.md                      # Home page
├── README.md                     # You are here
└── screenshot.png                # Screenshot in the README
```

## Site configuration
Open `_config.yml` in a text editor to change most of the blog's settings.

Jekyll website *without* a subpath (such as a GitHub Pages website for a given username):

```yml
  url: "https://username.github.io"
```
You'll need to do this before using the theme.

### Meta and Branding

Meta variables hold basic information about your Jekyll site which will be used throughout the site and as meta properties for search engines, social media sites, and browsers.

Change these variables in `_config.yml`:
```yml
  theme_settings:
    title: "BoldUX"                   # Name of website
    keywords:                         # SEO Keywords
    timezone:          UTC            # Post date timezone

    owner:
      email: "hello@jekyll.tld"       # Email used in meta and contact page
      phone: "404-500-1337"           # Phone used in meta and contact page
      city: "Toronto"                 # City used in meta and contact page
      state: "ON"                     # State used in meta and contact page
      postalcode: "M5A 0X0"           # Post Code used in meta and contact page
      address: "404 Address Ave"      # Address used in meta and contact page

      giphy: api_key_here             # Needed for 404 background fail gif
      rss: feed.xml                   # Default RSS feed location

      twitter: glitchwitchio          # Twitter handle used in meta and footer
      github: glitchwitchio           # GitHub handle used in meta and footer

```

### Customizing text

Text on each page can be modified within the tagline and text variables of that page.

```yml
tagline: "Lorem ipsum dolor sit amet."
text: "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt."
tagline2: "Lorem ipsum dolor sit amet."
text2: "Lorem Ipsum <br>Dolor sit amet consectetur."
```

### Footer's icons

Icons from [Font Awesome](https://fortawesome.github.io/Font-Awesome/) are used in the footer. All icon variables should be your username enclosed in quotes (e.g. "username") in `_config.yml`, except for the following variables:

```yml

# Social
owner:
  twitter: glitchwitchio
  github: glitchwitchio
```

### Comments (via Disqus)

Optionally, if you have a [Disqus](https://disqus.com/) account, you can show a
comments section below each post.

To enable Disqus comments, add your [Disqus shortname](https://help.disqus.com/customer/portal/articles/466208) to your project's `_config.yml` file:

```yml
owner:
  disqus-shortname: glitchwitchio # Used if comments are desired
```
You can modify the comments variable within the post file to enable or disable comments per post:

```yml
comments: true                   # Enable comments
```


### Google Analytics

To enable Google Analytics, add your [tracking ID](https://support.google.com/analytics/answer/1032385)
to `_config.yml` like so:

```yml
owner:
  google_analytics: UA-XXXXXXXX-X
```

### Layout: Post

These are the basic features you can use with the  `post` layout.

```yml
---
layout: post                                    
title:  "Sample Post"            # The title of the post
tagline: "This is a tagline"     # Displayed under the title and for seo if no description is set
description: "This is a description."   # Used for SEO
date: 2018-07-17                 # Post date
comments: true                   # Enable comments
image:
  feature: blog/sample-feature.jpg  # Used on the blog listing page
  hero: blog/sample-header.jpg      # Post header and Social Media image
tags: [sample, markdown, html]   # Post tags
---
```

With `feature`, you can add a smaller image than the `hero` to be displayed in the blog listing.
