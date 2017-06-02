---
layout: post
title:  "How to setup Jekyll on Ubuntu/Raspberry Pi"
date:   2017-03-13 21:17:57 +0800
excerpt_separator: <!-- excerpt -->
---

This is the installation guide for Jekyll on Ubuntu/Raspberry Pi. Normally there won't be any problem to install on Ubuntu but installation for Raspberry Pi is a little tricky due to SSL certificate issue.

<!-- excerpt -->

### 1. Install Ruby and gems
First we will install [Ruby](https://www.ruby-lang.org/en/documentation/installation/){:target="_blank"}.  

`$ sudo apt-get install ruby-full`

### 2. Update Rubygems
Ruby uses gem as package manager. We should get the lastest version of it.

`$ sudo gem update --system`

> :memo: You can check your Rubygems version by `gem --version`

### 2.1 Update Rubygems manually (If you have SSL certificate problem)
Download the Rubygems. You can get the latest version from [here](https://rubygems.org/pages/download){:target="_blank"} and more info about SSL issue [here](https://github.com/juthilo/run-jekyll-on-windows/issues/34){:target="_blank"}.

```
$ wget https://rubygems.org/gems/rubygems-update-2.6.10.gem --no-check-certificate
$ sudo gem install rubygems-update-2.6.10.gem
$ sudo gem sources --remove https://rubygems.org/
$ sudo gem sources -a http://rubygems.org/
$ sudo update_rubygems
$ sudo gem update --system
```

### 3. Install Dependency Packages

```
$ sudo apt-get install build-essential
$ sudo apt-get install zlib1g-dev
```

### 4. Install Jekyll
Now we start installing as Jekyll gem. More info from [here](http://jekyllrb.com/docs/installation/){:target="_blank"}.

`$ sudo gem install jekyll`

### 5. Install bundler gem
This is also a gem but it manages others gems. More info from [here](http://bundler.io/){:target="_blank"}.

`$ sudo gem install bundler`


### 6. Setup Jekyll
```
$ cd ~
$ mkdir JekyllWorkSpace
$ cd JekyllWorkSpace
$ jekyll new myblog
$ cd myblog
$ bundle install
```

> :exclamation: If you have again SSL certificate problem while you type in `sudo bundle install`, change `source "https://rubygems.org"` to `source "http://rubygems.org"` in `Gemfile` which is located in newly crated `myblog` folder.


### 7. Run server
Now it's time to see the real action.

`$ bundle exec jekyll serve`

Now type this `http://localhost:4000` into any browser address. You should see a home page.
