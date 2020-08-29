---
title: "Move to Hugo from Hexo"
date: 2020-08-19T08:51:46Z
draft: false
---

[Hugo](https://gohugo.io/) is another static website generator like [hexo](https://hexo.io/zh-cn/index.html). I'm not sure whether there are any advantages over Hexo, but I just move my blog to Hugo. This is a note on Hugo usage.
<!--more-->

## quick start
``` bash
brew install hugo
hugo version    # verify install

hugo new site quickstart    # create a new Hugo site in folder named quickstart
# add a theme
cd quickstart
git init
git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke
echo 'theme = "ananke"' >> config.toml      # config.toml

hugo new posts/my-first-post.md     # create post; when finish a post, set the header `draft: false`

hugo server -D      # raise the site http://localhost:1313/

hugo -D     # build static pages in ./public directory. or
hugo -d dest_dir -D (with draft)
```
## usage and configure
- draft, future, expired format

  in `front matter`
  ``` md
  draft: true/false
  publishdate
  expirydate 
  ```
- directory structure
  ``` bash
  .
  ├── archetypes    # custom front matter
  ├── config.toml   # config directory or single `config.toml`
  ├── content       # all content for website
  |                 # subdirectories as sections
  ├── data          # configuration files that used for generating website
  ├── layouts       # templates `.html` files
  ├── static        # static content: images, CSS, JS, etc.
  └── themes        # refer to the confiure example of specific theme
  ```
- toml config
TOML is another markup language for configure files. Refer to [All configuration settings](https://gohugo.io/getting-started/configuration/#all-configuration-settings) for Hugo-defined variables. 