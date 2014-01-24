---
layout: post
title: "Exploit Exercises - Nebula Level 08"
date: 2014-01-24 11:43:31 +0800
comments: true
categories: Exercises Nebula Security
---

### About

World readable files strike again. Check what that user was up to, and use it to log into flag08 account.

To do this level, log in as the *level08* account with the password *level08*. Files for this level can be found in /home/flag08. 

<!-- more -->

### Solution

用*level08*登录之后回看到一个名为*capture.pcap*的文件，记录了一次抓包的结果。把这个文件scp出来之后分析，中间有一处显示：

	Password: backdoor...00Rm8.ate

看样子是*flag08*用户的密码，不过用过十六进制编辑器的小伙伴们都知道，"."可能是任何空白字符，不能简单的认为密码就是“backdoor...00Rm8.ate”。查看这部分的十六进制，发现“.”对应的是0x7f，查了一下[ASCII码表](http://www.asciitable.com/)发现0x7f是DEL，也就是删除键，所以实际的密码应该是“backd00Rm8.ate”。

还挺会玩儿的，拿0替代了o……
