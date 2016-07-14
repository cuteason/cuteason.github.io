---
layout: post
title: Hibernate中写SQL语句
location: 北京
category: tech
tags: [sql, hibernate]
---
由于业务需求，系统中采用了分表的设计，用一个类映射数据库多个表。这样的话，不能采用Hibernate中的update，save和delete等语句。所以，得自己写SQL语句来实现。

因为系统中设计的也都是一些基本的SQL语句，所以写起来应该也没有多大的难度。之前看过一些关于安全方面的资料和书，涉猎了一些SQL安全方面的东西。所以想在这里实现SQL的时候，把安全性考虑进去。也算是学以致用吧，就从手头的事开始。

其实也只是略懂一二，知道SQL注入这么一个东西。因此，这里面实现带参数的SQL语句的时候，没有直接用字符串拼接的方式，而是用setParameter()等方法来实现。坑爹就从这里开始了。

一开始写了自认为没有问题的SQL语句，跑起来，结果却报错了。

```java
Caused by: java.lang.IndexOutOfBoundsException: Remember that ordinal parameters are 1-based!
```

看这个错误字面上的意思，我想应该是说这个占位符的下标从1开始。于是我把读取参数的下标从1开始算，代码如下：

```java
public SQLQuery createSQLQuery(SQLQuery sqlQuery, final String sql, final Object[] params) {
    if(params != null && params.length > 0){
        for(int i = 0; i < params.length; i++){
            // 报错之前是i,现在修改为i+1
            sqlQuery.setParameter(i+1, params[i]);
        }
    }
    return sqlQuery;
}
```

修改之后，还是报错！查找了相关资料后发现，其实这个下标还是从0开始的。那我表示这个错误提示的让人莫名其妙，难道是我英语太差？查看了Hibernate的API之后（[Query setParameter()](http://docs.jboss.org/hibernate/core/3.6/javadocs/org/hibernate/Query.html#setParameter(java.lang.String, java.lang.Object))），这才知道，其实Hibernate的Query的这个方法，下标还是从0开始的。至于这个错误说是1-based，据说是JPA（Java Persistence API）中是从1开始，Stack Overflow上有关于这个问题的一些[讨论](http://stackoverflow.com/questions/3144235/jpa-hibernate-native-queries-do-not-recognize-parameters)，可以参考。所以，之所以出现这个错误，应该不是这个地方的问题。

最近实验室里的网络不给力，不能上Google了，就只好百度。查了一些资料，大都说是sql语句里不需要参数，但是却添加了一个参数。也就是参数的个数个参数值的个数没有对应。但是我的参数应该是没有问题的啊。那这个问题到底是怎么产生的，不能理解。

人生经常有三个错觉，这个应该是没有bug的；这个今晚应该可以写完；这个应该满足设计师的需求了。恩，我写的应该没有bug啊，怎么会出现问题呢？哭也没有用，还是得继续debuging...

来看看我写的所谓“没有bug”的代码是什么样的：

```java
public void updateMaterial(String table, Material material) {
    String sql = "update " + table + " set ";
    sql += "category =:?, ";
    sql += "cancelDate =:? ";
    sql += "where id =:?";
	
    dao.updateBySQL(sql, new Object[] {
        material.getCategory()),
        material.getDate(),
        material.getId()
    });
}
```

之前写的带参数查询，都是用命名参数来查的，像下面这样：

```java
String sqlStr = "select * from tb where id =: id";
SQLQuery sqlQuery = getSession().createSQLQuery(sqlStr);
sqlQuery.setInteger("id", 1);
```

所以这里面也按照上面的写法，没有意识到，这两种情况的SQL语句有不同的地方。查了半天，才发现原来是这里写错了。`category =:?`应该写成`category =?`，这个冒号太多余了。还是自己知识储备不够，以前都只顾写，而没有去好好学习。出来混，迟早是要还的。

现在应该没有问题了，恩，系统已经顺畅的跑起来了。打开页面，编辑，确定。

等等，怎么又报错了呢？

```java
java.lang.UnsupportedOperationException: Update queries only supported through HQ
```

又一堆红色的错误信息映入眼帘。。。

继续Debug，幸好这个问题解决方案很多，应该不是什么大问题，[这篇博客](http://blog.csdn.net/pengchang_1981/article/details/8245162)提出了一个解决方案，升级Hibernate就可以搞定了。

然后就开开心心地吃饭去了，吃饭回来，升级Hibernate。当前用的Hibernate版本是3.1.3，太低，去官网看了一下Hibernate的版本，果断升级到Hibernate4。这下总该没有问题了吧。结果更新之后，发现原来的一些方法在Hibernate4中已经没有了；而且，更新之后，连启动都启动不了。于是又是各种修改，搞了半天，还是不行。

我擦，说好的升级就可以解决的呢？再次看看上面提到的博客，升级到3.2.5可以解决，是不是我升级跨度太大，扯着蛋了？

那就下一个Hibernate3.6.10.Final版试试，结果启动还是报错。。。NND

```java
NoClassDefFoundError: javax/persistence/EntityListeners
```

无语了，已经折腾了一天，还是没有搞定。给跪了。。。

3.6的版本还是不行，那我再试试更低一点版本的吧，或许3.6版本的升级跨度还是大了一点，升级到3.2试试。3.2相对3.1升级没有那么明显，应该可以了吧。

我擦，启动成功了。泪奔！应该就是升级跨度太大，Hibernate很多方法已经不再使用，所以升级还是一步步地慢慢升级，不然一下从较低版本升级到高版本，需要更新的东西会很多。难怪写Java的都不喜欢更新jdk，虽然Java 8已经出来了，但是很多人应该还在用着6、7，更新的代价太大，渐渐地就跟不上时代的步伐了。扯远了，希望这次执行更新不要报错啊。编辑，更新。终于运行成功了,真特么不容易。

点着点着，过了一会发现有出错了。。。显示数据类型不一致，应该为NUMBER，但却获得BINARY。苍天啊，大地啊，这又是怎么回事？？

有数据的时候，更新没有问题；更新空值的时候，报错了。那应该就是设置字段为空的时候出错了。于是试着判断了一下，如果为空的话，直接赋值为""，即空字符串，DATE类型也可以直接用SQL语句赋值为空。

```java
public void updateMaterial(String table, Material material) {
    String sql = "update " + table + " set ";
    sql += "category =:?, ";
    sql += "cancelDate =:? ";
    sql += "where id =:?";
    
    dao.updateBySQL(sql, new Object[] {
        material.getCategory()),
        material.getDate()==null?"":material.getDate(),
        material.getId()
    });
}
```

至此，问题最终解决。

The end.
