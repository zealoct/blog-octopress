---
layout: post
title: "printf_in_c"
date: 2014-04-04 21:48:15 +0800
comments: true
published: false
categories: 
---

这两天讨论起 c 中`*p++`的行为，特意写了个小程序想要验证一下：

``` c
main () {
	int a[3] = {0xdeaa, 0xdeab, 0xdeac};
	int *p = a;
	int *p1 = a;
	int *p2 = a;
	
	printf("%x, %x, %x\n",  (int)(*p++), (int)(*(p1++)), (int)(++(*p2)));
}
```

<!-- more -->

我期待的输出结果是这样的：

	0xdeaa, 0xdeaa, 0xdeab

然而实际的输出却是这样的：

	0xdeab, 0xdeab, 0xdeaa

完完全全相反啊妈蛋！我心目中的`*p++`应该是先执行`p++`，然后返回未增加之前 p 的值 pp，之后对 pp 进行解引用，最终返回`*pp`的值，而 p 应该等于 pp+4。执行效果等价于`(*p); p++`，但是从输出结果看来仿佛是`p++; (*p)`，肯定是哪里不对。

观察发现 a[0]+1 的值和 a[1] 相等，都是 0xdeab，因此事实上并不能知道输出的 0xdeab 到底是怎么来的，所以首先把数组 a 的初始值改成

	int a[3] = {0xdea0, 0xdea5, 0xdeaa};

这次的输出是

	0xdea1, 0xdea1, 0xdea0

所以，看来并不如想像的那样，仿佛`*p++`被执行成了`(*p)++`了。但这是不对的，而且就算是后者，也应该是先返回 *p 的值，然后再加对 p 所指向的内存加 1。而且毫无疑问第一个输出`*p++`会影响第二个输出`*(p1++)`的结果，所以事实并不是这样。不如来看下汇编吧：

``` c
00000000004004f4 <main>:
  4004f4:   55                      push   %rbp
  4004f5:   48 89 e5                mov    %rsp,%rbp
  4004f8:   48 83 ec 30             sub    $0x30,%rsp
  4004fc:   c7 45 d0 a0 de 00 00    movl   $0xdea0,-0x30(%rbp) // a[0]
  400503:   c7 45 d4 a5 de 00 00    movl   $0xdea5,-0x2c(%rbp) // a[1]
  40050a:   c7 45 d8 aa de 00 00    movl   $0xdeaa,-0x28(%rbp) // a[2]
  400511:   48 8d 45 d0             lea    -0x30(%rbp),%rax    
  400515:   48 89 45 f8             mov    %rax,-0x8(%rbp)     // p
  400519:   48 8d 45 d0             lea    -0x30(%rbp),%rax    
  40051d:   48 89 45 f0             mov    %rax,-0x10(%rbp)    // p1
  400521:   48 8d 45 d0             lea    -0x30(%rbp),%rax
  400525:   48 89 45 e8             mov    %rax,-0x18(%rbp)    // p2
  400529:   48 8b 45 e8             mov    -0x18(%rbp),%rax    // rax = p2
  40052d:   8b 00                   mov    (%rax),%eax         // eax = *p2 = 0xdea0
  40052f:   8d 50 01                lea    0x1(%rax),%edx      // edx = eax + 1
  400532:   48 8b 45 e8             mov    -0x18(%rbp),%rax    // rax = p2
  400536:   89 10                   mov    %edx,(%rax)
  400538:   48 8b 45 e8             mov    -0x18(%rbp),%rax
  40053c:   8b 08                   mov    (%rax),%ecx
  40053e:   48 8b 45 f0             mov    -0x10(%rbp),%rax
  400542:   8b 10                   mov    (%rax),%edx
  400544:   48 83 45 f0 04          addq   $0x4,-0x10(%rbp)
  400549:   48 8b 45 f8             mov    -0x8(%rbp),%rax
  40054d:   8b 00                   mov    (%rax),%eax
  40054f:   48 83 45 f8 04          addq   $0x4,-0x8(%rbp)
  400554:   89 c6                   mov    %eax,%esi
  400556:   bf 5c 06 40 00          mov    $0x40065c,%edi
  40055b:   b8 00 00 00 00          mov    $0x0,%eax
  400560:   e8 8b fe ff ff          callq  4003f0 <printf@plt>
  400561:   c9                      leaveq 
  400562:   c3                      retq   
  400563:   90                      nop
```