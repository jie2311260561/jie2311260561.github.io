---
layout: post
title:  "nezha上手网络连接"
date:   2021-8-5
categories: nezha 
tags: C Linux nezha network Tools SSH
excerpt: 哪吒网络连接上手
--- 


## 哪吒硬件
    哪吒上有板载的硬件无线网卡，加上使用Tina的测试固件，可以使用wifi_*****的wifi测试demo


哪吒的网卡为 R853XX驱动。

使用测试demo的时候需要  确保  wpa_supperar  服务启动起来了  


在调试的过程中就遇到了 驱动没有问题   上层的测试demo 因为是编译出来的 也没有问题   经过查看  是中间服务起不起来  然后就发现了问题  但是没有解决办法   


同时在 查看的过程中学了 两个  git 的指令


git diff .

git status .

