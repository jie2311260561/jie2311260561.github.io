---
layout: post
title:  "哪吒的眼睛的demo"
date:   2021-08-4 10:04
categories: vscode
tags: V4L2 C Linux
excerpt: 在哪吒上使用摄像头完成一些操作（未完）
---





USB carmer 使用opencv 进行调用  

编译参数

```
cmake -D CMAKE_BUILD_TYPE=RELEASE
-D CMAKE_INSTALL_PREFIX=/home/gaojies/workspace/d1-tina-open/tools/opencv/install
-D OPENCV_GENERATE_PKGCONFIG=ON
-D INSTALL_PYTHON_EXAMPLES=ON
-D INSTALL_C_EXAMPLES=ON
-D OPENCV_EXTRA_MODULES_PATH= /home/cht/opencv4/opencv_contrib-4.0.1/modules
-D OPENCV_EXAMPLES=ON ..
```

Video4Linux2(Video for Linux Two, 简称V4L2)是Linux中关于视频设备的驱动框架，为上层访问底层的视频设备提供统一接口。V4L2主要支持三类设备：视频输入输出设备、VBI设备和Radio设备，分别会在/dev目录下产生videoX、vbiX和radioX设备节点，其中X是0,1,2等的数字。如USB摄像头是我们常见的视频输入设备。FFmpeg和OpenCV对V4L2均支持。

使用V4L2库完成摄像头图片捕捉，获取成功保存为一张图片。

保存图片之后，在HDMI上显示摄像头画面，然后将当前的画面的图片进行截取保存。

​	在test/camerademo/test.c 程序可以实现

​	在test/usbvamer/photo.c 程序编译没有问题，会在mnmap的时候挂掉，还不知道是怎么回事。

继续尝试使用opencv来进行摄像头数据采集。