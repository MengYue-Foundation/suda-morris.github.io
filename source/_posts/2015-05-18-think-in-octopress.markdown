---
layout: post
title: "Octopress学习手记"
date: 2015-05-18 13:22:49 +0800
comments: true
categories: [Study,Octopress]
description: "Octopress学习手记"
keywords: Octopress
---
##Windows下环境搭建
1. 安装好Git，配置Git全局参数
	* `git config --global user.name "suda-morris"`
	* `git config --global user.email "362953310@qq.com"`
	* `git config --global --list`
2. 生成SSH密钥并与github关联
	* `ssh-keygen -t rsa -C "362953310@qq.com"`
	* `将id_rsa.pub里面的内容拷贝到github中新建的SSH密钥中保存`
	* 测试是否关联成功`ssh -T git@github.com`
3. 安装Ruby，安装时选择将Ruby添加到系统环境变量中
4. 安装Ruby后测试是否成功,终端下使用命令`ruby -v`
5. 安装DevKit，进入DevKit文件夹下运行`ruby dk.rb init`和`ruby dk.rb install`
6. 安装Octopress源码并且设置默认主题
	* `git clone git://github.com/imathis/octopress.git Octopress`
	* 修改gem的软件源`gem sources -a http://ruby.taobao.org`
	* 移除原始gem软件源`gem sources -r http://rubygems.org`
	* 打开Octopress文件夹下的Gemfile文件，将里面`source "http://rubygems.org"`修改为`source "http://ruby.taobao.org"`
	* 安装bundler`gem install bundler`
	* 安装bundler里面的软件包`bundle install`
7. 安装默认主题`rake install`
	
##常用命令
* 新建博客 						`rake new_post['New Blog']`
* 新建单页面 						`rake new_page['New Page']`
* 编译生成网页 					`rake generate`
* 本地服务器预览（默认4000端口）	`rake preview`
* 发布网页 						`rake deploy`

##基本语法
* 一级标题		`#`
* 二级标题		`##`
* 三级标题		`###`
* 引用			`>`
* 加粗			`**内容**`
* 斜体			`*内容*`
* 列表			`*`或者`-`
* 图片			`！[图片的描述](链接地址)`

##优化
* 依次进入source目录，_includes目录，打开里面的`head.html`文件
* 替换`head.html`文件里面的jquery链接地址为`src="//libs.baidu.com/jquery/1.9.1/jquery.min.js"`

##参考文献
1. [MarkDown中文语法详解](http://www.appinn.com/markdown/)
2. [极客学院Octopress课件](http://wenku.baidu.com/view/1e250851240c844769eaeec6)


![suda-morris](http://i.imgur.com/Nn7Krru.gif)