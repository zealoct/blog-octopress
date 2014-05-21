---
layout: post
title: "如何写 Ubuntu 的 Framebuffer"
date: 2014-05-20 20:26:25 +0800
comments: true
categories: 
- Ubuntu
- Linux
- Android
- Framebuffer
---

最近有个小项目，想在 Android 上跑一个通过直接读写 framebuffer 实现的 vnc 客户端，
所以发现了这个 [fbvnc](https://github.com/zohead/fbvnc)， 是 github 上一个小哥儿捣鼓的，基于现有的一个同名项目开发，专为嵌入式设备使用。这个小的 vnc 客户端的不足当然有很多，比如连基本的窗口都木有，直接占用了你整个 framebuffer，不能调整分辨率，巨慢无比，卡的紧了就直接挂，但是它有一个最大的优点，就是真的非常简单，除了一些基本的 Linux 库之外没有任何第三方的依赖。

以上算是个小广告吧 (=

但是一个很大的问题是，这货在 ubuntu 上不 work……什么原因呢，做个小测试看一看。

<!-- more -->
## Ubuntu 下修改 Framebuffer

其实之前我写过小程序测试直接写 ubuntu 下的 [framebuffer](https://wiki.ubuntu.com/FrameBuffer) 的，当时也是神马效果都木有，当时只是猜测和 x11 或 unity 有关，也没深究，这次为了跑 fbvnc，特意去搜了下，后来在 [这里](http://unix.stackexchange.com/questions/58420/writes-to-framebuffer-dev-fb0-do-not-seem-to-change-graphics-screen) 找到了**解决方法**：需要置上 `FB_ACTIVATE_NOW` 和 `FB_ACTIVATE_FORCE` 属性，具体代码如下：

``` c
vinfo.activate |= FB_ACTIVATE_NOW | FB_ACTIVATE_FORCE;
ioctl(fbfd, FBIOPUT_VSCREENINFO, &vinfo)
```

加上这段代码之后，我写的测试程序终于可以看到修改屏幕的效果了！

``` c
#include <unistd.h>
#include <stdio.h>
#include <fcntl.h>
#include <linux/fb.h>
#include <sys/mman.h>

int main()
{
    int fbfd = 0;
    struct fb_var_screeninfo vinfo;
    struct fb_fix_screeninfo finfo;
    long int screensize = 0;
    char *fbp = 0;
    int x = 0, y = 0，color = 255;
    long int location = 0;

    /* 打开 fb 设备文件 */
    fbfd = open("/dev/fb0", O_RDWR);
    ioctl(fbfd, FBIOGET_FSCREENINFO, &finfo);
    ioctl(fbfd, FBIOGET_VSCREENINFO, &vinfo);
    /* 把 fb 映射到内存 */
    screensize = vinfo.xres * vinfo.yres * vinfo.bits_per_pixel / 8;
    fbp = (char *)mmap(0, screensize, PROT_READ | PROT_WRITE, MAP_SHARED,fbfd, 0);
    /* 置上FB_ACTIVATE_NOW 和 FB_ACTIVATE_FORCE */
    vinfo.activate |= FB_ACTIVATE_NOW | FB_ACTIVATE_FORCE;
    ioctl(fbfd, FBIOPUT_VSCREENINFO, &vinfo);
    /* 渐变修改 fb */
    for(color = 255; color > 0 ; color --) {
        for(x = 100 ;x < 200 ;x++) {
            for(y = 100; y < 200; y++) {
                location = (x+vinfo.xoffset) * (vinfo.bits_per_pixel/8) + (y+vinfo.yoffset) * finfo.line_length;
                *(fbp + location) = color; /* B */
                *(fbp + location + 1) = 0; /* G */
                *(fbp + location + 2) = 0; /* R */
                *(fbp + location + 3) = 0; /* A */
            }
        }

        usleep(5000);
    }    
    munmap(fbp, screensize);
    close(fbfd);
    return 0;
}
```

不过我预期中的效果是直接在当前屏幕上多出一个渐变的蓝色色块，实际效果是在一个纯黑背景上。看来还是和 x11 的实现有关，不过我不了解 x11，所以并不清楚具体的原因是什么，仿佛 x11 并没有这么简单的使用 fb。

除此之外，其实还有**另一个**更加方便和人畜无害的方法去直接操作 framebuffer，那就是切到其他的 tty 去执行。

## Ubuntu 下运行 fbvnc
根据 fbvnc 的 Readme，在 Ubuntu 下运行需要修改 fbvnc.c 下的

	typedef unsigned short fbval_t;

为

	typedef unsigned int fbval_t;

在 fbvnc 的源码中加入了修改 activate 属性的代码之后，执行成功！

等等，怎么退出……不得已 ssh 上去强制 kill 了 fbvnc 进程，结果我擦整个桌面都不好了，完全黑屏没有反映啊，切换到 tty6 关闭了很多工作的 tmux 窗口之后，回到 tty7 发现又好了，果然会有奇怪的问题，怪不得大家建议在 Linux 下不要直接修改 framebuffer，而是利用 X window 接口。

如果直接在 tty6 中执行 fbvnc 就正常多了，可惜性能实在太差，几乎不能用，而且还容易挂。

## Android 下编译运行 fbvnc

简单写个 Android 应用然后把 fbvnc 代码拷进去这种方法肯定不够，普通应用没有操作 framebuffer 的权限。我把 fbvnc 放到了 Android 源码 external 目录下，然后重新编译了 android 镜像。为了成功编译，需要做如下修改：

1. 注释掉 fbvnc.c 中所有的 dprintf，bionic 不支持此函数；
1. 在 fbvnc.c 中把调用 getpass 函数那一行改成硬编码，或者自己实现个 getpass，bionic 也不支持此函数；
1. 修改 draw.h 中的 FBDEV_PATH 为 "/dev/graphics/fb0"，Android 中 fb 设备路径和 Linux 默认路径不同；
1. 在 fbvnc/ 下添加如下文件

``` text Android.mk
LOCAL_PATH:= $(call my-dir)

include $(CLEAR_VARS)
LOCAL_CFLAGS:= -Wall -Os
LOCAL_MODULE_TAGS:= debug eng
LOCAL_MODULE:= fbvnc
LOCAL_SRC_FILES:= d3des.c draw.c vncauth.c fbvnc.c
LOCAL_C_INCLUDES := $(LOCAL_PATH)
LOCAL_SHARED_LIBRARIES := \
        libcutils
include $(BUILD_EXECUTABLE)
```

OK，编译！好了之后 adb shell 上去，用 root 权限执行 
	
	fbenv myhostname

成功显示了远程 vnc 桌面，不过并不持久，很快就会被 Android 自己的界面刷掉。如果要解决这个问题，需要对 Android 系统进行更多的修改，以后有时间再写一篇吧。