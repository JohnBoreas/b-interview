#### mysql聚簇和非聚簇索引的区别是什么？

答：数据跟索引是否存储在一起



详解：

唯一键索引不一定是聚簇索引，非聚簇索引查询字段可能要回表查询

​		mysql的索引类型跟存储引擎是相关的，innodb存储引擎数据文件跟索引文件全部放在ibd文件中，而myisam的数据文件放在myd文件中，索引放在myi文件中，其实区分聚簇索引和非聚簇索引非常简单，只要判断数据跟索引是否存储在一起就可以了。

​		innodb存储引擎在进行数据插入的时候，数据必须要跟索引放在一起，如果有主键就使用主键，没有主键就使用唯一键，没有唯一键就使用6字节的rowid，因此跟数据绑定在一起的就是聚簇索引，而为了避免数据冗余存储，其他的索引的叶子节点中存储的都是聚簇索引的key值，因此innodb中既有聚簇索引也有非聚簇索引，而myisam中只有非聚簇索引。



其他：

更新主键的代价很高

Innodb通过主键聚集数据，如果没有定义主键，innodb会选择非空的唯一索引代替，没有唯一索引，会隐式的定义一个主键来作为聚簇索引。

非聚簇索引都是辅助索引，像复合索引、前缀索引、唯一索引

主键索引是一种聚簇索引

