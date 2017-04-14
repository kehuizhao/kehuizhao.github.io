---
layout:     post
title:      "一个小型翻译器"
subtitle:   "编译原理应用的小玩具"
date:       2017-3-30 10:30:00
author:     "ThdLee"
header-img: "img/THDTranslator/post-bg-THDTranslator.jpg"
tags:
    - Java
    - 编译原理
---

### 介绍
> 该程序代码改自[从零开始写个编译器吧系列](https://github.com/Moskize91/TaolanTutorial)。

> 源码可在我的[Github](https://github.com/ThdLee/THDTranslator)上查看。

> 由于改了删减了许多功能和语法，姑且将这个程序称为**翻译器**吧。 

该翻译器基于**LL(1)**语法，前端由词法分析器、语法分析器构成并将每个Token组织成语法树，然后将语法树转化成相应的中间代码，中间代码采用**三地址代码**，然后虚拟机部分将执行中间代码，显示结果。支持布尔值，浮点数，整数和字符串及其运算。由于去掉了定义函数部分，所以只能使用内建函数，每个函数都有返回值，如果函数不需要返回值（如`print()`），则函数返回`None`。

其实这是我的一个课程作业，由于作业不需要太多功能以及语法需要修改，我对源码做了部分修改，修改的部分有：

1. 删除了面向对象的所有相关代码和语法。
2. 删除了所有控制流语法和代码。
3. 删除了函数部分，不支持函数定义，但支持内部函数，将函数调用部分从编译时转移到运行时,并添加了部分内部函数。
4. 删除错误处理部分，对于语法错误，在分析完所有代码后，若有错误，则打印出全部错误，若运行时发生错误，直接停止运行并打印错误。
5. 删除和修改了中间代码。
6. 变量由关键字`var`定义改为直接定义，不需要修饰符。
7. 每行必须由`;`结尾。
8. 修改了不能进行较长运算式的bug。

### 内建函数
```
	print() 支持多个参数，打印参数并换行。
	max() 输入两个参数，输出最大值。
	min() 输入两个参数，输出最小值。
	sin() 输入一个数，输出对应的sin值。
	cos() 输入一个数，输出对应的cos值。
	random() 输出一个随机浮点数。
```	
### 基本文法  
```
	L0Expression -> L1Epression L0Expression'
	L0Expression'-> = L1Expression
	                | += L1Expression
	                | -= L1Expression
	                | *= L1Expression
	                | /= L1Expression
	                | %= L1Expression
	                | ε
	L1Expression -> L2Expression L1Expression'
	L1Expression'-> || L2Expression L1Expression'
	                | ε
	L2Expression -> L3Expression L2Expression'
	L2Expression'-> && L3Expression L2Expression'
	                | ε
	L3Expression -> L4Expression L3Expression'
	L3Expression'-> ^ L4Expression L3Expression'
	                | ε
	L4Expression -> L5Expression L4Expression'
	L4Expression'-> == L5Expression L4Expression'
	                | != L5Expression L4Expression'
	                | ε
	L5Expression -> L6Expression L5Expression'
	L5Expression'-> < L6Expression L5Expression'
	                | > L6Expression L5Expression'
	                | <= L6Expression L5Expression'
	                | >= L6Expression L5Expression'
	                | ε
	L6Expression -> L7Expression L6Expression'
	L6Expression'-> + L7Expression L6Expression'
	                | - L7Expression L6Expression'
	                | ε
	L7Expression -> L8Expression L7Expression'
	L7Expression'-> * L8Expression L7Expression'
	                | / L8Expression L7Expression'
	                | % L8Expression L7Expression'
	                | ε    
	L8Expression -> + L9Expression
	                | - L9Expression
	                | ! L9Expression
	                | ε
	L9Expression -> L10Expression
	                | L10Expression Invoker
	L10Expression -> ( L1Expression )
	                | Element
	Element -> Number
	          | Variable
	          | Boolean
	          | String          
                               
```
### 实例代码
```

num1 = 10 + 31 / (11 % 4) - 5.6;
num2 = 13.5;
num3 = random() * 20;
print(max(num1, num3), min(num2, num3));
num1 += 11.2;
print(num1);
#这是一条注释
print(sin(num1));
bool1 = True;
print(bool1 || False && True);
str = "string";
print(str);
	
```						 
