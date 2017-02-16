---
layout: post
title: Java传参是按值传递还是按引用传递？
location: 北京
date: 2015-04-29 13:30
category: tech
tags: [java]
---
这几天在看《JAVA核心技术(卷1):基础知识(原书第8版)》（[豆瓣链接](http://book.douban.com/subject/3146174/)），看到书中关于Java方法参数的讲解：

> Java程序设计语言总是采用值调用。也就是说，方法得到的是所有参数值的一个拷贝，特别是，方法不能修改传递给它的任何参数变量的内容。

这让我想起来之前看到的Stack Overflow上关于这个问题的一个讨论：[Is Java “pass-by-reference” or “pass-by-value”?](http://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value)，这个问题本身就有1985个赞，下面有53条回复，得票率最高的回答得到了1821个赞。这个问题讨论的激烈程度可见一斑。

最开始学习Java的时候，老师讲的也是Java方法参数是引用传递，所以一直以来的观点就是引用传递。但是随着学习的深入，诸如Stack Overflow上的讨论等让我对Java的方法参数有了更深的理解。这些天看《JAVA核心技术(卷1)》，再次看到这个问题，因此想进行一个总结，顺便也再加深理解。

首先，结论是：Java方法参数是值传递。这个毋庸置疑，关键看怎么理解。下面说说我的理解。

“方法不能修改传递给它的任何参数变量的内容”，这个是核心技术这本书中我们需要理解的一个重点。理解了这个这句话就能理解为什么Java中方法传递是按照引用传递。看个例子：

```java
public class TestPassByValueOrReference {
    public static void main(String[] args) {
        People cay = new People("Cay");
        System.out.println("Name is " + cay.getName());
	
        changeName2(cay); 
        System.out.println("After calling changeName2(), name now is " + cay.getName());
	
        changeName1(cay);
        System.out.println("After calling changeName1(), name now is " + cay.getName());
    }
    
    static void changeName1(People somebody) {
        somebody.setName("Gary");
    }	

    static void changeName2(People somebody) {
        somebody = new People("Gary");
    }
}

class People {
    private String name;
    public People() {
    }

    public People(String name) {
        this.name = name;
    }

    public void setName(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
}
```

我们来看这个代码跑出来的结果：

![java按值传递还是按引用传递的代码运行结果截图](/images/2015-04-29-is-java-pass-by-value-or-pass-by-reference-1.png)

如果是按照之前的理解，这里面应该两个地方都会改变name的值，但是结果却不是这样。“方法不能修改传递给它的任何参数变量的内容”，我们再来看，这句话对应的就是`changeName2()`这个方法，在这个方法里面想把形参somebody赋予一个新的People类的实例，结果表明这种方式在函数调用结束之后，实参cay没有发生改变。也就是书中说的“方法不能修改传递给它的任何参数变量的内容”。但是，为什么`changeName1()`就改变了cay的name了呢？我们可以看到，在这个`changeName1()`方法里面，使用的是People这个类的一个方法来改变的name属性的。

这两者有什么区别呢？我理解的区别在于：方法2中把形参的引用指向了一个新的对象，而方法1没有改变引用，只是改变了引用指向对象的内容。也就是说，方法不能改变对象参数的指向，只能改变对象参数的内容。我直白一点理解，那就是尝试在方法中使用赋值语句来改变对象参数（当然，原始类型参数更不用说）的值都不会成功。方法2中使用了赋值语句(=)来改变对象参数的引用，所以失败了。

其实，说到这，还是没有很好地说清楚为什么这就说明方法参数就是值传递而不是引用传递呢？我再说说我对这个值调用和引用调用的理解。在我看来，Java方法参数都是值传递。所谓的引用，也是一个值，它指向对象所在的地址（粗略这么理解）。比如cay所在地址是1021，那么调用`changeName2()`的时候，somebody的“值”也是1021（cay的地址值的一个拷贝），记住这里只是一个值，我把它等同于一个基本数据类型。那么在方法里面，如果你尝试把somebody重新指向一个比如1024的对象地址，相当于把一个基本数据类型的值由1021修改为1024，我们知道这样在方法调用结束，somebody的1024将不再被使用，cay的“值”还是1021，就像基本数据类型表现的那样。因此，方法2没有能修改cay的name。但是方法1不一样，方法1中的somebody也获得了一份cay所在地址的拷贝，“值”也是1021，和cay指向的是同一个对象；然后somebody调用了该对象的方法，修改了name的值，当方法结束之后，somebody也不再被使用，但是1021所指向的对象内容已经被修改，cay的值还是1021，还是指向原来的对象，但是对象的内容已经被方法1修改。因此，方法1成功修改了cay的name，但是同样它没有能修改cay的“值”——1021。

所以，Java中方法参数是值调用，而不是引用调用。不管参数是对象类型还是基本数据类型，**方法调用都不会改变参数的值**。

The end.
