---
layout: post
title: "Windows 下使用 Cocos2d-x 3.x + Lua 开发基础"
date: 2014-07-06 10:59:17 -0700
comments: true
categories: 
- Cocos2d-x
- Lua 
---

[Cocos2d-x](http://cocos2d-x.org/) 是国内非常成熟的 2d 游戏开发引擎了，拥有非常广大的开发者群体和活跃的社区，支持 Windows、Linux、Mac OS、iOS、Android 和 Window Phone 等各个平台，其本身是用 C++ 语言实现的，主要提供了面向 C++ 的接口，也提供了对 Lua、object-c 和 JavaScript 的支持。我们的 minigame 要基于 Cocos（为简单起见，我就把 Cocos2d-x 称为 Cocos 了） 和 Lua 实现，这有好处也有弊端，Lua 作为一门脚本语言，开发起来确实非常快捷，不过 Cocos 社区 Lua 相关的文档和资料并不如 C++ 那么完备，使得上手的难度增加，而且 Lua 本质并不是面向对象的编程语言，我们编码时的一些 OO 的思想也要需要转变。

出于对自己记忆力的极端不信任，决定把实习期遇到的关于 Cocos + Lua 的种种问题都记录下来，希望也能成为实习的成果之一吧！

<!-- more -->



开发系统和工具
------
首先是环境的搭建，Cocos 的使用方法挺多的，取决于你的开发系统和开发工具，刚开始看的时候我自己也有些被搞乱了，说来搞笑，我基本上按照 Cocos 的安装说明文档去配置的，到最后用的最多的却是 Cocos-console。官方网站上各种开发工具环境搭建相关的文档还是很详细的，我在这里就不再重复了，主要是简单介绍一下用到的各种工具是干嘛的，还有就是不同的 Cocos 开发环境如何抉择的问题。

我自己的系统是 Windows 7，目标平台是 Android，因此本文主要讨论的问题是 **Windows 平台下开发 Android 游戏的环境选择**。官方网站上有比较详细的搭建 Cocos + Android 开发环境的文档，可以参考 [How to run cpp-tests on Android (Terminal)](http://cocos2d-x.org/wiki/How_to_run_cpp-tests_on_Android) 和 [How to run cpp-tests on Android (Eclipse)](http://cocos2d-x.org/wiki/How_to_Build_an_Android_Project_with_Eclipse) 这两份文档，分别是在命令行下和 Eclipse 下开发 Android 游戏的环境配置步骤。总的来说，要在 Windows 上开发 Cocos + Lua 的 Android 游戏，我们需要准备如下东西，有点多，附上对每个工具的简单解释：

- **Cocos2d-x** 也就是 Cocos 的源代码，包含 Cocos 用到的各种脚本文件、 C++ 类、各平台工程文件等，可从 [此处](http://cocos2d-x.org/download) 下载，我选择的是 3.1.1 的版本，需要说明的一点是 Cocos 的 API 接口在一个大版本中是不会变的，也就是 3.x 的版本共享一套 API，但和 2.x 的 API 差别较大，目前我在网上找到资料 2.x 的比较多

- **Visual Studio** 如果是 Windows 开发环境的话 VS 是一定少不了的，编译 Cocos 的源代码就全依赖它了，这个木有源码链接，请大家自行搜索吧……

- **JDK 1.6+** Java 虚拟机，可以从 [Oracle 官网](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 上下载

- **ADT Bundle** 是一个集成了 Android SDK + Eclipse + ADT（Eclipse 开发 Android 使用到的插件）的压缩包，使得配置 Android 开发环境变得非常方便，可从 [Android Developer](http://developer.android.com/sdk/index.html) 下载

- **Android NDK**，提供使用 Native Code 开发 Android 应用程序的支持，可从 [Android Developer](http://developer.android.com/tools/sdk/ndk/index.html) 下载

- **Ant 1.9.4** 根据 build.xml 编译 Android 项目的命令行工具，可从 [apache.org](http://ant.apache.org/bindownload.cgi) 下载

- **Python 2.7** 脚本语言，Cocos 的安装、运行脚本是用 Python 写的，地址 [python.org](https://www.python.org/download/)

- **Lua 5.2** Lua 脚本虚拟机，可执行程序下载地址 [Lua 5.2.3](http://joedf.users.sourceforge.net/luabuilds/)



Cocos 项目结构
-----

分两个部分说下，一个是 Cocos C++，一个是 Cocos Lua。

Cocos C++ 项目根目录如下：

![](/images/blog/cocos-basic-cpp.png)

其中 `/Classes` 文件夹是主要的代码文件夹，开发者所需要写的全部代码文件都包含在这个文件夹中，`/Resources` 则是资源文件夹，开发者的资源都存储在这个文件夹中，`/cocos2d` 文件夹则包扩了 Cocos 完整的源代码。除此之外我们看到还有大量的 `/proj.xxxx` 文件夹，这是 Cocos 为不同平台、不同编辑器生成的项目文件夹，所有这些项目均使用 `/Classes` 中的源代码和 `/Resources` 中的资源文件。

而 Cocos Lua 项目目录如下：

![](/images/blog/cocos-basic-lua.png)

与 C++ 项目结构类似，Lua 项目所有的程序文件都在 `/src` 文件夹下，资源文件都在 `/res` 文件夹下，`/frameworks` 包含了 Cocos 的完整源代码以及各个项目文件夹， 如下图所示为 `/frameworks/runtime-src` 文件夹的结构：

![](/images/blog/cocos-basic-lua-runtime-src.png)

而 `/runtime` 文件夹内则包含了生成的个平台的可执行代码。



Console vs. Eclipse vs. Visual Studio
-----

这是个让人比较纠结的话题，我们到底应该配置哪种环境，使用哪种工具进行 Cocos 开发呢？其实根据上一节的介绍，大家应该已经明白了，Cocos 项目可以**方便的在各个工具之间切换**，你可以使用 Visual Studio 写 C++ 代码，生成 Windows 程序进行调试，然后跑到 Eclipse 里去生成一个 APK 部署到自己的手机上。所以大家对工具其实不用太在意，用自己喜欢的就行。

目前我知道的可选方案有四种，简单介绍一下他们用法上的区别，以及我对这些方法的感受。

### Cocos Console
纯粹使用命令行来进行 Cocos 项目的新建、编译、运行等操作，代码使用纯文本编辑器比如 Sublime Text 编写。这种方法比较轻量级，不需要运行 VS 或 Eclipse 等消耗系统资源的编辑器，可以更方便的认识整个项目的架构，了解每个文件夹是干嘛的，不论是 C++ 项目还是 Lua 项目，这种模式都可以胜任，Linux 程序员的首选；缺点就是没有代码提示，也不是很方便调试，上手写代码比较困难，编码效率不够高。

### Cocos + Eclipse
使用 ADT 可方便的对 Android 游戏进行开发、部署与调试，熟悉 ADT 的 Android 开发人员的首选，开发 Cocos + C++ 会比较方便，相应的写 Lua 脚本就不很方便，需要另装插件。

### Cocos + Visual Studio
Visual Studio 真是 Windows 程序员的专有福利，其功能简直不能更强大，使用 Visual Studio 开发 C++ 简直是件幸福的事，如果要用 Cocos + C++ 开发 Win32 游戏，那 Visual Studio 绝对算是首选了。要说美中不足，就是写 Lua 需要另装插件，而且如果要生成 Android 游戏需要使用 Eclipse 或 Console 去编译部署。

### Cocos Code IDE
[Cocos Code Ide](http://www.cocos2d-x.org/wiki/Code_Editor) 这个工具刚才我没有列在必备工具里，因为其本身只提供了 Cocos + Lua/JS 的开发环境，也是基于 Eclipse 包装的，针对 Lua 语言提供了 Cocos API 的**代码提示**功能，这一点上来看非常方便，而且开发环境搭建起来也非常方便，不过我试用的时候 debug 信息老是打不出来……另外需要注意的是，用 Cocos Code IDE 生成的项目文件夹跟 Console 生成的 Cocos Lua 项目文件夹稍有不同，并没有包含完成的 Cocos 源文件，因此大小要小很多（大概60+MB），而且在 `runtime-src` 中只包含了 Eclipse 项目。


总结一下，我觉得如果使用 C++，目标平台 Windows，那么 Visual Studio 无疑；如果目标平台是 Android，也可以使用 Visual Studio 进行编码和调试（生成 exe 进行调试），最后使用 Eclipse 部署；对于 Lua 开发者，Code IDE 是个不错的选择，支持断点、单步、watch 等调试手段，而且自动生成各平台的二进制文件，的缺点就是目前还有 bug，如果打印语句没有问题的话就好了，另外一点就是 Code IDE 完全不能写任何 C++ 代码，对于项目中既需要写 C++ 也需要写 Lua 的高级开发者而言就显得不够灵活了；最后如果你之前是 Linux 开发者或是希望对项目有更直接更灵活的控制，请选择命令行。


命令行 + 文本编辑器开发
-----

前边多次提到我自己使用的是 Cocos Console + Sublime Text 来编写 Cocos 项目的，这里就谈一些使用上的经验技巧吧！一开始之所以选择 Console 是因为安装部署起来稍微方便一点点，后来我发现还有另一个好处（以至于现在连 Console 都不用了）。


### 创建项目

使用如下命令：

	cocos new MyCCLua -p com.test.hjc -l lua -d ./
	
其中 `MyCCLua` 是项目名，`-p com.test.hjc` 指定生成的 Android 项目的包名，`-l lua` 指定生成项目的语言种类，合法项包括 `cpp`，`lua` 和 `js`，`-d ./` 是新建项目所在文件夹，默认是当前文件夹。

### 编译/运行项目

使用如下命令：

	cocos compile/run -p platform

其中 `compile` 是单纯编译，`run` 是编译且运行，`-p win32` 是编译生成的目标平台可选项为 ios、mac、android、web 以及 win32，`-s source` 项目文件夹路径，如不指定则为当前目录，`-q` 安静模式，不输出日志系信息，`-m mode` 控制编译生成模式，可以是 `debug` 或 `release`，默认是 `debug` 模式。

如果要编译运行 Cocos 自带的 lua-tests，可以直接去 `cocos2d-x-3.1.1/tests/lua-tests` 下执行：

	cocos run -p win32

### 编写与调试
我的目标平台是 Android，不过平时都是在 Windows 上进行调试，部署、调试起来会方便很多，对于 Lua 开发者而言，还有一个好处就是 Lua 作为一门脚本语言，代码修改之后实际上是**不需要重新编译**的，**重启 exe 就可以看到最新的代码效果**了！

我平时编写、调试的步骤为：

1. 新建一个名为 `MyCCLua` Cocos Lua 项目之后，使用 `cocos compile -p win32` 编译此新建项目，完成后可以在 `MyCCLua/runtime` 下找到一个名为 `win32` 的文件夹，此文件夹包含了编译生成的 Windows 可执行程序、所有的资源文件和所有用到的 Lua 脚本；
2. 把 `MyCCLua/runtime/win32` 文件夹单独拷贝出来，命名为 `MyCCLua-win32`；
3. 使用 Sublime Text 修改 `MyCCLua-win32/src` 下的 Lua 脚本，执行 `MyCCLua-win32/MyCCLua.exe` 查看最新效果，由于免去了事实上无用的编译环节，所以可以加快调试的速度；
4. 如果脚本产生错误闪退，则使用 [Decoda](http://unknownworlds.com/decoda/) 工具加载 `MyCCLua.exe`，该工具可以捕获 Lua 异常，设置断点和 watch，是我调试过程中不可或缺的利器；
5. 如果需要生成 APK，则把 `MyCCLua-win32` 下的 `res` 和 `src` （取决于你做了哪些修改）两个文件夹拷回到 `MyCCLua` 内，执行 `cocos compile -p android`
