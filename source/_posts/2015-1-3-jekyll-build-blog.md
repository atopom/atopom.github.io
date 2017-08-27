---
layout: post
title:  "通过GitHub和Jekyll搭建属于自己的博客"
date:   2015-01-03 20:29:37 +0800
author: Atopom
categories:
- Work
- Technology
tags:
- Jekyll
- GitHub
published: true
---

> 很早之前就想弄一个自己的博客玩一玩，不过由于各种原因，主要还是本人比较懒，一直拖到现在才开始整。以前写一些笔记和心得都是通过CSDN或Evernote，都是方便留着自己查看的，写的也不是特别细心。现在为了提升自己，强逼自己做出一些改变，用心总结并分享出去。言归正传，今天给大家分享的是如何搭建自己的博客！

* content
{:toc}

# 搭建博客的条件
1. 注册GitHub的账号
2. 安装Git环境、Jekyll环境
3. 学会Markdown基础语法

## Jekyll安装

``` bash
$ gem install jekyll
```

注意：Jekyll环境在Mac上安装稍微遇到点问题（主要原因是Mac版本升级后对在其上面安装的程序的目录和权限有些变化，导致的一些乱七八糟的问题，随后百度、谷歌，也都解决了)。  

## 手动搭建博客
### 目录结构
根据以下目录结构在你的username.github.io文件夹下建立以下目录结构：  

``` bash
* _config.yml  
- _drafts  
    * begin-with-the-crazy-ideas.textile  
    * on-simplicity-in-technology.markdown  
- _includes  
    * footer.html  
    * header.html  
- _layouts  
    * default.html  
    * post.html  
- _posts  
    * 2015-1-3-jekyll-build-blog.md  
    * 2015-1-3-Markdown-Grammar.md  
- _site  
    * index.html  
```

可以一个个依次建立起来，然后再自己编写一个你想要的博客。  

### 配置
项目需要配置的只有一个文件`_config.yml`，打开之后按照如下进行配置。  

> 特别注意`baseurl`的配置。如果是`***.github.io`项目，不修改为空''的话，会导致JS,CSS等静态资源无法找到的错误

``` bash
name: 博客名称
email: 邮箱地址
author: 作者名
url: 个人网站
### baseurl修改为项目名，如果项目是'***.github.io'，则设置为空''
baseurl: ""
resume_site: 个人简历网站
github: github地址
github_username: github用户名称
FB:
  comments:
    provider: duoshuo
    duoshuo:
        short_name: 多说账户
    disqus:
        short_name: Disqus账户
```

### 写文章
在`./_posts`目录下新建一个文件，可以创建文件夹并在文件夹中添加文件，方便维护。在新建文件中粘贴如下信息，并修改以下的`titile`,`date`,`categories`,`tag`的相关信息，添加`* content {:toc}`为目录相关信息，在进行正文书写前需要在目录和正文之间输入至少2行空行。然后按照正常的Markdown语法书写正文。

``` bash
---
layout: post
title:  标题
date:   2016-08-27 01:08:00 +0800
categories: Work
tag: Markdown
---

* content
{:toc}


正文......
正文......
正文......
```

### 查看效果

在终端执行如下命令：

``` bash
$ jekyll server
```

打开浏览器并输入URL`http://127.0.0.1:4000/`,回车。

## 快速搭建博客
参考[Jekyll-Now](https://github.com/barryclark/jekyll-now/)，可以快速搭建极简样式博客。  
参考[Jekyll主题](http://jekyllthemes.org/)，里面提供Jekyll的各种博客模版下载。  

# 参考链接
1. [GitHub官网](https://github.com/)
2. [Jekyll-CN](http://jekyllcn.com/)
