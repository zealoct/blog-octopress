---
layout: post
title: "Exploit Exercises - Nebula Level 06"
date: 2014-01-13 15:48:33 +0800
comments: true
categories: 
- Security
- Exercises
- Nebula
---

### About

The *flag06* account credentials came from a legacy unix system.

To do this level, log in as the *level06* account with the password *level06* . Files for this level can be found in /home/flag06.

<!-- more -->

Check out /etc/passwd

	flag06:ueqwOCnSGdsuM:993:993::/home/flag06:/bin/sh

Decrypted with John the Ripper, and got the login password. 

[reference](http://www.governmentsecurity.org/articles/crack-unix-linux-passwords.html)
