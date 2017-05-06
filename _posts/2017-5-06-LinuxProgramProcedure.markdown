---
layout:     post
title:      "Linux x86-64 静态链接程序的运行过程"
date:       2017-5-06 15:00:00
author:     "ThdLee"
tags:
    - Linux
    - C/C++
---

> 本文主要讲述在Linux x86-64中，静态链接程序（即不使用任何共享库（shared libaray）的程序）的运行过程。

## 序章

C/C++程序员在写程序时，总是默认程序是从`main`函数开始的，我们会认为这理所当然，但事实上，当程序在执行到`main`函数时，很多事情已经完成了。我们可以看看一下几个例子：

```c
#include <stdio.h>
int a = 10;
int main(int argc, const char * argv[]) {
    printf("%d\n", argc);
    printf("%d\n", a);
    return 0;
}
```
在运行main函数的时候，全局变量`a`已经初始化完成，并且`main`的两个参数`argc`与`argv`已经被传了进来。


```c
#include <stdio.h>
__attribute((constructor)) void before_main()
{
    printf("%s\n",__FUNCTION__);
}

__attribute((destructor)) void after_main()
{
    printf("%s\n",__FUNCTION__);
}
int main(int argc, const char * argv[]) {
    return 0;
}

```
执行结果：

```
before_main
main
after_main
```

构造函数`before_main`会在`main`函数开始之前被调用，析构函数`after_main`会在`main`函数结束之后被调用。 

而C++中，`main`函数之前所能执行的代码还会更多。所以，`main`函数既不是一个程序的开始，也不是一个程序的结束。那么，一个程序到底是怎样开始和结束的呢，`main`函数前后到底发生了那些事呢？这就是我们要讨论的话题。

## Linux内核装载ELF过程

当我们在Linux系统的bash下输入一个命令执行某个ELF程序时，bash进程会调用`fork`系统调用创建一个新的进程，然后新的进程会调用`exec`系统调用执行指定的ELF文件。

