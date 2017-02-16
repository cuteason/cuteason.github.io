---
layout: post
title: MySQL中汉字的存储长度的问题
date: 2017-2-16 20:59:48
location: 北京
category: tech
tags: [mysql, character]
---
之前一直以为在MySQL中，汉字能存储的个数是涉及的最大长度除以2或3（取决于所使用的字符集是utf8还是gbk）。刚好今天同事提到了这个问题，我本来是想确认一下在utf8下一个汉字占2个字节还是3个，结果发现之前的认知竟然是错误的。在网上查找了相关资料后，整个人感觉有点懵，因为有点绕，所以我决定自己验证一下。下面是我的验证过程。

表结构如下：

```sql
CREATE TABLE `t_user` (
  `id` int(11) NOT NULL,
  `username` varchar(10) DEFAULT NULL,
  `password` varchar(45) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `u_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='用户表';
```
这里设置的username长度是10，我们看看实际能存储多少中文：

![t_user表的数据](/images/mysql-data.png)

id=3的这条数据中文个数刚好也是10个，如果超过了10个汉字或者10个字母，则会报错：

![插入错误，数据过长](/images/mysql-insert-error-data-too-long-1.png)

![插入错误，数据过长](/images/mysql-insert-error-data-too-long-2.png)

所以，MySQL里varchar(10)能够存储10个汉字或者10个字符，而不是10/3或者10/2个汉字（分别对应utf8字符集和gbk字符集）

那么utf8字符集和gbk字符集有什么区别呢？这里的区别指的是汉字长度的区别。实际上，虽然都是10个汉字，utf8下的10个汉字其实是30的长度。

```sql
SELECT username, LENGTH(username), CHAR_LENGTH(username) from t_user order by id
```
![MySQL中汉字的实际长度](/images/chinese-character-length-in-mysql.png)

所以应该是MySQL中的varchar(10)的10个长度，是根据`CHAR_LENGTH`这个函数计算出来的，而不是字符实际的长度。

##结论

- varchar(n)能存储n个中文或者n个英文字符
- 实际的字符长度，utf8下一个汉字是3个字节，而gbk下是2个字节

完。
