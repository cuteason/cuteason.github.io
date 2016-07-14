---
layout: post
title: Java教程：基本类——异常篇（Exceptions）
date: 2015-12-23 00:28
location: 北京
category: tech
tags: [java]
---
每次回来的时候，总是感觉无所事事。不想浪费这段时间，无意中打开了Java教程的英文文档，就想着把这个文档翻译一遍，顺便再熟悉一次，练练自己的英文能力也好。本教程包含入门、学习Java语言、基本类、集合、日期时间API、部署、准备Java程序员语言认证等几个部分。因为使用Java也有一段时间了，这里不打算从头看起，准备从第三部分——基本类——开始。本文就是介绍基本类的教程，主要内容包括：异常、基本输入/输出、并发、正则表达式和平台环境。

首先介绍异常（Exceptions），主要解释异常机制，以及如何利用异常机制来处理错误和其他异常情况。这节课描述什么是异常，如何抛出和捕获异常，一旦捕获异常之后如何处理以及如何运用异常类的层次结构。

## **什么是异常？**

术语异常（*exception*）是“异常事件”的简写。

>**定义：**一个异常是指一个发生在程序执行的时候，打乱了程序指令的正常流向的事件。

当一个方法中发生错误的时候，该方法就会产生一个对象，并把它传递给运行时系统（runtime system）。这个对象，被称为异常对象，包含了有关错误的信息，包括其类型和当错误发生时程序的状态。创建一个异常对象并把它传递给运行时系统被称为抛出一个异常（*throwing an exception*）。

一个方法抛出异常之后，运行时系统会尝试着去找一些东西来处理它。这一系列可能的处理异常的“东西”（somethings），是一些方法的有序列表，这些方法被调用直到到达发生错误的方法。这些方法的列表就是所谓的调用栈（*call stack*）（见下一张图）。

![调用栈][the-call-stack]
**调用栈**

运行时系统会搜索这个调用栈，目的是找到一个包含能处理异常的代码块的方法，这个代码块被称之为异常处理器（*exception handler*）。会从发生错误的方法开始找起，在调用栈中按照该方法被调用的顺序逆序查找。当一个合适的异常处理器被找到的时候，运行时系统就会把这个异常传递给这个处理器。如果被抛出的异常对象的类型和某个异常处理器能够处理的异常的类型相符，那么这个异常处理器就被认为是合适的。

被选中的异常处理器据说（is said to）会捕获这个异常（*catch the exception*）。如果运行时系统彻底找遍了调用栈中的所有方法，却还是没能找到一个合适的异常处理器，如下图所示，运行时系统（和程序）会终止运行。

![搜索调用栈以找到异常处理器][searching-the-call-stack-for-the-exception-handler]
**搜索调用栈以找到异常处理器**

使用异常来管理错误比传统的错误管理技巧有一些优势，详见异常的优点一节。（后面会加上这部分）

今天刚好是2015年12月31日，还有一刻钟就是2016年了，虽然感觉上没过春节这一年就还是没有完，也不妨做个展望：希望来年能把这个系列坚持写完。千里之行，始于足下。自勉，及与能看到这篇文章的诸位，共勉。

第一部分完。

------

## **捕获或指定要求**

有效的Java编程语言代码必须遵从捕获或指定要求（*Catch or Specify Requirement*）。这意味着可能会抛出某些异常的代码，必须被下面其一包围（enclosed）：

+ 一个能捕获这个异常的try语句。这个try必须提供一个异常处理器，如捕获和处理异常一节所描述的。（后面会加上这部分）
+ 一个指定会抛出异常的方法。这个方法必须提供一个列出了异常的throws子句，如指定方法抛出的异常一节所描述的。（后面会加上这部分）

没有遵从捕获或指定要求这个规则的代码不能编译。

不是所有的异常都必须遵从捕获或指定要求这个规则，想要明白为什么，我们需要了解一下异常的三种基本类型，只有其中的一个受制于这个要求。

### **三种类型的异常**

第一种类型的异常是已检测异常（*checked exception*），这些异常条件是一个优秀的（well-written）的应用程序应该预见到并且从中恢复的。例如，假定一个应用程序提示用户输入一个文件名，然后通过传递这个名称给
`java.io.FileReader`的构造器来打开文件。正常情况下，用户会提供一个存在的、可读的文件的名称，这样`FileReader`对象能够成功构造；并且应用程序的执行能够正常进行。但是，有时候用户提供了一个不存在的文件的名称，然后构造器抛出了`java.io.FileNotFoundException`异常。一个优秀的应用程序能够捕获这个异常，并且告知用户这个错误，或者有可能提示输入一个正确的文件名。

已检测异常**受制于**捕获或指定要求这个规则。所有的异常都是已检测异常，除了那些由`Error`, `RuntimeException`以及它们的子类表示的异常。

第二种异常类型是错误（*error*），这些异常条件是应用程序外部的，并且应用通常不能预见或者从中恢复的。比如，假设一个应用成功地根据输入打开了一个文件，但是却由于硬件或者系统故障而不能够读这个文件，这个未成功的读操作会抛出一个`java.io.IOError`异常。应用可能会选择捕获这个异常，来告知用户这个问题——但是，程序打印出堆栈跟踪（stack trace）然后退出也有可能是合理的。

错误**不受制于**捕获或指定要求这个规则，错误是那些由`Error`和它的子类来表示的异常。

