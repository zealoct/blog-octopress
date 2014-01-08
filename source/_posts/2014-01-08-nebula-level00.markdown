---
layout: post
title: "Exploit Exercises - Nebula Level00"
date: 2014-01-08 19:19:58 +0800
comments: true
categories: Security, Exercises
---

之前斌哥推荐了一个[Exploit Exercise](http://exploit-exercises.com/)，上面有一些小练习，最近决定去做一做玩玩。网站上练习的话分为以下几个子项目：

- Nebula，simple and intermediate challenges that cover Linux privilege escalation, common scripting language issues, and file system race conditions
- Protostar，basic memory corruption issues such as buffer overflows, format strings and heap exploitation under "old-style" Linux system that does not have any form of modern exploit mitigiation systems enabled
- Fusion, continues the memory corruption, format strings and heap exploitation but this time focusing on more advanced scenarios and modern protection systems

<!-- more -->

### 安装

去网站上[下载](http://exploit-exercises.com/download)Nebula的镜像，看很多人的评论之前提供的是OVA镜像，直接加载就行，现在提供的bin是liveCD格式的，有各种问题。我在安装的时候也出现了问题，在vmware中启动图形界面会报“piix4 smbus not enabled”的错误，看了看网上的各种解决方案基本上是在modprobe中禁用掉i2c_piix4这个module，不过对于安装过程中出问题没有找到更好地解决方法，反正这些练习也不需要在本地些什么代码，所以就选择直接从liveCD启动，然后terminal里去做了。

### Level 00

第一个练习非常简单，主要目的应该是让大家熟悉这个练习的一般过程吧。网站上对这一个练习的描述为我们可以找到一个设置用户ID的程序，运行这个程序我们就能以flag00这个用户来执行应用。

由于描述太简单了，第一次看的时候完全不知道该干啥，后来无意间在/bin目录下看到一个名为getflag的程序，在普通系统里没见过，试运行提示“getflag is executing on a non-flag account, this doesn't count”，这样一来就基本明白了，登录用户名是level00，而这个练习的目的是要用flag00这个用户执行getflag这个程序。所以题目的意思是系统当中有个应用可以让我们用来更改用户id，这个练习的目的就是找到这个程序并运行之！

开始的时候不太知道应该怎么搜索，先尝试匹配文件名


	$ find / -name "*set*id*" | vim -


搜出来好多结果，都没啥用，感觉这种方法不科学。分析下，感觉这个程序既然能切换到flag00这个用户，那么想着跟这个用户得有点儿关系吧，去home目录下看了一眼，发现/home/flag00这个目录的owner是flag00，但是owngroup却是level00，所以想到通过用户名和用户组去搜索命令


	$ find / -group "level00" | vim -


结果的第一行是


	/bin/.../flag00


flag00这个程序看起来非常可疑（因为自己的shell配置中...被alias成了../..，所以一度还错愕到/bin/../..是个啥目录……），尝试运行

``` bash
$ /bin/.../flag00
Congrats, now run getflag to get your flag!
$ getflag
You have successfully executed getflag on a target account
```

嗯！看来确实是这样，对比一下前后的uid

``` bash
$ id
uid=1001(level00) gid=1001(level00) groups=1001(level00)
$ /bin/.../flag00
Congrats, now run getflag to get your flag!
$ id
uid=999(flag00) gid=1001(level00) groups=999(flag00),1001(level00)
```

### 补充

objdump了一下flag00这个应用，发现代码实现基本是这样的

``` c
  gid_t gid;
  uid_t uid;
  gid = getegid();
  uid = geteuid();
  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);
  puts("some string");
  execve(arg1, arg2)
```

这段代码是main函数的，最后一句比较奇怪，传入的两个参数分别是0x10(%ebp)和0xc(%ebp)，看起来貌似是调用main函数的参数，但是main函数调用的时候应该没传参才对。不过这句无伤大雅，根据运行效果来看应该是启动了一个bash，关键是前边的setresgid和setresuid，整段程序的运行逻辑就是获得该程序的effective uid，然后设成当前的uid和gid。看下flag00这个程序的权限

``` bash
$ ll /bin/.../flag00
-rwsr-x--- 1 flag00 level00 7358 2011-11-20 21:22 flag00*
```

注意运行权限设置了set-user-ID位，也就是说flag00执行的时候effective uid为程序文件所属用户的uid，在本例中执行时effective uid就是999(flag00)。这是这个练习成功的重要基础，如果没有置上这一位，flag00执行时的effective uid就是当前用户level00，就无法达到获得flag00权限的目的。
