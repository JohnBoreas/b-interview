# 简述Myisam和Innodb的区别？

InnoDB存储引擎: 主要面向OLTP(Online Transaction Processing，在线事务处理)方面的应用，是第一个完整支持ACID事务的存储引擎(BDB第一个支持事务的存储引擎，已经停止开发)。
特点：

1 支持行锁
2 支持外键
3 支持自动增加列AUTO_INCREMENT属性
4 支持事务
5 支持MVCC模式的读写
6 读的效率低于MYISAM
7.写的效率高优于MYISAM
8.适合频繁修改以及设计到安全性较高的应用
9.清空整个表的时候，Innodb是一行一行的删除，

MyISAM存储引擎: 是MySQL官方提供的存储引擎，主要面向OLAP(Online Analytical Processing,在线分析处理)方面的应用。

特点：

1 独立于操作系统，当建立一个MyISAM存储引擎的表时，就会在本地磁盘建立三个文件，例如我建立tb_demo表，那么会生成以下三个文件tb_demo.frm,tb_demo.MYD,tb_demo.MYI
2 不支持事务，
3 支持表锁和全文索引
4 MyISAM存储引擎表由MYD和MYI组成，MYD用来存放数据文件，MYI用来存放索引文件。MySQL数据库只缓存其索引文件，数据文件的缓存交给操作系统本身来完成；
5 MySQL5.0版本开始，MyISAM默认支持256T的单表数据；
6.选择密集型的表：MYISAM存储引擎在筛选大量数据时非常迅速，这是他最突出的优点
7.读的效率优于InnoDB
8.写的效率低于InnoDB
9.适合查询以及插入为主的应用
10.清空整个表的时候，MYISAM则会新建表


# 