第三种类型的异常是运行时异常（*runtime exception*），这些异常是应用程序内部的，并且应用程序通常不能预见或者从中恢复的。这些异常通常表示程序有错误（bugs），比如逻辑错误或者不恰当地使用API。比如，考虑一个之前已经描述过的传递一个文件名给`FileReader`的构造器的应用，如果一个逻辑错误导致空（null）被传递给构造器，构造器将会抛出`NullPointerException`异常。应用程序可以捕获这个异常，但是也有可能消除这个导致异常出现的bug是更有意义的。

运行时异常**不受制于**捕获或指定要求这个规则，运行时异常是那些由`RuntimeException`及它的子类表示的异常。

错误和运行时异常被统称为未检测异常（unchecked exceptions）。

### **绕过捕获或指定**

一些程序员认为捕获或指定要求这个规则是异常处理机制的一个严重缺陷，并且通过使用未检测异常而不是检测异常来绕过捕获。一般情况下，这种做法是不推荐的。未检测异常——争论一节（后面会加上）讨论了什么时候使用未检测异常比较合适。

第二部分完。

------

## **捕获和处理异常**

这一节将会介绍如何使用三个异常处理组件——try，catch和finally块——来写一个异常处理器。然后，介绍了Java SE 7中引入的try-with-resources声明。try-with-resources声明尤其适用于那些使用可关闭的（Closeable）资源，比如流（streams）的时候。

本节的最后一部分将会通过一个例子来分析在不同的场景下会发生什么。

接下来的例子定义并实现了一个名为ListOfNumbers的类，构造的时候，ListOfNumbers创建了一个包含连续的从0到9十个整数类型元素的数组。ListOfNumbers类同时还定义了一个writeList的方法，该方法会把这些数的列表写入一个名为OutFile.txt的文本文件中。这个例子使用了java.io包中定义的输出类，java.io包的内容在基本I/O一章里有介绍（后面会翻译这一章的内容）。

```java 
// Note: This class will not compile yet.
import java.io.*;
import java.util.List;
import java.util.ArrayList;

public class ListOfNumbers {

    private List<Integer> list;
    private static final int SIZE = 10;

    public ListOfNumbers () {
        list = new ArrayList<Integer>(SIZE);
        for (int i = 0; i < SIZE; i++) {
            list.add(new Integer(i));
        }
    }

    public void writeList() {
        // The FileWriter constructor throws IOException, which must be caught.
        PrintWriter out = new PrintWriter(new FileWriter("OutFile.txt"));

        for (int i = 0; i < SIZE; i++) {
            // The get(int) method throws IndexOutOfBoundsException, which must be caught.
            out.println("Value at: " + i + " = " + list.get(i));
        }
        out.close();
    }
}
```

黑体的第一行是调用构造函数。（由于Markdown中没有办法改变代码块中的样式，因此无法给某块代码加粗，这里用行号表示，指第20行的**new FileWriter("OutFile.txt")**）。这个构造函数在一个文件上初始化了一个输出流，如果这个文件不能被打开，构造函数会抛出IOException异常。黑体的第二行（第24行的**list.get(i)**）调用了ArrayList类的get方法，如果参数的值太小（小于0）或者太大（大于当前ArrayList所包含的元素数量），它将会抛出IndexOutOfBoundsException。

如果你尝试编译ListOfNumbers类，编译器会打印出一条关于FileWriter构造函数抛出的异常的错误信息。然而，它不会显示关于get方法抛出的异常的错误信息。原因是：由构造函数抛出的异常，IOException，是一个已检测异常；而另外一个由get方法抛出的异常，IndexOutOfBoundsException，是一个未检测异常。

现在你已经熟悉了ListOfNumbers这个类，以及其中会抛出的异常，准备好编写一个异常处理器来捕获并处理那些异常。

### **Try块**

构造一个异常处理器的第一步，就是用一个try块把可能会抛出异常的代码包围起来，一般来说，一个try块看起来像下面这样：

```java
try {
    code
}
catch and finally blocks...
```
示例中的标记的代码片段*code*包含了一条或多条合法的可能会抛出异常的代码行。（catch和finally块在接下来的两小节中介绍。）

为了给ListOfNumbers类中的writeList方法构建一个异常处理器，我们需要把writeList方法中抛出异常的语句用try块包围起来。有多个方法可以达到此目的。你可以把每一个可能会抛出异常的代码行都用try块包围起来并且给每一个try块都提供一个单独的异常处理器；或者，你可以把所有的writeList代码都放到一个单独的try块中并关联上多个异常处理器。下面的清单对整个方法使用了一个try块，因为有问题的代码很短。

```java
private List<Integer> list;
private static final int SIZE = 10;

public void writeList() {
    PrinterWriter out = null;
    try {
        System.out.println("Entered try statemen");
        out = new PrinterWriter(new FileWriter("OutFile.txt"));
        for (int i = 0; i < SIZE; i++) {
            out.println("Value at: " + i + " = " + list.get(i));
        }
    }
    catch and finally blocks ...
}
```
如果try块中发生了异常，那么这个异常会被关联的异常处理器处理。为了把一个异常处理器和一个try块关联起来，你必须把catch块放在它的后面；下一节，catch块，将会向你展示如何去做。

### **catch 块**

to be continued.

[the-call-stack]: /images/the-call-stack.png
[searching-the-call-stack-for-the-exception-handler]: /images/searching-the-call-stack-for-the-exception-handler.png
