---
layout: post
title: "6.824 Lab1 MapReduce Notes"
date: 2014-04-09 22:01:37 +0800
comments: true
published: false
categories: 
- Exercises
- Distributed Systems
---

作为 MIT 6.824 *分布式系统* 这门课的第一个 Lab，其主要的目的是让学生熟悉下 Go 语言，因此不是特别难。事实上，2013 年的 Lab1 是做一个 P/B 架构的 LockService，感觉今年的这个 MapReduce 难度上有所下降，更加适合上手，而内容方面涉及了 MapReduce 的简单应用和实现，做完之后收获更大，总的来说整体水准高于去年的 LockService。

Lab 共分为三个部分，Part I 要求在 MapReduce 上实现一个 WordCount，因为大部分的 MapReduce 框架都已经实现好了，所以这部分非常简单；支持代码（ Support code ）中的 MapReduce 是个非常简化的版本，没有 Master，顺序执行每一个 Map 和 Reduce 操作，Part II 的要求是实现一个简单的 Master； Part III 要在 Part II 的基础之上实现容忍客户端失败的情况。

总的来讲，这个 Lab 中我们只需要实现一个简化版的 MapReduce 中的 Master Node 的逻辑部分。

### Part I

只有一点点代码要写，不过在动手之前还是要认真阅读支持代码，主要需要搞明白的是 Map 和 Reduce 函数的输入输出分别是什么：

``` go wc.go
func Map(value string) *list.List {}

func Reduce(key string, values *list.List) string {}
```

为此需要看一看 mapreduce.go 中 DoMap 和 DoReduce 这两个方法。