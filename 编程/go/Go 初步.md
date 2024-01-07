# Go 初步



Go语言的演化主要分为三个分支

- 从C语言中，Go继承了表达式、控制流语句、基本数据类型、指针等等语法。而且它还继承了C语言所要强调的要点，即程序需要编译成高效的机器码，并且自然地与所处的操作系统提供的抽象机制相配合。
- 从Module-2、Oberon等语言借鉴了包、模块化地概念
- Go also is inspired by the concept of **communicating sequential processes (CSP)** from TonyHoare’s seminal 1978 paper on the found ations of concurrency. In CSP,a program is a parallel composition of processes that have no shared state; the processes communicate and synchronize using channels.

![image-20231224150022801](C:\Users\AtsukoRuo\AppData\Roaming\Typora\typora-user-images\image-20231224150022801.png)





Go是编译型的语言，它的工具链将程序的源文件转变成机器相关的原生二进制指令。

Go对于代码格式有着十分苛刻的要求，以下情况都会编译失败：

- 缺失导入或者存在不需要的包
- 声明但未使用的变量
- 左大括号与某些语句（for、func）不在同一行。