+++
title = "初识Golang—defer"
date = 2017-09-23T16:25:10+08:00
categories = ["Development","Golang"]
trags = ["Development","Golang"]
type = "post"
+++

本文参考自[Andy's Blog](https://chinazt.cc/2017/06/30/deferde-shi-yong-gui-ze/)、[golang的defer精析](https://studygolang.com/articles/742)

关键字defer是golang中一个实用且常见的特性，defer代码块会在函数调用链表中增加一个函数调用，**<font color="red">它会在函数正常返回之前执行</font>**。因此，defer通常用来释放函数内部的对象资源。

    func CopyFile(dst, src string) (w int64, err error) {
        srcFile, err := os.Open(src)
        if err != nil {
            return
        }
        defer srcFile.Close()

        dstFile, err := os.Create(dstName)
        if err != nil {
            return
        }
        defer dstFile.Close()

        return io.Copy(dstFile, srcFile)
    }

通过defer,即使其中的Copy()函数抛出异常，Go仍然会保证dstFile和srcFile会被正常关闭。

## defer的使用规则

### 规则一 当defer被声明时，其参数会被实时解析

    func example1() {
        i := 1
        defer fmt.Println("v1", i)
        i = 2
        defer fmt.Println("v2", i)
        i = 8
        defer fmt.Println("v3", i)
        fmt.Println("v4", i)
    }

    // v4 8
    // v3 8
    // v2 2
    // v1 1

在上面的例子中我们可以看出，虽然我们在defer后调用的是带变量的输出函数``fmt.Println(i)``，但这个形参的值在defer声明的时候就已经确定了，而不是defer真正执行时的变量值。

### 规则二 defer执行顺序为先进后出

当同时定义了多个defer代码块时，golang按照先定义后执行的顺序依次调用defer。在例子``func example1()``中，我们可以看到输出依次为v4->v1，结果为8821。

### 规则三 defer是在return之前执行的

    func example2() (result int) {
        defer func() {
            result++
        }()
        return 0
    }

    //return 1

在[官方文档](https://golang.org/ref/spec#Defer_statements)中提到defer是在return之前执行的，需要明确的是golang中返回值的方式与C不同，为了支持多值返回，golang使用的是栈返回值，而C则为寄存器。此外，语句``return xx``并非一条原子指令，而是先给返回值赋值，再做一次空的return，即**赋值指令 + RET指令!**而defer的执行是在return之前，即**赋值指令 + defer指令 + RET指令。**根据这一规则，``func example2()``可以改写为

    func example2() (result int) {

        result = 0  //给返回值赋值

        func() {  //调用defer函数
            result++
        }()

        return  //执行RET
    }

根据这个规则进行转换后，对golang中defer的执行顺序与函数返回值是否有了更多的认识呢，在定义defer时遇到涉及修改返回值的时候不妨根据规则三转换一下。
