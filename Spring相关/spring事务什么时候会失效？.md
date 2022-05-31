#### spring事务什么时候会失效？

答：存在异常，异常被捕获或者配置错误，没有被spring管理，数据库不支持，未使用public



详解：

		1、bean对象没有被spring容器管理
	
		2、方法的访问修饰符不是public（@Transactional 只用于public，否则失效，非public，可开启AspectJ 代理模式）
	
		3、自身调用问题（使用this调用本类发方法，此时这个this对象不是代理类，而是UserService对象本身）
	
		4、数据源没有配置事务管理器
	
		5、数据库不支持事务（MyISAM不支持）
	
		6、异常被捕获（异常被catch掉，事务不会回滚）
	
		7、异常类型错误或者配置错误（抛出的异常没有被定义，默认为RuntimeExcelption）

