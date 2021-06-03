## 4. 存储过程和函数

### 4.1 存储过程和函数概述

​		存储过程和函数是事先经过编译并存储在数据库中的一段SQL语句的集合，调用存储过程和函数可以简化应用开发人员的很多工作，减少数据在数据库和应用服务器之间的传输，对于提高数据处理的效率是有好处的。

​		存储过程和函数的区别在于函数必须有返回值，而存储过程没有。

​		函数：有返回值；

​		过程：没有返回值；

### 4.2 创建存储过程

```mysql
CREATE PROCEDURE procedure_name([proc_paramter])
begin
		-- SQL语句
end;
```

示例：

```mysql
delimiter $
-- 由于在mysql中, 默认使用的是分号作为结束符，而在存储过程中的begin与end之间是多条sql语句，所以需要改变结束符
use mysql$
CREATE PROCEDURE pro_test()
begin
	SELECT * FROM user WHERE Host = 'localhost';
end $
```

Tips:

1、在mysql8中，创建存储过程需要使用use语句指定某个数据库，否则会报**ERROR 1046 (3D000): No database selected**

### 4.3 存储过程的调用

```mysql
call procedure_name();
-- 使用call关键字可以调用存储过程
```

### 4.4 查看存储过程

```mysql
-- 查看存储过程的状态信息
show procedure status like 'procedure_name';
-- 例如：
show procedure status like 'pro_test1'\G ;
*************************** 1. row ***************************
                  Db: mysql
                Name: pro_test1
                Type: PROCEDURE
             Definer: root@%
            Modified: 2021-06-04 00:46:18
             Created: 2021-06-04 00:46:18
       Security_type: DEFINER
             Comment: 
character_set_client: utf8mb4
collation_connection: utf8mb4_0900_ai_ci
  Database Collation: utf8mb4_0900_ai_ci
  
-- 查看存储过程的定义
show create procedure db_name.procedure_name \G;
-- 例如：
show create procedure mysql.pro_test1 \G;
*************************** 1. row ***************************
           Procedure: pro_test1
            sql_mode: STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
    Create Procedure: CREATE DEFINER=`root`@`%` PROCEDURE `pro_test1`()
begin SELECT 'Hello World'; end
character_set_client: utf8mb4
collation_connection: utf8mb4_0900_ai_ci
  Database Collation: utf8mb4_0900_ai_ci
```

Tips:

在mysql5中，存储过程存放在mysql数据库中的proc表中，然而在mysql8中并没有这张表

### 4.5 删除存储过程

```mysql
```

