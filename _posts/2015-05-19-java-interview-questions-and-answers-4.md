---
layout: post
title: Java面试问题及回答（四）
location: 北京
date: 2015-05-19 21:29
category: tech
tags: [java, interview]
---
本文是上三篇博客[Java面试问题及回答（一）](http://sijh.github.io/2015/05/12/java-interview-questions-and-answers-1)、[Java面试问题及回答（二）](http://sijh.github.io/2015/05/18/java-interview-questions-and-answers-2)、[Java面试问题及回答（三）](http://sijh.github.io/2015/05/19/java-interview-questions-and-answers-3)的后续，系列第四篇。

**1. 什么时候可以使用parseInt()方法？**

答：该方法用于获取某个字符串的原始数据类型。

**2. StringBuffer类和StringBuilder类的差别？**

答：只要有可能就使用StringBuilder类，因为它比StringBuffer类快。但是如果线程安全是必须的，那么就要使用StringBuffer对象。

**3. 字符串中不可变的意思是什么？**

答：不可变的简单意思就是不可修改或者不可改变，字符串对象一旦创建，它的值就不能被改变。

**4. 正则表达式中用于进行模式匹配的包是什么？**

答：java.util.regex包用于这个目的。

**5. Exception类下面的两个子类是什么？**

答：Exception类有两个子类：IOException类和RuntimException类。

**6. 为什么Java使用字符串字面量的概念？**

答：为了让Java内存更加高效（因为如果字符串常量池中存在的话，新对象就不会被创建）。

**7. Java中toString()方法的目的？**

答：toString()方法返回的是任何对象的字符串表示。如果你打印任何对象，那么Java编译器会自动在内部调用对象的toString()方法。所以你可以覆盖toString()方法，返回需要的结果。

**8. 类会继承它父类的构造器吗？**

答：类不会继承来自它任何父类的构造器。

**9. 为什么String类是不可变的？**

答：String类不可变，这样字符串对象一旦创建，它就不能被更改。因为String类不可变，所以可以在多个线程中共享。

**10. 为什么要使用Vector类？**

答：Vector类提供了实现可扩充的对象数组的能力。如果你之前不知道数组的大小，那么Vector类非常有用。

**11. 什么是I/O过滤器？**

答：一个I/O过滤器（I/O filter）是可以从一个流读取然后写入另外一个流的对象，通常是在数据在一个流传递到另一个的时候以某种方式修改数据。

**12. 什么是序列化？**

答：序列化是把一个对象的状态写进一个字节流的过程，主要用于在网络中传输对象的状态。

**13. 什么是反序列化？**

答：反序列化是从序列化状态中重建对象的过程，是序列化的逆过程。

**14. 什么是外部化？**

答：外部化接口用来把对象的状态以压缩格式写入字节流，它不是一个标记接口。

**15. System类的作用？**

答：System类的作用是提供对系统资源的访问。

**16. 静态和非静态变量的差别？**

答：静态变量与类相关，和类作为一个整体，而不是类的某一个实例。非静态变量在每个对象实例上都有不同的值。

**17. 解释Java程序中子类的使用？**

答：子类继承了所有的共有的和受保护的方法及其实现，同时也继承了所有的默认的修饰方法及其实现。

**18. 什么是瞬态变量（transient variable）？**

答：瞬态变量是在序列化过程中不能被序列化的变量，而且在反序列化过程中被初始化为它的默认值。

**19. break语句和continue语句的差别？**

答：break语句会导致应用它的语句（switch，for，do，while）的终止。一个continue语句用于结束当前循环迭代然后返回循环语句的控制。

**20. 什么异常类是由Java运行时系统产生的？**

答：Java运行时系统产生RuntimeException和Error异常。

The end.
