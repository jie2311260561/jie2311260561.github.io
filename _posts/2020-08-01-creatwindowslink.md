---
layout: post
title:  "缩减自己系统C盘"
date:   2021-8-2
categories: Windows Mklink
tags: Using Windows Tools
excerpt: 缩减C盘空间
--- 
# 由于自己C盘空间不是很够，所以想办法为自己的C盘缩减点空间

```
    mklink /j d:\recivefiles \\138.20.1.141\e$\recivefiles
```

# note1 C
```
    C:\Program Files (x86)\Tencent  E:\Program Files\Tencent

    mklink /j "C:\Program Files (x86)\Tencent" "E:\Program Files\Tencent"  

    成功创建   yes！ 
```

# nite 2 

```
mklink /j "C:\Users\23112\.huawei-liteos-studio" "E:\Users\23112\.huawei-liteos-studio"

mklink /j "C:\Users\23112\.stm32cubemx" "E:\Users\23112\.stm32cubemx"

mklink /j "C:\Users\23112\openSourceTools" "E:\Users\23112\openSourceTools"

```