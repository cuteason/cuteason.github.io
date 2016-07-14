---
layout: post
title: Hibernate处理一个实体类映射多张表的做法
location: 北京
category: tech
tags: [hibernate, java, web]
---
这次正在开发一个新的系统，考虑涉及到大量的数据存储和读写，因此准备采用数据库分表的方式，按照某个类型，把数据分别存储到不同的表中，但是字段完全相同。

## **1. 起因**
因为之前做的系统，都没有这个需求，因此都是一个数据库表对应一个Java类。这样的方式，有多少张表就会产生多少个Java类。考虑到目前的系统，每天都会从另外的数据库中导入大量的数据，因此数据量会不断增长。所以在系统开发之处，想到用数据库分表的方式进行。但是如果还是用前面的这种方式，将会要手动创建大量的Java类，同时还要写相应的Hibernate映射文件，而且这些类的属性都一样，只是类名的不同。（当然可以用自动化工具来实现映射文件的生成，但这里不作为我考虑的办法。）这样显然效率很低下，所以想研究看看有没有更好的办法。

## **2. 尝试union-subclass**
在网上搜寻一番之后，尝试用Hibernate的`union-subclass`来解决这个问题。一开始也不是很了解，所以就尝试了一下。`union-subclass`作用在有继承关系的类上面，父类的属性，子类继承之后，会自动映射到子类对应的表中去。比如，我们现在要建立需求表，由于要分表，因此数据库中存在如T_Demand_Type_A, T_Demand_Type_B 等等一系列的表。如果按照传统的方法，我们将要建立相应的Demand_Type_A.java, Demand_Type_B.java等等对应的java类。而这些数据表和类里面的字段和属性都完全一样。采用union-subclass之后，我新建了一个Demand类，像下面这样：

```java
public abstract class Demand {
    int id;
    int data;
    //这里省略了getter和setter方法
}
```

在这里，我把Demand类设置为abstract类型，这样就不用在数据库中建立与Demand对应的表。也就是，这里我只是把所有的属性都放到这个里面，其他子类只需要继承Demand类即可。但是Demand并不实际对应一张表。
Demand类对应的映射文件会像下面这样：

```xml
<hibernate-mapping>
    <class name="Demand" abstract="true">
        <id name="id" type="int">
            <generator class="assigned" />
        </id>
        <property name="data" type="int">
            <column name="DATA" />
    	</property>
        <!--  用union-subclass元素给每个具体子类映射到一张表  -->
        <union-subclass name="Demand_Type_A" table="T_DEMAND_TYPE_A">
            <!-- 映射本子类的属性 -->
        </union-subclass>
        <union-subclass name="Demand_Type_B" table="T_DEMAND_TYPE_B">
            <!-- 映射本子类的属性 -->
        </union-subclass>
        <!-- 可以继续添加 -->
    </class>
</hibernate-mapping>
```

这里的配置文件中，设置了abstract="true"，和上面的抽象类对应，这样Demand就不需要映射到某一张表上。在我们的目前的需求中，子类Demand_Type_A和Demand_Type_B等，都是空的，并没有实际的属性，因为子类都完全相同，没有自己的属性。这些类会像下面这样：

```java
public class Demand_Type_A extends Demand {
    // nothing here
}
```

用`union-subclass`的好处是，对于有继承关系的表和类来说，很方便。其最大的优势在于，如果表之间有一部分字段一样，但是还包含其它不相同的字段，即有自己的特有字段，比如在上面的例子中，T_Demand_Type_A中还有name字段，这样就比较适合采用`union-subclass`。但是在我们的这个项目需求下，这个还是不能解决我们的问题。因为还是需要一个表建立对应的Java类。虽然这个类里面没有任何东西，但是还是有这个工作量。所以最好还是放弃了`union-subclass`。

## **3. 尝试NamingStrategy**
继续在网上搜索结果，有一票结果说可以用Hibernate的`NamingStrategy`来实现，（诡异的是这些博客几乎写的东西都一样，给跪+_+）可以实现动态表名映射，即用一个类，一个配置文件，动态映射多个表。当然，这些表的结构都完全相同。然后我就看了一下这个怎么实现的。我找了一个发文时间最早的，应该是博主的原创，文章在[这里](http://jinguo.iteye.com/blog/209642)。这个需求跟我要解决的问题一样，都是想用一个映射文件，动态映射不同的表。但是这篇博客中提到的情况，是根据月份来生成表名的策略，而我的要求的是根据一个类型来生成表名，而这个类型参数应当是可以被作为参数生成相应的表名的。我查了一下，发现`NamingStrategy`好像不能接受参数，这篇博客写的情况是，根据系统时间来生成，刚好可以不通过函数传参，直接在函数内部就生成了动态的表名，所以也不能采用这个方法。`NamingStrategy`在那些生成Java类对应的数据库映射文件的时候，可以很方便的解决命名规范的问题，可以通过配置，写更少的配置文件即可。这里不符合我们的系统要求。

## **4. 最终解决方案**
尝试无果之后，几乎快要绝望了。这时候，又看到了一种新的解决办法，可以采用`SQLQuery`来实现动态表的映射。这个方法，可以获取数据库的数据，然后可以动态映射到自定义的类中，这样和我们的要求很符合。有一篇[博客](http://callan.iteye.com/blog/156123)，百度了一下，应该这个是原创的，其他的应该都是转的。这篇博客上写到了这种方法，已经很清楚了，这里就不再赘述，经验证确实可行。关键的一句代码就是`addEntity(ReadInfo.class)`，还有xml配置文件中的`mutable="false"`，这个把数据库取出来的数据动态映射到ReadInfo这个类上。这样，我们可以只用一个类，映射数据库中的多个表，只要这些表的表结构一样即可。完美解决我们的需求。解决一个问题太不容易了，好歹是找到了解决办法。

但是还有一个问题，上面的博客中还提到了数据保存的问题。使用这种方式，查询数据很便利；但在数据的存储和修改以及删除的时候，稍微麻烦一点。因为，不能直接采用Hibernate的映射机制，直接操作Java类来实现。因此，我们这里需要手动写SQL语句来实现。[博客](http://callan.iteye.com/blog/156123)已经有很好的描述，可以直接参考。

我们的系统，和这个也有一点小小的不同，因为我们的数据库表名也是需要从数据库中读取的。所以，写SQL语句插入的时候，需要首先获取表名，然后传参到SQL语句中，执行增删改查的功能。这个可以完美解决我们的问题，就是需要自己实现很多更新的功能。毕竟没有完美的解决办法，能较好的解决问题就可以了。

## **5. 后记**
写这篇文章的时候，又查询了一些资料，据说[通过hibernate拦截器实现自定义分表策略](http://www.iteye.com/topic/866142)，没有测试，改天可以验证一下。（估计改天就只是说说而已了。。。）

The end.
