---
title: "Let's make our own blogs."
date: 2018-09-18T12:00:00+08:00
weight: 100
keywords: ["blog","Hugo"]
description: "Make a blog website."
tags: ["Hugo"]
categories: ["other"]
author: "Fred"
banner: ""
menu: ""
---

### 1. Blog.

**Nowadays, when we want to create a blog website, we don't need to write so much codes of front end, back-end or even set up resources for the website. Thanks to our Open source community. At all times, we can use many blog engines for setting our own blogs, which is a very convenient way to make it done. This article will show you some basic process to complete this task.**

**There are several engines you can use for your blog such as Jekyll, Octopress, Hexo and so on. Please google it. All of them are awesome and easy to use. Choose the one you prefer. Here, we will use a engine named Hugo for example which is a world-famous web framework and a static site generator written in Go.**

***Notice: There are some other ways to set up blog website such that use wordpress docker and deploy it on the Cloud or just buy the Cloud static site service. Furthermore, there are many social websites which can help you to set your own blog site with your domain. The way this article will offer is interesting but not easy. However, it is for free.***

### 2. Preparation.

**2.1 Become a github user.**

**How to get it? Please refer to [Github Bootcamp](https://help.github.com/categories/bootcamp/) and [Github Setup](https://help.github.com/categories/setup/)**

**2.2 Choose a Markdown editor**

**This is for editing your blog. Please google it and choose one. I recommand Atom here if you have no idea. How to get it? Please refer to [Atom](https://atom.io/)**

### 3. Engine installation.

**Installation details please refer to [Hugo manual](https://gohugo.io/getting-started/installing/). If you use Ubuntu and golang environment, you can directly use my script.**

```
wget https://github.com/gohugoio/hugo/releases/download/v0.48/hugo_0.48_Linux-64bit.tar.gz -P ~
cd /usr/local/bin
tar zxvf ~/hugo_0.48_Linux-64bit.tar.gz hugo    //We only need this binary file.
mkdir -p $GOPATH/src/github.com/$YOURUSERNAME    //Please replace $YOURUSERNAME with your Github account.
cd $GOPATH/src/github.com/$YOURUSERNAME
hugo new site $YOURREPO    //$YOURREPO will be your new Github repo.
```
**Below is the directory structure of your site.**
```
blog
├── archetypes
│   └── default.md
├── config.toml
├── content
├── data
├── layouts
├── static
└── themes
```
**"config.toml" is the normal configration of your website. In most cases, we copy the "config.toml" of the example site in the theme as template and then modify it.**

**"content" directory will store article files which formed as "xx.md".**

**"static" directory will store image files and other static files such as css and js files. All of them are customized by yourself.**

**"themes" directory will store the themes you choose which are downloaded from the theme owner.**

### 4. Choose your favorite theme.

**You can choose your favorite theme from [Hugo Themes](https://themes.gohugo.io/) and [Chinese Hugo](http://www.gohugo.org/theme/). Please read the user manual of your themes. Most of them are put on Github and very esay to use. Then, please create a repo($YOURREPO) for your blog site on your own github.**

**After repo creation, please follow my script to relate your local repo to Github repo.**

```
cd $GOPATH/src/github.com/$YOURUSERNAME/$YOURREPO    \\Please replace $YOURUSERNAME and $YOURREPO mentioned before.
git init
git remote add origin https://github.com/$YOURUSERNAME/$YOURREPO.git
git submodule add https://github.com/$THEMEOWNER/$THEME.git themes/$THEME    \\Please replace $THEMEOWNER and $THEME with the info you find within your theme on the Github. For example, you found your theme under curttimson/hugo-theme-massively. Then you will input "git submodule add https://github.com/curttimson/hugo-theme-massively themes/hugo-theme-massively"
```

### 5. Post your articles.

**You can use `hugo new post/$YOURARTICLE` command to create your article files or directly create new files on the path "content/post" of your site. Please notice the file should named ending with ".md" such as "new.md" etc. which means using Markdown to format your articles. Open your Markdown editor to edit your articles. Markdown is easy to use. Please google it if you have no idea. In Atom, press `ctrl+alt+m` for markdown preview.**

**Below is the meta template in your article for your reference.**
```
---
title: "Let's make our own blogs."
date: 2018-09-18T12:00:00+08:00
weight: 100
keywords: ["blog","Hugo"]
description: "Make a blog."
tags: ["Hugo"]
categories: ["other"]
author: "Fred"
---

Here will be your article content.
```

### 6. Preview your website.

**Please modify "config.toml" and other customized file we mentioned in 1.2 and then follow my script.**

***Notice: You need to add `staticDir = ["static", "../../static"]` in your "config.toml" if you add files in "static" directory.***
```
cd $GOPATH/src/github.com/$YOURUSERNAME/$YOURREPO    \\Please replace $YOURUSERNAME and $YOURREPO mentioned before.
hugo server    \\This will start a web service.
```
**After that, you will see something like below.**
```
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```
**Please open the URL(http://localhost:1313/) with your browser to check your website. If something wrong, please review the manual of your theme and your customized file again.**

### 7. Publish it on Github Page.

Please follow below script to push repo to Github.
```
cd $GOPATH/src/github.com/$YOURUSERNAME/$YOURREPO    \\Please replace $YOURUSERNAME and $YOURREPO mentioned before.
mkdir docs    \\This folder will be used by Github Page
hugo -d ./docs    \\This will generate all static files to "docs" directory for your website.
git add -A
git add -u .
git commit -m "$UPDATE"    \\Please replace $UPDATE with any words for this update.
git push origin master
```
You will see the changes on your Github repo.

Please find Github Page options under the settings tab of your repo. Select "master branch /docs folder" in the "Source" block. Then click "Save". After a while, you will see the message "Your site is published at https://haroldwo.github.io/xx/". Now, you can follow the URL with you browser to see your website.