---
layout: post
title: "简单的递归和动态规划"
date: 2014-04-05 13:31:50 +0800
comments: true
categories: 
- Algorithm
---

###题目：WordBreak


首先定义如下表示

**str[n:m] **为字符串 str 从下标 n 开始到下标 m-1 的字串

例如 str = "abcdef" str[4:6] = "ef"，str[1:3] = "bc"，str[3:4] = "d"

**str[n:] **为字符串从下标 n 开始直到字符串结束的子串

例如 str = "abcdef" str[2:] = ”cdef“

**str[:m] **为字符串从头开始直到下标 m-1 的子串

例如 str = "abcdef" str[:3] = ”abc“

**stat[n] **表示子串 str[n:] 能不能用字典中的单词表示


<!-- more -->

假设字符串长度是 L，那么字符下标从 0 到 L-1，定义 stat[L] = True （因为 str[L:] 是空字符串）

这道题的目标是计算 stat[0]，为了计算 stat[n]，我们有如下**递归方程**



    stat[n] = 
    { dict.contains( str[n:n+1] ) && stat[n+1] } ||
    { dict.contains( str[n:n+2] ) && stat[n+2] } ||
    ...
    { dict.contains( str[n:L] ) && stat[L] }



其中花括号括起来的部分 `dict.contains( str[n:n+m] ) && stat[n+m]` 表示：子串 str[n:] 的前 m 个字符组成的子字符串是字典中的单词，并且，子串 str[n+m:] 可以被字典中的单词组成



只要有任意一个 m 满足上述语句，就可以说 stat[n] 为 True！



对于**递归思想**来说，我们**从等式左边到右边**，不去管函数具体怎么解决子问题的，只是**给出如何把问题分解为更小的子问题的方法**。

所以一个递归方法去解这道题就是：
``` python
bool stat(n) {
    if n = L { return True }

    for i = n+1..L {
        #依次计算每一个花括号，任何一个为 True 就返回 True
        if dict.contains(str[n:i]) && stat(i) 
            return True
    }

    #所有花括号都是 False，返回 False
    return False
}
```

这跟你写的那个递归方法是一样的，只不过我参数里用的并不是一个真正的字符串，而是用一个数字表示当前子字符串是从那个下标开始的。



递归方法的一个问题就是**重复计算**，比如 str="aaaaab", dict = ["a", "aa", "aaa", "aaaa", "aaaaa"] 这个问题，当你计算 stat("aaaaab") 和 stat("aaaab") 的时候都要去计算 stat("aaab")，字符串越长，递归调用层数越深，这个问题越明显。



如何解决重复计算的问题，一个直观的方法就是，每当我计算完一个 stat(n) 的值的时候，我把 stat(n) 缓存下来，下次再需要计算 stat(n) 的时候我直接返回，而不去计算。这种思想用递归写是这样的：

``` python
boole stat_cache[L]; #缓存 stat[n] 的结果
boole stat_calced[L]; #标识 stat[n] 是否已计算
bool stat(n) {
    if n = L { return True }

    if stat_calced[n] { return stat_cache[n] }

    stat_calced[n] = True

    for i = n+1..L {
        #依次计算每一个花括号，任何一个为 True 就返回 True
        if dict.contains(str[n:i]) && stat(i) 
            stat_cache[n] = True
            return True
    }

    #所有花括号都是 False，返回 False
    stat_cache[n] = False
    return False
}
```


我没有试过，这种方法应该已经可以解决超时的问题了，但是这种方法还是会出现多层函数调用栈，而且需要一个额外的 stat_calced 数组，怎么办嘞？



我们再去看看那个递归方程，发现要计算 stat[n] 的值，我们需要知道 stat[n+1]...stat[L] 的值，这次我们换个方向思考，从**等式右边到左边**，先把右边需要用到的都计算好，然后再计算 stat[n]，先尝试解决子问题，进而解决更大的问题，这就是**动态规划的方法**，在这里，递归方程被称为**状态转移方程**。



已知计算 stat[n] 需要 stat[n+1]..stat[L]，而计算 stat[n+1] 需要 stat[n+2]..stat[L]，一步步退下来，我们从 stat[L] 开始，一步步往前算。当然，递归也好，动态规划也好，都需要一个**根**，就像数学归纳法中的 base 一样，这一题中我们的根就是 stat[L] = True



所以我们的代码应该是这个样子的：

``` python
for n = L-1 .. 0 { #loop1
    #假设所有花括号都是 False
    stat[n] = False
    for i = n+1..L { #loop2
        #依次计算每一个花括号，任何一个为 True 就返回 True
        if dict.contains(str[n:i]) && stat(i) 
            stat[n] = True
            break
    }
}
```


注意在上述代码中，loop1 相当于递归方法中的函数调用，都是对 n 做轮询，不同的是递归是从 0 到 L-1 轮询，而这里是从 L-1 到 0 来轮询。而 loop2 和递归函数中的循环是一模一样的。



总结一下，递归的思想是这样的，我要算 stat[n]，那我就直接开始算 stat[n]（递归调用从stat(0)开始，直接尝试计算 stat[0] ），算到算不下去了（发现计算 stat[0] 需要 stat[1] ），那就把当前的执行压栈，然后去算 stat[1]。而动态规划的思想是这样的，我先观察，发现 stat[0] 需要 stat[1]，而 stat[1] 又需要 stat[2]，最终发现 stat[L] 不依赖任何人，直接是 True，那么这时候知道了 stat[L] 我就可以算 stat[L-1]，进而可以算 stat[L-2]，然后一步步算出 stat[0]。
