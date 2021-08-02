---
layout: post
title:  "repo版本管理工具的SDK"
date:   2021-8-2
categories: Python repo git
tags: git repo
excerpt: 使用repo做版本管理
--- 


## 版本管理工具 repo

repo 是git 的python 封装  清单文件是xml 的格式

### repo 拉取命令

` repo init -u ssh://ppgerrit.com/Mstar648/manifest.git -b 648_cultraview -m ppos4.5.0_cultraview.xml`

- -u：指定一个URL，其连接到一个manifest仓库

- -b：选择manifest仓库中的一个特殊分支

- -m：在manifest仓库中选择一个xml文件

  

### 同步代码

​	repo sync	



## 自己在获取服务器资源上的努力

没有权限访问外网，所以下载不了东西，不能使用repo 拉取代码



repo init -u ssh://verify@172.16.1.49/git_repo/D1_Tina_Open/manifest.git -b master -m tina-d1-open.xml



不能下载SDK 先研究一下



犯了一个及其致命的错误  就是没有设置 git 的 user 信息  同时下载服务器的路径还填错了，太致命了



经过这些操作  完成了 Hello Word 的输出  并整理了文档