![ELF结构](http://thdlee.com/img/LinuxProgramProcedure/ELF.png "ELF结构")

在进入exec系统调用之后会触发Linux中的`sys_execve`，它会进入内核为新进程的运行做一些准备工作，然后调用`do_execve`。`do_execve`会首先查找被执行的文件，然后将文件的前128个字符读入内存中，判断文件的格式。之后会调用`search_binary_handler`去搜索和匹配合适的可执行文件装载处理过程。所以ELF可执行文件的装载处理过程`load_elf_binary`被调用。

内核读取ELF的头部检查文件格式的有效性，之后寻找动态链接的`.interp`段，由于我们只讨论静态链接，所以没有这个段。然后内核根据ELF可执行文件的程序头表的描述，将程序的段映射到内存中。调用`create_elf_tables`初始化环境变量，并把参数放入栈中。最后系统调用返回的地址被修改成ELF可执行文件的入口点，对于静态链接的ELF文件，这个程序入口就是ELF文件的文件头中`e_entry`所知的地址。

当`load_elf_binary`执行完毕，返回至`do_execve`再返回至`sys_execve`时，系统调用返回的地址已经被改成了被装载的ELF程序的入口地址了。所以当`sys_execve`从内核返回到用户态是，`rsp`寄存器直接跳转到了ELF程序的入口地址，于是新的程序喀什执行，ELF可执行文件装载完成。

## 入口函数

> Linux中的[glibc](http://ftp.gnu.org/gnu/glibc/)的子目录`csu/`中有关于程序启动的代码。

一般情况下，系统链接器程序`ld`会将编译器产生的**可重定位目标文件**构建成**完全链接的可执行目标文件**。而`ld`链接器的链接脚本默认地指定`_start`为程序的入口。

以下面这个简单的程序为例，我们来探查一下该程序编译后的入口函数：

```c
int main() {
    return 0;
}
```
编译:

```
$ gcc main.c
```
查看产生的可执行文件的ELF头部信息：

```
$ readelf -h a.out

ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  ...
  Entry point address:               0x4004d0
  ...
```
我们只关心它的入口地址，但是这个入口地址指的是哪里呢？查看`a.out`的反汇编代码：

```
$ objdump -d a.out

  ...
  
00000000004004d0 <_start>:
  4004d0:	31 ed                	xor    %ebp,%ebp
  4004d2:	49 89 d1             	mov    %rdx,%r9
  4004d5:	5e                   	pop    %rsi
  4004d6:	48 89 e2             	mov    %rsp,%rdx
  4004d9:	48 83 e4 f0          	and    $0xfffffffffffffff0,%rsp
  4004dd:	50                   	push   %rax
  4004de:	54                   	push   %rsp
  4004df:	49 c7 c0 90 06 40 00 	mov    $0x400690,%r8
  4004e6:	48 c7 c1 20 06 40 00 	mov    $0x400620,%rcx
  4004ed:	48 c7 c7 04 06 40 00 	mov    $0x400604,%rdi
  4004f4:	e8 a7 ff ff ff       	callq  4004a0 <__libc_start_main@plt>
  4004f9:	f4                   	hlt    
  4004fa:	66 0f 1f 44 00 00    	nopw   0x0(%rax,%rax,1)
  
  ...
```
可以看到`0x4004d0`就是这个程序中`_start`的地址。`xor`是异或运算，`ebp`指向栈底，所以与自身做异或运算会把`ebp`设置为零，表面当前是程序的最外层函数。`rdx`中其实存放的是`rtld_fini`的函数指针，并将其存入`r9`。`pop`将`argc`存入`rsi`中，然后将站指针指向`argv`，再通过`mov`将该地址存入`rdx`中。栈指针寄存器与掩码进行`and`运算来重置自己。接下来，我们通过查看反编译的代码，会发现`0x400690`是`__libc_csu_fini`的地址，被存入`r8`中，`0x400620`是`__libc_csu_init`的地址，被存入`rcx`中，`0x400604`是`main`的地址，被存入`rdi`中。最后代码调用`__libc_start_main`，这个函数才是实际执行代码的函数，而之前的那些对寄存器的设置其实就是对`__libc_start_main`的函数参数的设置。

x86-64的`_start`的实现在`sysdeps/x86_86/Start.S`中，我们其中的参数与相应地址和寄存器匹配：

```
	main:		%rdi <-- 0x400604
	argc:		%rsi <-- [RSP]
	argv:		%rdx <-- [RSP + 0x8]
	init:		%rcx <-- 0x400620
	fini:		%r8  <-- 0x400690
	rtld_fini:	%r9 <-- rdx on entry
	stack_end:	stack <-- rsp
```

## \_\_libc\_start\_main

`csu/libc-start.c`中定义了`__libc_start_main`函数：

```c
int LIBC_START_MAIN
	(int (*main) (int, char **, char ** MAIN_AUXVEC_DECL),
	int argc,
	char **argv,
	__typeof (main) init,
	void (*fini) (void),
	void (*rtld_fini) (void),
	void *stack_end)
```
这里可以看到其实`main`函数有三个参数，除了`argc`与`argv`外还有一个环境变量表，这个环境表参数是由`exec`传入的。事实上，三个参数的`main`函数的出现只是历史遗留问题，我们可以直接使用全局变量`environ`来获取这个环境表的数组指针。

`__libc_start_main`函数的具体实现中关于函数指针参数的使用顺序是：

```c
1. __cxa_atexit ((void (*) (void *)) rtld_fini, NULL, NULL);
2. __cxa_atexit ((void (*) (void *)) fini, NULL, NULL);
3. (*init) (argc, argv, __environ MAIN_AUXVEC_PARAM);
4. result = main (argc, argv, __environ MAIN_AUXVEC_PARAM);
5. exit (result);
```

`__cxa_atexit`函数是`glibc`的内部函数，用于将注册的函数在`main`结束之后，即被`exit`函数调用，注册顺序与调用顺序相反。在静态链接程序中`__cxa_atexit(func, NULL, NULL)`与`atexit(func)`作用相同。所以，函数的执行顺序为`init -> main -> fini -> rtld_fini`。我们甚至可以猜出这些函数的作用：

* `init`：`main`调用前的初始化工作。
* `fini`：`main`结束后的收尾工作。
* `rtld_fini`：和动态加载有关的收尾工作，`rtld`是`runtime loader`的缩写。

我们再回过头看看之前的反汇编代码，就会发现`init`就是代码中的`__libc_csu_init`，而`fini`是`__libc_csu_fini`。至于`rtld_fini`的函数指针，是通过`rdx`寄存器传入`_start`中的。

所以`__libc_start_main`函数进行了如下的操作：

1. 确定环境变量在栈的何处。
2. 如果需要，准备[辅助向量](https://www.gnu.org/software/libc/manual/html_node/Auxiliary-Vector.html)（auxiliary vector）。
3. 初始化线程相关的功能（比如pthread，TLS等）。
4. 执行某些安全相关的`bookkeeping`（这不是一个独立的步骤，而是遍及整个函数）。
5. 初始化`libc`。
6. 为执行的退出函数注册`fini`，`rtld_fini`。
7. 通过传入的指针（`init`）调用程序初始化函数。
8. 调用`main (argc, argv, envp)`。
9. 以`main`的结果作为退出码调用`exit`。

## exit相关

由`__libc_start_main`函数的实现可以看出，程序在执行完`main`函数后都会执行`exit`函数（具体实现在`stdlib/exit.c`中）。所以，在`main`函数中返回一个整型数值与在`main`末尾用该值调用`exit`函数是等价的。`exit`会执行通过`atexit`注册过的函数，然后调用`_exit`来直接结束进程。进程正常结束有两种情况：

1. `main`正常返回，由`__libc_start_main`来调用`exit`函数。
2. 程序中直接使用`exit`退出。

这样一来，进程在`__libc_start_main`函数末尾直接通过`exit`结束，所以在`main`函数中，我们可以不用`return`。

```c
\\ main1.c
int main() {

}
\\ main2.c
int main() {
	return 0;
}
```

我们先编译这两个文件并运行，然后通过`echo $?`来打印终止状态。查看结果会得到`main1.c`的终止状态是一个随机值，而`main2.c`的终止状态是0。为了进一步了解其中的差异，我们再使用`objdump`分别进行反汇编，对比汇编代码，我们会发现在`main`函数的汇编代码里，`main2.c`多了一个操作：

```
	mov    $0x0, %eax
```

所以，`return`语句只是为`eax`寄存器赋值，而终止状态就存在这个寄存器中，在`main1.c`中，由于没有`return`语句，所以最终由`eax`寄存器得到的值是一个默认值。

> *c99*规定，如果`main`函数最后没有使用`return`语句，那么编译器要在生成的目标文件中自动加入`return 0`。*C++*编译器也同样如此。

还有一个问题，之前提到的反汇编代码中为什么在调用`__libc_start_main`后还会有一个`hlt`操作呢？

事实上，在`Linux`里，进程必须使用`exit`系统调用结束。一旦`exit`被调用，程序的运行就会终止，因此`__libc_start_main`永远不会返回，所以`_start`末尾的`hlt`不会执行。但是如果`__libc_start_main`没有调用`exit`，即函数返回了，那么`hlt`指令就可以发挥作用强行把程序给停下来。

## 补充

在本文开头提到的在`main`函数前执行的那些特殊代码，会由链接器负责插入到`__libc_csu_init`函数中，然后由它调用。`__libc_csu_fini`也是如此。然而这两个函数在程序以`_start`为入口时会被调用，但是我们将程序入口换成其他函数，会发生什么呢？

我们先看个例子：

```c
#include <stdlib.h>
#include <stdio.h>
int foo() {
    printf("%s\n",__FUNCTION__);
    return 0;
}

void bar() {
    printf("%s\n",__FUNCTION__);
    exit(0);
}

__attribute((constructor)) void before_main()
{
    printf("%s\n",__FUNCTION__);
}

__attribute((destructor)) void after_main()
{
    printf("%s\n",__FUNCTION__);
}

int main() {
    printf("%s\n",__FUNCTION__);
}
```

强行把`foo`函数换成程序的入口，并读取`a.out`的ELF文件头部，然后进行反编译：

```
$  gcc -e foo main.c
$  readelf -h a.out

ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
...
  Entry point address:               0x4005d7
...

$  objdump -d a.out

...
00000000004005d7 <foo>:
...
```

此时`foo`函数已经代替`_start`成为程序入口，执行程序：

```
$  ./a.out

foo
Segmentation fault (core dumped)
```

可以看到，没有了`_start`函数，也就不会执行`__libc_start_main`，所以构造函数和析构函数并没有被执行，而且由于程序没有用`exit`来结束，所以产生了一个段错误。

再将程序入口换为`bar`：

```
$  gcc -e bar main.c
$  ./a.out

bar
```

程序成功运行，而且成功退出，但是同样没有调用构造函数和析构函数。至此，我们就明白了`main`函数执行前的初始化工作和结束后的收尾工作到底是谁在执行了。

## 结尾

最后，我们通过一张图来总结一下Linux中一个C/C++程序的运行过程：

![C程序的开始和结束](http://thdlee.com/img/LinuxProgramProcedure/CStartAndTerminate.png)

内核通过`exec`运行一个进程，在`C start-up routine`中，系统会自动调用一些初始化函数，再执行`main`函数，然后通过调用`exit`，先执行那些通过`atexit`注册的函数，再进行一些收尾工作，最后使用`_exit`结束进程。

---
要想进一步了解，可以阅读以下资料：

1. [How statically linked programs run on Linux](http://eli.thegreenplace.net/2012/08/13/how-statically-linked-programs-run-on-linux#id8)
2. [Linux x86 Program Start Up](http://dbp-consulting.com/tutorials/debugging/linuxProgramStartup.html)
3. [《深入理解计算机系统》](https://book.douban.com/subject/5333562/)
4. [《程序员的自我修养》](https://book.douban.com/subject/3652388/)
5. [《UNIX环境高级编程》](https://book.douban.com/subject/1788421/)