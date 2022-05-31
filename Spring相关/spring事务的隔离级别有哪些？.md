#### spring事务的隔离级别有哪些？

	spring中的事务隔离级别就是数据库的隔离级别，有以下几种：
	
	read uncommitted
	
	read committed
	
	repeatable read
	
	serializable
	
	在进行配置的时候，如果数据库和spring代码中的隔离级别不同，那么以spring的配置为主。

# 