---
layout: post
title:  "nezha上手hello word"
date:   2021-8-5
categories: nezha C
tags: C Linux nezha
excerpt: 哪吒软件上手
--- 



# 哪吒 从上手到hello word

经过上面的编译烧录固件相信你的哪吒已经可以跑起来了（假装起飞），如果没有烧录它默认也是可以跑起来的因为出厂已经搭载了固件（可能不是最新版）。

现在让我们正式开启嵌入式编程的第一步：Hello Word！！！ （真正起飞）

### 前要知识储备：

- 掌握Linux基本命令行指令及工具
  - 如 git make 以及文件操作等
- 掌握Linux 简单编程
  - 基本的C语言编程
- 掌握嵌入式基本知识
  - 嵌入式设备组成
  - 嵌入式外设连接方式
    - 如UART USB 网口 等基本接口
- 了解哪吒板载资源与对应接口分布

## 资源获取

（1）代码及Makefile文件获取链接：

（2）adb工具获取链接：

（3）针对没有进行SDK整编和镜像烧录的同学我们提供了哪吒的交叉编译工具链，**如果有下载SDK的同学此处可直接略过**：

下载好的交叉编译链（压缩包）发送至Ubuntu虚拟机中，推荐在用户目录下创建Tools文件夹，用来存放各种工具。

使用一下命令对交叉编译链进行解压：

```
tar -zxvf riscv64-glibc-gcc-thead_20200702.tar.gz
```

解压后可以看到已经解压出来的交叉编译链的文件夹：

![image-20210715162914008](..\img\image-20210715162914008.png)

## 代码编写

创建文件，根据Tina开发惯例，开发者的个人代码或应用工程位于package中，在package中创建test文件夹，在test文件夹中创建`hello_word.c`文件。

`mkdir package/test`

```bash
# 进入test文件夹
cd package/test
# 创建hello_word.c文件
touch hello_word.c
```

然后编写 Hello Word 代码：

```C
#include <stdio.h>
int main(int argc, char const *argv[])
{
    printf("Hello NeZha\n");
    return 0;
}

```

## 交叉编译

交叉编译是指在我们的PC机上编译可以在开发板上运行的可执行程序文件。这里提供两种方法

（1） 直接使用交叉编译链工具

（2） 使用makefile

***注：星号处请自行补齐目录**

交叉编译链在SDK中的路径：`**/prebuilt/gcc/linux-x86/riscv/toolchain-thead-glibc/riscv64-glibc-gcc-thead_20200702/`

![DBBB1A66-3F21-4f14-B6D5-84FEA7B07E57](..\img\DBBB1A66-3F21-4f14-B6D5-84FEA7B07E57.png)

**可选：将交叉编译链设置为当前环境变量*

```bash
export PATH=**/prebuilt/gcc/linux-x86/riscv/toolchain-thead-glibc/riscv64-glibc-gcc-thead_20200702/bin/:$PATH
```

### 使用编译指令：

``` 
/prebuilt/gcc/linux-x86/riscv/toolchain-thead-glibc/riscv64-glibc-gcc-thead_20200702/bin/riscv64-unknown-linux-gnu-gcc -o hello_word hello_word.c
```

会在当前文件夹生成`hello_word`文件，为哪吒可执行文件。

然后将Ubuntu中的hello_word文件拷贝出来，拷贝至Windows中，我将其放在`E:\BedRock\nezhaimg`中。![67EF04E4-DC2C-4077-9147-3D0CAB57CFAD](E:\BedRock\allwinnerofcode\img\67EF04E4-DC2C-4077-9147-3D0CAB57CFAD.png)

## 传入（烧录）文件方法

传入文件可使用的方法多种多样，仁者见仁智者见智。可用的方法简单列举

1. ADB工具
2. nfs挂载文件系统
3. 使用SD卡挂载

在这里推荐使用我们的ADB工具来进行传输，不需要增加多余的连接，仅仅只需要一根USB线即可。

### ADB

ADB的使用及介绍链接就贴在这里了：[ADB使用上手连接](https://www.cnblogs.com/lsdb/p/9438215.html)

确认设备连接正常后：

```CMD
adb push hello_word /root
```

ADB为Windows下工具，所以使用CMD来执行。

**确保ADB已经添加进环境变量中**

***注意：Windows 下的路径为反斜线  Linux中为斜线**

此时在Tina跟文件系统中的`/root`目录下就有`hello_word`文件。

赋予它可执行权限

```
chmod +x hello_word
./hello_word
```

执行结果：

```bash
root@TinaLinux:~# ls
hello_word  main
root@TinaLinux:~# chmod +x hello_word
root@TinaLinux:~# ./hello_word
Hello NeZha
```

![416F6F32-1640-43ed-827B-EA1FC1292C68](..\img\416F6F32-1640-43ed-827B-EA1FC1292C68.png)



## 扩展：

当你看到Hello NeZha时，自己写的程序通过交叉编译的方法已经跑在我们的板子上了。为了紧密结合嵌入式开发，此处提供使用Makefile 文件来进行编译Hello word 方法：

在源码目录创建Makefile文件：

```bash
touch Makefile
```

编写Makefile:

```makefile
#设置编译链路径及工具
CTOOL := riscv64-unknown-linux-gnu-
CCL := /home/kunyao/workspace/d_tina_d1_open_v1.0/prebuilt/gcc/linux-x86/riscv/toolchain-thead-glibc/riscv64-glibc-gcc-thead_20200702
CC := ${CCL}/bin/${CTOOL}gcc

#设置编译规则
hello_word:hello_word.c
	${CC} -o hello_word hello_word.c

#清理规则
clean:
	rm hello_word
```

保存后在终端`make`即可生成`hello_word`文件，用如上ADB方法将其传入开发板即可。