---
title: "Make blog posting as a simple Continuous Delivery."
date: 2018-09-18T12:00:00+08:00
weight: 90
keywords: ["blog","Travis"]
description: "Make a simple CD."
tags: ["Travis"]
categories: ["Other"]
author: "Fred"
---

## 1. Continuous Delivery.

What is CD? Please look into [Wiki CD](https://en.wikipedia.org/wiki/Continuous_delivery).
We can see our blog posting as a delivery every time, which can be used as CI/CD process. Well, Let me describe it simpler. Do you remember that we need to run `hugo -d ./docs` each time we doing our posting? This command means generating code from your local repo by Hugo. Only "docs" directory will be used as your website. Now, we try not to do this locally. We need something to do generating for us and then send us a report that the generating is ok or not. The only thing we need to do is push our repo to Github and wait for the report. Here, I need to explain that "generating" we said is actually a very complex part in a complete process of CI/CD for development. This article aims to give you a sense of CI/CD so that you can decouple your regular development process.

Notice: This process is actually not necessary for blog posting.

## 2. CI service.

We choose Travis CI here for our Github repo. Travis CI is a hosted, distributed continuous integration service used to build and test software projects hosted at GitHub. It is a public platform and for free for personal. Please read [Travis CI](https://docs.travis-ci.com/user/getting-started#to-get-started-with-travis-ci)  to get start.

Also, you can deploy your own CI serivce locally such as Jenkins which will be introduced in another article.

## 3. Configuration

### 3.1 Generate a Github Access Token

Please generate a new token through [this link](https://github.com/settings/tokens/new) in your Github. Input "Travis" or any words in the description blank and tick "public_repo". Then click "Generate token" at the end of this page. Please note the token info which will be used in Travis CI.

[![blog-_CI01.jpg](https://i.postimg.cc/L6r6jdH5/blog-_CI01.jpg)](https://postimg.cc/PPQjkRvn)

### 3.2 Configure Travis CI

Please login Travis CI with your Github account and then sync your account and enable your repo.

Click "settings" on the right side.

[![blog-_CI02.png](https://i.postimg.cc/L6d81K4X/blog-_CI02.png)](https://postimg.cc/Z0cSGQdt)

Please input "GITHUB_TOKEN" in "1" and your token info in "2". Then, click "Add" on the right side.

[![blog-_CI03.png](https://i.postimg.cc/TwDXd1Dc/blog-_CI03.png)](https://postimg.cc/cg0zTsTK)

### 3.3 Create .travis.yml

Please create a file named .travis.yml in your repo as below.

[![blog-_CI04.png](https://i.postimg.cc/05YLcLdr/blog-_CI04.png)](https://postimg.cc/c6J9JkzN)

Here is the content for your reference. You can directly use it.

```
sudo: false
language: go
git:
    depth: 1
install:
    - wget https://github.com/gohugoio/hugo/releases/download/v0.48/hugo_0.48_Linux-64bit.tar.gz
    - tar zxvf hugo_0.48_Linux-64bit.tar.gz hugo
script: ./hugo -d ./docs
after_success:
    - rm hugo*
deploy:
    provider: pages
    skip_cleanup: true
    github_token: $GITHUB_TOKEN
    keep-history: true
    on:
        branch: master
    target_branch: master
```

You can customize the life cycle of your build. Please look into [Travis CI Docs](https://docs.travis-ci.com/user/customizing-the-build/)

## 4. Trigger Travis CI

Now, Travis CI will automatically build your code and update your repo every time you push your repo to Github. You will receive an email for the build info. You can also see the process log on the your Travis CI page.

[![blog-_CI05.png](https://i.postimg.cc/rsRbGPJC/blog-_CI05.png)](https://postimg.cc/WqjXTSxh)
