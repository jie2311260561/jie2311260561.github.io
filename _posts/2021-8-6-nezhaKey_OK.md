---
layout: post
title:  "nezha上手按键key"
date:   2021-8-6
categories: nezha C
tags: C Linux nezha Key
excerpt: 哪吒外设上手Key
--- 



# 板载按键输入demo

上一节我们给开发板"驱动了眼睛"，聪明的你是不是已经想到脱离了电脑的控制我们的开发板应该怎么工作呢？那么本章节将带你使用开发板上的OK按键。

## 硬件

OK？这是什么，相信细心的小伙伴已经发现了除了在板子烧录的位置有一个fel按键，还有一个OK按键。它在这里~

![271FF782-F04D-4a45-A2B5-FE18F2DE6E28](img\271FF782-F04D-4a45-A2B5-FE18F2DE6E28.png)

那么我们就来看看它是怎么使用吧！

关于开发板的 硬件，大家还是去全志客户服务中心里 ，因为Tina固件默认已经配置了他的驱动。如果你想了解按键的具体配置或者配置自己的按键请移步[全志客户服务中心](https://open.allwinnertech.com/)查看基础组件开发文档。

在这里我们默认还是使用的Tina官方固件。此时我们查看系统设备。

![5457D658-D9B8-4feb-910D-B2DC4C2012F0](img\5457D658-D9B8-4feb-910D-B2DC4C2012F0.png)

可以看到有两个输入设备，**event0**和**event1**。

event0所对应的输入设备就是我们的OK按键。下面我们就通过读取这个文件来完成对按键状态的读取吧。

## 程序

老套路：我们直接下载代码

## 程序编译

因为编译不需要依赖任何库文件，所以直接：

```
riscv64-unknown-linux-gnu-gcc key_ok.c -o key_ok
```

## 文件传输

此时就会生成`key_ok`可执行文件。我们把它传入开发板中。

```
adb push key_ok /
```

## 程序运行

然后在开发板中执行

```
root@TinaLinux:/# chmod +x key_ok
root@TinaLinux:/# ./key_ok 2
```

![C02BF3F4-5960-4ae2-BE4F-D844D0894F43](img\C02BF3F4-5960-4ae2-BE4F-D844D0894F43.png)

当我们按下按键时，他会打印当前的键号。是不是很简单呢（大笑）

## 进阶：开发板的按键程序及按键杂谈

### 1.关于按键

哪吒开发板上的按键是接在ADC端口上的，可能有些人就会问，ADC上的端口不是会影响效率吗，或者说很多余？

我要大声的说NO！！！

由于没有按键按下时当前端口默认是接VCC的，当有按键按下时，此时IO口为两个电阻的分压。（好像还是需要原理图 打脸打脸）



![50FCC181-2887-4abd-B988-53EFBF926AB1](img\50FCC181-2887-4abd-B988-53EFBF926AB1.png)

因为使用了ADC 那么可以在按键后继续增加按键和电阻，就可以实现识别多个按键，因为ADC精度有限，所以这里推荐在8个按键以内哈。比如下面接两个：

![B5930579-782A-4634-9AC5-6E9493E0002B](img\B5930579-782A-4634-9AC5-6E9493E0002B.png)

相关的内核配置可以参考 文档：[KEY开发文档](https://open.allwinnertech.com/#/doc?menuID=2&projectId=264)

### 2.关于按键的软件

软件上我们使用查询方式读取按键的键值。

其中使用到了Linux 中一个非常重要的结构体。

input_event

其结构体定义如下：

```C
struct input_event {
struct timeval time;
__u16 type;
__u16 code;
__s32 value;
};
```

```C
//type: 设备类型。可以设置为：
#define EV_SYN          0x00    表示设备支持所有的事件
#define EV_KEY          0x01    键盘或者按键，表示一个键码  
#define EV_REL          0x02    鼠标设备，表示一个相对的光标位置结果
#define EV_ABS          0x03    手写板产生的值，其是一个绝对整数值
#define EV_MSC          0x04    其他类型
#define EV_LED          0x11    LED灯设备
#define EV_SND          0x12    蜂鸣器，输入声音
#define EV_REP          0x14    允许重复按键类型
#define EV_PWR          0x16    电源管理事件
#define EV_FF_STATUS 0x17
#define EV_MAX 0x1f
#define EV_CNT (EV_MAX+1)
code:根据Type的不同而含义不同。
例如：
Type为EV_KEY时，code表示键盘code或者鼠标Button值。
取值范围：
#define EV_SYN 0x00
到：
#define KEY_MIN_INTERESTING KEY_MUTE
#define KEY_MAX 0x2ff
#define KEY_CNT (KEY_MAX+1)
//Type为EV_REL时，code表示操作的是哪个坐标轴，如：REL_X，REL_Y。(因为鼠标有x,y两个轴向，所以一次鼠标移动，会产生两个input_event)
//取值范围：
#define REL_X 0x00
#define REL_Y 0x01
#define REL_Z 0x02
#define REL_RX 0x03
#define REL_RY 0x04
#define REL_RZ 0x05
#define REL_HWHEEL 0x06
#define REL_DIAL 0x07
#define REL_WHEEL 0x08
#define REL_MISC 0x09
#define REL_MAX 0x0f
#define REL_CNT (REL_MAX+1)
//Type为EV_ABS时，code表示绝对坐标轴向。
//value:根据Type的不同而含义不同。
例如：
Type为EV_KEY时，value: 0表示按键抬起。1表示按键按下。（4表示持续按下等？）。
Type为EV_REL时，value:　表明移动的值和方向（正负值）。
Type为EV_ABS时，code表示绝对位置。
```

所以我们的代码，通过判断当前如是节点是不是`EV_KEY`

然后再去判断` value `是否为1 当为KEY 1 时，说明按键按下。

此时打印` code `就是按键的键号。可以看到哪吒上按键的键号为：352。

```C
#include <stdio.h>  
#include <linux/input.h>  
#include <stdlib.h>  
#include <sys/types.h>  
#include <sys/stat.h>  
#include <fcntl.h>  
#include <unistd.h>

#define DEV_PATH "/dev/input/event0"   //difference is possible  

int main(int argc, char *argv[])  
{  
    int keys_fd;  
    char ret[2];  
    struct input_event t;
    if(argc <2)
    {
    printf("input device error\n");
    return -1;
    }  
    keys_fd=open(DEV_PATH, O_RDONLY);  
    if(keys_fd <= 0)  
    {  
        printf("open /dev/input/event2 device error!\n");  
        return -1;  
    }

    while(1)  
    {  
        if(read(keys_fd, &t, sizeof(t)) == sizeof(t))  
        { 
            //表示当前设备为  键盘设备 
            if(t.type==EV_KEY )  
                if( t.value==1)  //按键按下为1 松开为0
                {  
                    printf("key %d \n", t.code);  
                    if(t.code == KEY_ESC)  
                        break;  
                }  

       }  
    }  
    close(keys_fd);  
    return 0;  
} 
```

### 3.关于Linux按键的驱动方式

上面我们的软件是通过查询方式来进行按键的检测。除了此方式外，还有其他方式，如：

* 查询方式

* 休眠-唤醒
* poll
* 异步通知

对嵌入式应用开发感兴趣的朋友可以移步教程哦~

参考资料: https://www.cnblogs.com/amanlikethis/p/6915485.html 　　　

《嵌入式linux应用开发完全手册V2.3_韦东山》