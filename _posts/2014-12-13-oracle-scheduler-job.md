---
layout: post
title: Oracle中的定时任务
location: 北京
category: tech
tags: [oracle, database]
---
最近在做的项目，需要每天定时从一个远程服务器上的数据库导数据到我们的数据库。因此，研究了一下Oracle中的定时任务。

## **1. DBMS_JOB**
一开始尝试的是DBMS_JOB，因为一开始网上搜索Oracle定时执行的结果靠前的都是这个，这也是导致被坑很久的原因之一。网上关于DBMS_JOB如何实现定时任务的文章很多，因此写出来一个并没有很困难，命令大致像下面这样：

```sql
variable jobN number;
begin
    dbms_job.submit(
        job => :jobN,
        what => 'procdure_name;',       --需要执行的存储过程，注意引号里面的分号
        next_date => sysdate,           --立即执行
        interval => 'trunc(sysdate)+1'  --每天凌晨00:00执行一次
    );
end;
```

job可以调用PL/SQL代码块或者存储过程来实现定时任务，因为这里只关注使用存储过程来实现，所以what参数指定的是存储过程的名称。

存储过程这里我们实现的是导入数据，从远程服务器上的数据库（A）导入到我们的数据库（B），采用的是建立Database Link的方式。然后因为A服务器上的数据库的表中有一些字段是clob类型的，直接导入B数据库会出问题，这是个典型的问题，解决办法网上都有，这里就不阐述。因为Oracle数据库的限制，这里需要建立一个临时表，然后先把数据导入临时表，最后再把临时表的数据导入到B在的数据库表中。具体为什么要这样，没有过多研究，着急要实现这么一个东西，所以也管不了那么多，先用起来再说。（拿来主义， :)）所以存储过程中要建立临时表，这时候问题就来了。

什么问题呢，问题在于我对存储过程不熟，刚认识，她不认识我，我也不熟悉她，你看看我我看看你干瞪眼，还经常给我出个难题让我十分狼狈。我存储过程的SQL文件是这么写的：

```sql
create or replace procedure proc_import_data is
begin
    --创建临时表
    create global temporary table temp_1 as select * from table_from_b;
    --把A数据库中表的数据导入到临时表
    insert into temp_1 select * from table_from_a@database_link_name;
    --把临时表的数据导入到B数据库的表中
    insert into table_from_b select * from temp_1;
    commit;
end proc_import_data
```

按理说这么做应该没有问题啊，应该是按我的理来说没有问题，但是偏偏就出问题了。不熟悉存储过程的弊端就体现出来了，执行这个SQL文件的时候，会报错，显示表temp_1不存在，也就是创建临时表失败了。但是创建临时表的命令确实是没有问题的，这个可以确保。但是怎么写到这个SQL文件里面就不能创建成功呢？十万个为什么伴随着十万匹草泥马一齐涌上心头，五味杂陈。肿么回事？

既然命令没有问题，那是不是不能写在存储过程里面？或者是在存储过程里面不能这么写？

盯着错误她也不会告诉你是哪儿有问题的，还得自己动手去找解决办法，去网上找吧。搜寻了半天，发现似乎确实不能这么写。于是改成了下面这样：

```sql
create or replace procedure proc_import_data is
begin
    --创建临时表
    execute immediate 'create global temporary table temp_1 as select * from table_from_b';
    --把A数据库中表的数据导入到临时表
    insert into temp_1 select * from table_from_a@database_link_name;
    --把临时表的数据导入到B数据库的表中
    insert into table_from_b select * from temp_1;
    commit;
end proc_import_data
```

其实命令还是跟上面的一样，加了一个execute immediate，意思是她给我立即执行这个命令，结果运行之后，还是跟上面一样报错，根本就没有给我立即执行！我勒个去，是怎么了。百思不得其解，查找了很多资料都是这么个写法，但是不知道怎么我写的就不行，跟我过不去吧可能是。

既然此路不通，那就换路。放到存储过程里面不行，那我就放到存储过程外面，试一试行不行：

```sql
--创建临时表，放到存储过程外面
create global temporary table temp_1 as select * from table_from_b;
create or replace procedure proc_import_data is
begin
    --把A数据库中表的数据导入到临时表
    insert into temp_1 select * from table_from_a@database_link_name;
    --把临时表的数据导入到B数据库的表中
    insert into table_from_b select * from temp_1;
    commit;
end proc_import_data
```

结果奇迹发生，执行通过了！虽然还是不懂为什么放在里面不行，但是既然能用就这么着吧，好读书不求甚解。

存储过程这一部分的拦路虎总算是干掉了，然后就是后面的大boss定时任务了。执行了设置JOB的脚本后，确实可以实现定时执行存储过程：

```sql
--查看建立的JOB的ID
select job, what from user_jobs;

begin
    dbms_job.run(job_id); --这个job_id就是上面查询出来的
end
```

然后很happy地以为事情到此为止就解决了。但是真正的问题还在后面。

设置了JOB之后，通过下面的命令可以查看当前正在执行的JOB：

```sql
select * from dba_jobs_running;
```

建立了JOB之后，执行这个命令确实可以看到当前的job在跑，但是关闭了连接，比如关闭SQL Plus之后，再登陆数据库查看正在运行的job时，却发现没有之前建立的JOB？这个又是神马情况？草泥马似乎又要万马奔腾了...

