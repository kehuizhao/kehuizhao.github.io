---
layout:     post
title:      "浅谈程序的内存分配"
subtitle:   "假装我有一套房..."
date:       2017-3-31 00:05:00
author:     "ThdLee"
header-img: "img/MemoryAlloc/post-bg-memoryAlloc.jpg"
tags:
    - 操作系统
    - 编译原理
---

## 内存分配

尽管现在的许多高级语言已经不需要程序员去直接处理内存分配和垃圾回收，但是内存的管理是学习编程过程中的一个很重要的概念，理解相关概念和应用能够让我们对编程和计算机有更深的理解。

一般来讲，程序分为栈区、堆区、文字常量区、静态/全局区和代码区。

**栈区**：一般存储局部变量和常量，包括函数的参数，由编译器分配和释放。存取速度较快，但存储的数据一般生命周期较短。

**堆区**：一般存储对象，由程序员分配和释放，如果程序员不释放，则由操作系统释放。当然，如果使用的高级语言有垃圾回收机制，则大多数情况下程序员也不用担心内存空间的释放问题了。

**文字常量区**：存放字符串。为了节省内存空间，编译器一般会将字符串常量存储在文字常量区，在给字符串变量赋值时先在文字常量区寻找，如果有相同字符串，则共享该字符串地址，如果没有，则在常量区中添加该字符串，并将地址传给变量。

**静态/全局区**：存放静态变量和全局变量，程序结束后由系统释放。

**代码区**：存放函数体的二进制代码。

## OC中的内存地址

我们以Objective-C语言为例，定义一些常量、变量和对象来简单地观察一下内存的分配（Objective-C是扩充C的面向对象的编程语言，NSArray、NSString都是类）。

```
const int constA = 100;
int a = 10;
int b = 11;
int c;
static int staA = 0.5;
static int staB = 0.6;
static int staC;
NSArray *array1 = @[@1, @2, @3];
NSArray *array2 = @[@"2333"];
NSArray *array3 = [[NSArray alloc] initWithArray:array1];
NSString *str = @"Hello world";
NSString *str1 = @"Hello world";
NSString *str2 = @"Hello";
static NSArray *stArrayA;
static NSArray *stArrayB;
stArrayA = @[@1];
stArrayB = @[@21,@3];
```

将相应的值的地址打印出来：

```
NSLog(@"const int %lx", &constA);         // const int 7fff5fbff6ec
NSLog(@"int %lx", &a);                    // int 7fff5fbff6e8 
NSLog(@"int %lx", &b);                    // int 7fff5fbff6e4
NSLog(@"int %lx", &c);                    // int 7fff5fbff6e0
NSLog(@"static int %lx", &staA);          // static int 100002330
NSLog(@"static int %lx", &staB);          // static int 100002334
NSLog(@"static int %lx", &staC);          // static int 100002338
NSLog(@"NSArray* %lx", &array1);          // NSArray* 7fff5fbff6d8
NSLog(@"NSArray* %lx", &array2);          // NSArray* 7fff5fbff6d0
NSLog(@"NSArray* %lx", &array3);          // NSArray* 7fff5fbff6c8
NSLog(@"NSString* %lx", &str);            // NSString* 7fff5fbff6b8
NSLog(@"NSString* %lx", &str1);           // NSString* 7fff5fbff6b0
NSLog(@"NSString* %lx", &str2);           // NSString* 7fff5fbff6a8
NSLog(@"static NSArray* %lx", &stArrayA); // static NSArray* 100002340
NSLog(@"static NSArray* %lx", &stArrayB); // static NSArray* 100002348
NSLog(@"NSArray %lx", array1);            // NSArray 1002032a0
NSLog(@"NSArray %lx", array2);            // NSArray 1002013f0
NSLog(@"NSArray %lx", array3);            // NSArray 1002035c0
NSLog(@"NSString %lx", str);              // NSString 100002080
NSLog(@"NSString %lx", str1);             // NSString 100002080
NSLog(@"NSString %lx", str2);             // NSString 1000020a0
NSLog(@"static NSArray %lx", stArrayA);   // static NSArray 1002035f0
NSLog(@"static NSArray %lx", stArrayB);   // static NSArray 1002013d0
```
分析一下打印结果，可以得出：

1. 局部的常量和变量（包括指针）存于高地址，连续声明的变量地址会紧挨在一起，地址从高地址向低地址扩展。这一部分就是**栈**。
2. 对象存储于低地址，而且连续声明的对象地址不连续。这一部分就是**堆**，其实堆内存是由类似链表的结构串起来的，每一个空闲内存块地址并不一定连续，所以会出现这样的现象。
3. 静态的变量会存储于低地址，比堆空间地址还低，静态的对象只有指针存储在静态区，分配给对象的空间在堆中。
4. 表示字符串的类`NSString`的对象存储在比静态区地址还要低一些的地址中，变量`str`与`str1`的值相同，所以共享字符串地址。

## 更进一步地探讨

至此，我们就会对程序中内存分配和管理有了一定的了解，但是程序中的内存地址是怎么来的呢？它们是计算机中的物理地址吗？如果不是，那又和物理地址有什么关系呢？

这几个问题牵扯到了**编译器**的和**操作系统**的一些相关知识，但对这些内容深入地探讨超出了本文的范围，因此本文只能尽量描述清楚其中的关系。

进入正题，编译器将代码转换为可执行程序时，必须为代码产生的各个值分别分配一个存储位置。编译器必须理解值的类型、长度、可见性和生命周期。编译器必须考虑一系列对代码的内存处理问题来定义一组约定来解决这些问题。

为分配存储，编译器必须理解全系统范围内对内存分配和使用的约定。编译器、操作系统和处理器协助，以确保多个程序能够以交错的方式（时间片）安全地执行。

除了创建栈，对于大多数语言，编译器都需要创建堆，以便为动态分配的数据结构提供内存。为保证高效地利用内存空间，堆和栈被置于开放空间的两端，彼此相向增长，所以就出现了我们所观察到的现象。当然，将堆和栈互换位置效果也是一样的。下图是单个程序编译后所用地址空间的典型布局，具体的实现和细节可能会因编译器和语言的不同而不同：

![逻辑地址的空间布局](http://thdlee.com/img/MemoryAlloc/logicalAddress.jpg)

这只是编译器的视角所看到的地址空间，编译器会为编译的每个程序分配一个独立的地址空间，让程序以为自己拥有独立的内存（其实也让程序员以为程序有独立的内存），其实这只是假象。在执行程序时，操作系统会将这些逻辑地址空间映射到处理器支持的物理地址空间中。

![地址空间的不同视图](http://thdlee.com/img/MemoryAlloc/addressSpace.jpg)

所以说，我们所看到的只是逻辑地址，我们在代码中对所谓内存的操作，也只是对逻辑地址的操作，这些操作的具体过程是由编译器和操作系统以及硬件为我们完成的。				 
