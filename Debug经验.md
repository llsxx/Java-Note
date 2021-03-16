### MybatisGenerator生成mapper代码重复

#### 解决措施

生成器的配置文件里的数据库连接地址(就是jdbcUrl)中添加这个参数nullCatalogMeansCurrent：

> <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver" connectionURL="jdbc:mysql://127.0.0.1:3306/vhr?useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=UTC" userId="root" password="123456">
> 			<property name="nullCatalogMeansCurrent" value="true"/>    
> </jdbcConnection>

#### MybatisGenerator使用

1. 修改generator.xml文件中的
   + <jdbcConnection>标签中的driverClass、connectionURL
   + 修改mapper,model,sqlmap对应的生成路径
   + 修改<table>标签项
   + 注意<classPathEntry>标签中数据库驱动对应
2. 运行  java -jar lib/mybatis-generator-core-1.3.1.jar -configfile generator.xml -overwrite (写入.bat文件时，直接运行)

#### 总结

==出错时，首先注意认真查看运行时的输出信息==

