---
title: "设置Fire TV"
layout: post
date: 2019-07-17 22:01
image: /assets/images/markdown.jpg
headerImage: False
tag:
- 闲聊
category: blog
author: bobzhai
description: 设置ADB在FireTV上安装第三方软件
---

这两天Amazon的Prime day，闲着没事，于是买了个Fire TV，买不起4K的，就买了个贫民版本的。今天收到货，打开一看，官方的应用很多都需要钱买，本着折腾的精神(和今天option亏了几千块钱)，就装点第三方应用吧。
很奇怪的是，搜索中文发现很少在海外安装国内应用的，估计是大家都不太care，直接用hulu，HBO等等。我主要是想看国内的一些频道（CCTV啥的），想有点国内的感觉，所以折腾一下。Anyway，废话不说了，说说怎么设置吧。

首先因为在国外，所以不用担心翻墙的问题，正常按流程输入wifi信息，账号就能顺利初始化好，接着就开始装第三方软件。

1. 进入setting，找到My Fire TV（旧版本叫做Device）-> Developer Options
   * ADB debugging 设置成 ON （设置这个为了后面可以用ADB插入软件）
   * Apps from unknown source 设置成 ON （这个为了可以安装软件）

2. 返回上一级，找到About -> Network 查看局域网IP

   * 通常是192.168.x.x 这个地址后面需要，为了locate这个fireTV

3. 现在在电视上的设置就完了，然后打开电脑，先安装ADB

   * ADB全程是android debug bridge，这里不讲技术，就知道这个是个command line tool，可以把电脑上的软件传输到Fire TV上就好了

   1. 打开Terminal，```brew cask install android-platform-tools```
      * 提示没有Brew的先安装Homebrew：`ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
   2. 测试一下：`adb devices`没有报错的话就是安装好了
   3. 下载想要的软件，软件的后缀是apk, 不知道下载什么软件的可以搜‘’Fire TV 安装什么软件‘，我在这里用穿梭做一个例子，可以在[穿梭下载地址](https://www.transocks.com/tv_wechat.html) ,点击一下就下载到本地（这里假设在Download文件夹）
   4. 电脑连接Fire TV: `adb connect 192.168.0.1`把‘'192.168.0.1'换成自己的IP地址
      1. 如果提示需要加入端口，就加入5555，例如‘192.168.0.1:5555’
      2. 可以看这个[端口问题](https://www.cnblogs.com/zzugyl/p/10607845.html)
   5. 连接好了以后确认一下`adb devices` , 如果看到List of devices attached的话就是连接好了
      * 如果没有连接好，就是看不到有devices的话
        1. 看看电视，看是否叫授权，没有授权
        2. `adb kill-server`
        3. `adb connect xxx.xxx.xxx.xxx` 或者 `adb start-server`
   6. 上传软件：`adb install /Users/Downloads/Transocks_TV_3.0.0_align.apk `把后面的软件地址换成自己刚下载的软件位置就好了
   7. All set

   再不会的话欢迎留言

   一些帖子：

   下面第一个链接其实也很方便，或者说更方便，我这篇文章基本上就是翻译了他的文章

   https://sideloadfiretv.com/sideload-apps-amazon-fire-tv-mac/
   https://www.cnblogs.com/zzugyl/p/10607845.html
   http://www.blogjava.net/anchor110/articles/335866.html

   