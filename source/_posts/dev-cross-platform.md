---
title: 跨平台应用程序开发方法大盘点（建议收藏备用）
date: 2020-03-18 23:43:00
categories:
- [开发,应用程序]
- [开发,应用软件]
tags:
- 开发
- 跨平台
---
>让自己开发的软件能够跨平台运行，既是每个软件开发者多年以来的梦想，也是许多软件开发者的噩梦。到今天为止，软件界在跨平台开发、运行应用程序方面已经取得了很大的进展，但离理想的目标还有很远的路要走。
# 1.什么是跨平台应用程序开发

当我们谈论"跨平台开发“、"跨平台运行“、"跨平台发布“等等之类的话题时，所提到的“平台”是指各类操作系统，既包括PC操作系统，如windows、mac os、linux，也包括移动设备操作系统（如苹果IOS,谷歌Android);"跨平台应用程序开发"则是指“开发可以运行在多个操作系统上的应用程序”，就如上面提到那些操作系统。

# 2.编译型语言的跨平台
在软件业早期，跨平台开发是比较麻烦的。早期的软件开发语言大多是编译型的，且编译出来的代码(BinaryCode)与操作系统密切相关，所以是无法跨平台运行的。

那么，编译型语言就没法跨平台了吗？当然是有办法的，既然在BinaryCode层次无法实现跨平台，那就在源代码（SourceCode)层次实现跨平台。

## 2.1c/c++跨平台开发方案

以c/c++语言为例，在不同的操作系统上，均有相应的编译器，开发人员在开发相应软件时，根据编译器中定义的预编译宏，对于代码中的部分系统API调用代码单独编写，然后通让特定平台的编译器去编译对应的代码。

听起来似乎一切都不错，只需要改少量源代码，然后用目标系统进行编译即可运行，事情真的这么简单吗？

对于用编译型语言开发的软件来说，如果软件是没有图形界面（GUI）的话,源代码改动工作量的确不大；但如果涉及GUI界面，问题就会比较复杂。当然，也有办法：不要使用与操作系统绑定的GUI接口或框架，例如windows下的MFC库；你应该使用一个跨平台的GUI库，例如QT,GTK。

## 2.2c/c++各平台预编译宏的判断示例

**需要注意几点：**

* windows32/64平台_WIN32都会被定义，而_WIN64只在64位windows上定义，因此要先判断_WIN64
* 所有的apple系统都会定义 __APPLE__，包括MacOSX和iOS
* TARGET_IPHONE_SIMULATOR 是 TARGET_OS_IPHONE 的子集，
TARGET_OS_IPHONE 是 TARGET_OS_MAC的子集。也就是说iOS模拟器上会同时定义这三个宏。因此判断的时候要先判断子集。
* 另外mac上可以用以下命令行获取GCC定义的预编译宏：
gcc -arch i386 -dM -E - < /dev/null | sort  （i386可替换为arm64等）


```c

#ifdef _WIN32
   //define something for Windows (32-bit and 64-bit, this part is common)
   #ifdef _WIN64
      //define something for Windows (64-bit only)
   #else
      //define something for Windows (32-bit only)
   #endif
#elif __APPLE__
    #include "TargetConditionals.h"
    #if TARGET_IPHONE_SIMULATOR
         // iOS Simulator
    #elif TARGET_OS_IPHONE
        // iOS device
    #elif TARGET_OS_MAC
        // Other kinds of Mac OS
    #else
    #   error "Unknown Apple platform"
    #endif
#elif __ANDROID__
    // android
#elif __linux__
    // linux
#elif __unix__ // all unices not caught above
    // Unix
#elif defined(_POSIX_VERSION)
    // POSIX
#else
#   error "Unknown compiler"
#endif
```
# 3.解释型编程语言的跨平台

解释型编程语言有良好的平台兼容性，它一边解释一边运行，在任何环境中都可以运行--前提是安装了解释器。灵活，修改代码的时候直接修改就可以，可以快速部署，不用停机维护。

解释型编程语言通常也是动态类型编程语言：

动态类型语言是指在运行期间才去做数据类型检查的语言，也就是说，在用动态类型的语言编程时，永远也不用给任何变量指定数据类型，该语言会在你第一次赋值给变量时在内部将数据类型记录下来，并且在你再次更改值类型时自动更改数据类型--换句说，变量类型是值类型决定的。

例如：php、python、Ruby等，这些语言在设计之初就考虑了在PC操作系统上的运行问题，分别为不同操作系统设计了对应的解释器，因此它们能够实现“写一份代码，在多个操作系统上运行“--移动端系统除外（IOS和Android等)。

# 4.基于虚拟机的伪编译型编程语言

典型代表语言：Java。

为什么说它是伪编译型呢?因为，用Java开发的代码，会编译成一个字节码的文件，它能够在安装Java虚拟机的操作系统上被执行，在执行字节码时则是解释执行。

如上所述，Java程序通过虚拟机实现跨平台运行(GUI程序 、非GUI程序均可)，但仅能跨PC操作系统平台，没法同时跨移动平台。虽然Java语言同时是Android系统上的原生开发语言，但Android和PC操作系统的Java代码也只能实现部分兼容（比如一些通用的无界面的代码）。

# 5.基于浏览器内核实现跨平台开发

## 5.1 web前端开发语言
使用HTML+CSS+JavaScript技术，设计开发的网页，可以在浏览器中运行；于是，
