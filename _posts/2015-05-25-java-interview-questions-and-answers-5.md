---
layout: post
title: Java面试问题及回答（五）
location: 北京
date: 2015-05-25 13:20
category: tech
tags: [java, interview]
---
本文是上四篇博客[Java面试问题及回答（一）](http://sijh.github.io/2015/05/12/java-interview-questions-and-answers-1)、[Java面试问题及回答（二）](http://sijh.github.io/2015/05/18/java-interview-questions-and-answers-2)、[Java面试问题及回答（三）](http://sijh.github.io/2015/05/19/java-interview-questions-and-answers-3)、[Java面试问题及回答（四）](http://sijh.github.io/2015/05/19/java-interview-questions-and-answers-4)的后续，系列第五篇。本篇主要与图形用户界面编程相关。

**1. 什么容器使用边框布局（border layout）作为它的默认布局？**

答：Window、Frame和Dialog类采用边框布局作为默认布局。

**2. System类的作用是什么？**

答：System类的作用是提供对系统资源的访问。

**3. 什么事件是点击一个按钮的结果？**

答：ActionEvent的产生是点击一个按钮的结果。

**4. 滚动条（ScrollBar）和滚动窗格（ScrollPanel）的区别是什么？**

答：滚动条是一个组件，而不是一个容器；滚动窗格是一个容器，它处理自己的事件，并且执行自己的滚动。

**5. 什么是Swing？**

答：Swing是一个GUI控件库。Swing中的类不依赖于操作系统，它们不会创建对等组件（peer components），所以它们是轻量级的，不像AWT。

**6. 什么是AWT？**

答：抽象窗口工具提供了在Java中编写图形用户界面的应用程序接口。

**7. Swing和AWT之间的差异是什么？**

- AWT组件被认为是重量级的，而Swing则是轻量级的。
- Swing有可插入的外观和感觉。
- AWT是平台依赖的，同样的GUI在不同的平台上外观会不一样；而Swing是Java开发的，是平台无关的。（原文这句话没有标点，错误较多。）

**8. 什么是重量级的组件？**

答：对于每一个绘制调用，都会有一个本地调用来获得图形单元。比如，AWT。

-----------------------------------

下面都是图形化界面相关的内容，因此忽略。

The end.
