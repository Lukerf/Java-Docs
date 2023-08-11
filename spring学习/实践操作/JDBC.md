## 介绍

​	Spring Boot访问数据库，常用的方式有Mybaits、Hibernate以及Spring Boot提供的JDBC这三种方式。其中，Spring JDBC，是Spring中最基本、最底层的访问数据库的实现方式。

## 数据源配置

	1. pom文件引入引入数据库驱动依赖
 	2. pom文件引入JDBC依赖
 	3. 在properties中配置数据库连接信息
 	4. 使用时通过自动注入获取datasource数据源

## 连接池