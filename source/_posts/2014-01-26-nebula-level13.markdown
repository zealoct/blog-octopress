---
layout: post
title: "Exploit Exercises - Nebula Level 13"
date: 2014-01-26 21:54:12 +0800
comments: true
published: true
categories: 
- Security
- Exercises
- Nebula
---

### About

There is a security check that prevents the program from continuing execution if the user invoking it does not match a specific user id.

To do this level, log in as the *level13* account with the password *level13* . Files for this level can be found in /home/flag13.

<!-- more -->

### Source code

``` c 
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <sys/types.h>
#include <string.h>

#define FAKEUID 1000

int main(int argc, char **argv, char **envp)
{
	int c;
	char token[256];

	if(getuid() != FAKEUID) {
		printf("Security failure detected. UID %d started us, we expect %d\n", getuid(), FAKEUID);
		printf("The system administrators will be notified of this violation\n");
		exit(EXIT_FAILURE);
	}

	// snip, sorry :)

	printf("your token is %s\n", token);

}

```



### Solution

There is no way for me to be uid 1000, but this executable which contains the token is right here, we cannot be stopped by a simple `if` branch.

Disassemble the executable *flag13*

``` bash 
level13@nebula:~$ objdump -D /home/flag13/flag13 > /home/level13/flag13.asm
```

Go to the instruction corresponding to the last *printf()* in the c code, I thought I could read the password out directly from the memory location where *token* is stored. Turned out that *token* is calculated with mass of code. Well, as I cannot read the password directly, I could always change the execution flow and let the program print that out.


```text flag13.asm
080484c4 <main>:
 80484c4:       55                      push   %ebp
 80484c5:       89 e5                   mov    %esp,%ebp
 80484c7:       57                      push   %edi
 80484c8:       53                      push   %ebx
 80484c9:       83 e4 f0                and    $0xfffffff0,%esp
 80484cc:       81 ec 30 01 00 00       sub    $0x130,%esp
 80484d2:       8b 45 0c                mov    0xc(%ebp),%eax
 80484d5:       89 44 24 1c             mov    %eax,0x1c(%esp)
 80484d9:       8b 45 10                mov    0x10(%ebp),%eax
 80484dc:       89 44 24 18             mov    %eax,0x18(%esp)
 80484e0:       65 a1 14 00 00 00       mov    %gs:0x14,%eax
 80484e6:       89 84 24 2c 01 00 00    mov    %eax,0x12c(%esp)
 80484ed:       31 c0                   xor    %eax,%eax
 80484ef:       e8 cc fe ff ff          call   80483c0 <getuid@plt>
 80484f4:       3d e8 03 00 00          cmp    $0x3e8,%eax
 80484f9:       74 36                   je     8048531 <main+0x6d>
 80484fb:       e8 c0 fe ff ff          call   80483c0 <getuid@plt>
 8048500:       ba d0 86 04 08          mov    $0x80486d0,%edx
 8048505:       c7 44 24 08 e8 03 00    movl   $0x3e8,0x8(%esp)
...
```

This is the snippet of function *main()*, note that line 16 compare *%eax* (which is the return value of function call *getuid()*) with *0x3e8*, and line 17 will jump to memory location 0x8048531 if they are equal.

In a normal execution, these are apparantly not equal, but we could make it equal with *gdb*.

1. copy *flag13* into /home/level13
1. start it with *gdb*
1. set a break point at 0x80484f4, which is the instruction to compare
1. run the program
1. modify %eax to 1000 at the break point
1. continue run the program

``` bash
# start flag13 with gdb
level13@nebula:~$ gdb flag13
# set break point and run
(gdb) b *0x80484f4 
Breakpoint 1 at 0x80484f4
(gdb) run
Starting program: /home/level13/flag13
# reach break point, let's take a look at where we are
Breakpoint 1, 0x080484f4 in main ()
(gdb) disassemble
Dump of assembler code for function main:
   0x080484e0 <+28>:    mov    %gs:0x14,%eax
   0x080484e6 <+34>:    mov    %eax,0x12c(%esp)
   0x080484ed <+41>:    xor    %eax,%eax
   0x080484ef <+43>:    call   0x80483c0 <getuid@plt>
=> 0x080484f4 <+48>:    cmp    $0x3e8,%eax
   0x080484f9 <+53>:    je     0x8048531 <main+109>
   0x080484fb <+55>:    call   0x80483c0 <getuid@plt>
# here we print the registers out, %eax is 1014
(gdb) i r
eax            0x3f6    1014
ecx            0xbffff804       -1073743868
...
# change %eax
(gdb) set $eax=1000
(gdb) i r
eax            0x3e8    1000
ecx            0xbffff804       -1073743868
...
# continue execution 
(gdb) continue
Continuing.
your token is b705702b-76a8-42b0-8844-3adabbe5ac58
[Inferior 1 (process 31018) exited with code 063]
```
Now *su* to *flag13* with the token.

