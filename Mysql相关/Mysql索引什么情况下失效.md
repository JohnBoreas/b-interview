#### 索引什么情况下失效？

索引失效条件？

写sql时候需要注意什么？



答：

1、查询条件包含or，可能导致索引失效

2、如何字段类型是字符串，where时一定用引号括起来，否则索引失效

3、like通配符可能导致索引失效

4、在索引列上使用mysql的内置函数，索引失效

5、索引字段上使用运算（如，+、-、*、/）（！= 或者 < >，not in），is null， is not null时，可能会导致索引失效

6、左连接查询或者右连接查询查询关联的字段编码格式不一样，可能导致索引失效



其他：查询数据过多，超过表的30%；字段的索引不好