出了问题还是得自己解决，问她为什么她根本不理你，机器就是这么冰冷！于是开始找相关资料，为什么JOB不允许了呢？这个问题的解决方案几乎都是一样的，就那么几个原因，具体可以参见这个链接[Why are my jobs not running?](https://community.oracle.com/thread/648581)。于是便开始一个个试，试了三四个之后，发现都不行，后面的几个看了看，估计也不是这些原因。因为我设置的JOB在不关闭连接的时候是可以实现定时执行的，但是一旦关闭连接之后，JOB就不跑了；也就是说，JOB的自动执行只在当前登录的会话中才有效，一旦会话关闭，JOB就失效了。但是为什么会跟会话有关系？查找无果之后，因为也没有太多时间可以花费在这个上面了，所以就放弃了这个思路。

## **2. DBMS_SCHEDULER**
同样，此路不通就得考虑换道了。既然这个DBMS_JOB不行，那就换别的方式。Oracle 10g之后，引入了`DBMS_SCHEDULER`来代替上面的`DBMS_JOB`，提供了更加强大的功能。强大不强大暂且顾不上了，既然有新的方法可以用，那就赶紧拿来试一试。

一开始找到的是这个网址[Oracle FAQ](http://www.orafaq.com/wiki/DBMS_SCHEDULER)，上面提到了一些关于使用`DBMS_SCHEDULER`来创建JOB实现定时任务的，但是内容比较少，后来参考的是官方文档，[Examples of Using the Scheduler](https://docs.oracle.com/cd/B28359_01/server.111/b28310/schedadmin006.htm#ADMIN12063)和[DBMS_SCHEDULER](https://docs.oracle.com/cd/B28359_01/appdev.111/b28419/d_sched.htm#i1000363)，这两个地方对`DBMS_SCHEDULER`进行了详细的说明。因为这里我只考虑实现JOB的方式，因此其他的什么Program、Chain等一些其他的方式，太多了也看不过来。最后写出来的其实也很简单：

```sql
begin
    dbms_scheduler.create_job(
        job_name        => 'import_data',
        job_type        => 'STORED_PROCEDURE',
        job_action      => 'proc_import_data',
        repeat_interval => 'freq=MINUTELY;INTEVERAL=2', --每两分钟执行一次
        enabled         => TRUE,
        auto_drop       => FALSE
    );
end;
```

写好了SQL文件之后，直接执行就OK了。上面没有指定首次执行时间，那么默认的就是立即执行。恩，就是这么简单。当然，为了防止出现上面的问题，首先得检查一下当前的JOB是不是在跑。下面的命令可以查询JOB的状态和一些属性：

```sql
select job_name, last_start_date, next_run_date, enabled from user_scheduler_jobs;
```

可以查看当前用户的所有JOB、上一次开始执行的时间、下一次将要执行的时间、是否可用等信息，如果要查看更多的信息，可以参考上面的官方文档。写好了之后，执行这个命令，确实JOB是在跑了，下一次执行的时间也没有问题。就是有一个问题，应该是执行完这个SQL文件之后，就会立即执行的，也就是应该就会开始导数据了，但是查看数据库里的表，却发现没有数据过来，也就是没有立即执行！我擦，不会这个也不行吧？而且过了2分钟之后，数据也没有导进来，但是过了4分钟，开始有数据了。也就是这里的数据导入的似乎有延迟？怎么会这样呢？哪里不对？

如果这个也不行，那就真是要疯了，一定要弄出来啊。既然它不能立即执行，那么就手动让它执行，这样总可以吧。

```sql
exec dbms_scheduler.run_job('import_data'); --这里的import_data就是上面指定的JOB的名称
```

执行了这个命令之后，数据导过来了，因为系统涉及的数据量略大，这条命令执行了有十分钟左右的样子。不知道为什么会这么慢，使用前面提到的`DBMS_JOB`这种方式的话，导入数据秒秒钟就过来了，这个怎么这么慢？管不了那么多了，可以用就OK了，先解决温饱问题。剩下来需要解决的就是延迟的问题了，这里的需求是每天晚上12点开始导数据，因此我们设置好了JOB，并且让它执行了之后，查看了状态和下一次执行的时间，都没有问题；而且关闭了连接之后再查看JOB还在跑着，那就等着第二天过来看看有没有数据过来了。

折腾了这么久，就等着这一下了。第二天过来查数据，导过来了！终于搞定了。解决一个问题真不容易。但是发现数据量和之前导的相比，明显少了很多。不知道是不是导数据的过程中有一些失败了，没有成功导过来；或者就是数据量就是这么少。但是折腾了这么久，也顾不上了，既然能导数据就可以了。下一次再好好研究一下，这次实在是时间太仓促，而且具体这个有没有延迟还没有搞清楚，延迟多久也暂时不了解。太多细节还不明确，先这么着吧，至少这个路子是走通了。

## **3. 总结**
折腾了两天，“算是搞定了”数据定时导入的这么一个功能。但是很多细节还是没有研究透，先放着吧，以后再好好研究一下。总结一下这两天的收获：

- 如果一个问题用一种方法尝试太久还得不到解决，应该换一种方法。此路不通，就要换道。时间毕竟有限。
- 尝试用最新的实现方式，比如这里的`DBMS_SCHEDULER`，虽然`DBMS_JOB`应该也是可行的。像这次用后者出现了问题，解决起来很棘手。
- 办法总是比问题多，多尝试总是没有错的。在学习中成长。

The end.
