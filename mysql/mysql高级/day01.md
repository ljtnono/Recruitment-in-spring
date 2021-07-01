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
DROP PROCEDURE [IF EXISTS] procedure_name;
```

### 4.6 语法

存储过程是可以编程的，意味着可以使用变量，表达式，控制结构，来完成比较复杂的功能

#### 4.6.1 变量

* DECLARE

通过DECLARE可以定义一个局部变量，该变量的作用范围只能在BEGIN...END块中

```mysql
DECLARE var_name[,...] type [DEFAULT value]
```

例如：

```mysql
delimiter $
create procedure pro_test2()
begin
	declare num int default 5;
	select num+10;
end $
delimiter ;
```

* SET

直接复制使用SET，可以赋值常量或者表达式，具体语法如下：

```mysql
SET var_name = expr [, var_name = expr] ...
```

示例：

```mysql
DELIMITER $

CREATE PROCEDURE pro_test3()
BEGIN
	DECLARE NAME VARCHAR(20);
	SET NAME = 'MYSQL';
	SELECT NAME;
END $

DELIMITER ;
```

也可以通过SELECT ... INTO 方式进行赋值操作：

```mysql
DELIMITER $

CREATE PROCEDURE pro_test5() 
begin 
  DECLARE count_num int; 
  SELECT COUNT(*) INTO count_num FROM city; 
  SELECT count_num; 
END$

DELIMITER ;
```

#### 4.6.2 if条件判断

语法结构：

```mysql
if search_condition then statement_list

	[elseif search_condition then statement_list] ...
	
	[else statement_list]
	
end if;
```

需求：

```mysql
-- 根据定义的身高变量，判定当前升高的所属的身材类型
-- 180及以上 --------------》身材高挑
-- 170-180 --------------》标准身材
-- 170一下 --------------》一般身材
```

示例：

```mysql
DELIMITER $

CREATE PROCEDURE pro_test6()
BEGIN

DECLARE height int DEFAULT 181;
IF height >= 180 THEN
	SELECT '身材高挑';
ELSEIF height >= 170 THEN
	SELECT '标准身材';
ELSE 
	SELECT '一般身材';
END IF;
-- ENDIF 后面的分号不能掉
END $

DELIMITER ;
```

#### 4.6.3 传递参数

语法格式：

```mysql
CREATE PROCEDURE procedure_name([in/out/inout] 参数名 参数类型)
...

IN:			该参数可以作为输入，也就是需要调用方传入值，默认
OUT:		该参数作为输出，也就是该参数可以作为返回值
INOUT: 	既可以作为输入参数，也可以作为输出参数
```

**IN - 输入**

需求：

根据定义的身高变量，判定当前身高的所属的身材类型

示例：

```mysql
DELIMITER $

CREATE PROCEDURE pro_test5(in height int)
BEGIN
	DECLARE description varchar(50) default '';
	if height >= 180 then
		SET descirption = '身材高挑';
	elseif height >= 170 then
		SET descritpion = '标准身材';
	else
		SET description = '一般身材';
	end if;
	SELECT CONCAT('身高', height, '对应的身材类型为：', description);
END $

DELIMITER ;
```

**OUT - 输出**

需求：

根据传入的身高变量，获取当前身高的所属的身材类型

示例：

```mysql
CREATE PROCEDURE pro_test5(in height int, out description varchar(100))
BEGIN
if height >= 180 THEN
	SET description='身材高挑';
elseif height >= 170 THEN
	SET description='标准身材';
else
	SET description='一般身材';
end if;
end$
```

调用：

```mysql
call pro_test5(168, @description)$
select @description$
```

Tips:

@description：这种变量要在变量名称前面加上“@”符号，叫做用户会话变量，代表整个会话过程他都是具有作用的，整个类似于全局变量一样。

@@global.sort_buffer_size：这种在变量前加上“@@”符号，叫做系统变量

#### 4.6.4 case结构

语法结构：

```mysql
-- 方式一：
CASE case_value
	WHEN when_value THEN statement_list
	[WHEN when_value THEN statement_list] ...
	[ELSE statement_list]
END CASE;

-- 方式二：
CASE
	WHEN search_condition THEN statement_list
	[WHEN search_condition THEN statement_list] ...
	[ELSE statement_list]
END CASE;
```

需求：

给定一个月份，然后计算出所在的季度

示例：

```mysql
DELIMITER $

CREATE PROCEDURE pro_test9(in month int)
BEGIN
	DECLARE result varchar(20);
	CASE
		WHEN month >= 1 and month <= 3 THEN
			SET result = '第一季度';
		WHEN month >= 4 and month <= 6 THEN
			SET result = '第二季度';
		WHEN month >= 7 and month <= 9 THEN
			SET result = '第三季度';
		WHEN month >= 10 and month <= 12 THEN
			SET result = '第四季度';
	END CASE;
	SELECT CONCAT('您输入的月份为:', month, '该月份：', result) AS content;
END $

DELIMITER ;
```

#### 4.6.5 while循环

语法结构：

```mysql
DELIMITER $

CREATE PROCEDURE pro_test8(in n int)
BEGIN
	DECLARE total int default 0;
	DECLARE num int default 1;
	WHILE num <= n DO
		SET total = total = num;
		SET num = num + 1;
	END WHILE;
	SELECT total;
END $

DELIMITER $
```

#### 4.6.6 repeat结构

有条件的循环控制语句，当满足条件的时候退出循环。while是满足条件才执行，repeat是满足条件就退出循环。

语法结构：

```mysql
REPEAT
	statement_list
	UNTIL search_condition
END REPEAT;
```

需求：

计算从1加到n的值

示例：

```mysql
DELIMITER $

CREATE PROCEDURE pro_test10(in n int)
BEGIN
	DECLARE total int default 0;
	REPEAT
		SET total = total + n;
		SET n = n - 1;
		UNTIL n = 0
	END REPEAT;
	SELECT total;
END $

DELIMITER ;
```

#### 4.6.7 loop语句

LOOP实现简单的循环，退出循环的条件需要使用其他语句定义，通常可以使用LEAVE语句实现，具体语法如下：

```mysql
[begin_label:] LOOP
	statement_list
END LOOP [end_label]
```

如果不再statement_list中增加退出循环的语句，那么LOOP语句可以用来实现简单的死循环。

#### 4.6.8 leave语句

用来从标注的流程构造中退出，通常和BEGIN...END或者循环一起使用。下面是一个使用LOOP和LEAVE的简单例子，退出循环：

```mysql
DELIMITER $

CREATE PROCEDURE pro_test11(in n int)
BEGIN
	DECLARE total int default 0;
	ins: LOOP
	
		IF n <= 0 THEN
			LEAVE ins;
		END IF;
		
		SET total = total + n;
		SET n = n - 1;
	END LOOP ins;
END $

DELIMITER ;
```

#### 4.6.9 游标/光标

游标是用来存储查询结果集的数据类型，在存储过程和函数中可以使用光标对结果集进行循环的处理。光标的使用
包括光标的声明、OPEN、FETCH和CLOSE，其语法分别如下。

声明光标：

```mysql
DECLARE cursor_name CURSOR FOR select_statement;
```

OPEN光标：

```mysql
OPEN cursor_name;
```

FETCH光标：

```mysql
FETCH cursor_name INTO var_name [, var_name] ...
```

CLOSE光标：

```mysql
CLOSE cursor_name;
```

示例：

初始化脚本：

```mysql
CREATE TABLE emp(
	id int(11) NOT NULL AUTO_INCREMENT,
  name varchar(50) NOT NULL COMMENT '姓名',
  age int(11) COMMENT '年龄',
  salary int(11) COMMENT '薪水',
  primary key('id')
) ENGINE=innodb DEFAULT CHARSET=utf8;

INSERT INTO emp(id, name, age, salary) VALUES(null,'金毛狮王',55,3800),(null,'白眉鹰
王',60,4000),(null,'青翼蝠王',38,2800),(null,'紫衫龙王',42,1800);
```



```mysql
-- 查询emp表中数据，并逐行获取进行展示
DELIMITER $

CREATE procedure pro_test11()
BEGIN
	DECLARE e_id int(11);
	DECLARE e_name varchar(50);
	DECLARE e_age int(11);
	DECLARE e_salary int(11);
	DECLARE emp_result cursor for SELECT * FROM emp;
	
	OPEN emp_result;
	FETCH emp_result INTO e_id, e_name, e_age, e_salary;
	SELECT CONCAT('id=', e_id, ', name=', e_name, ', age=', e_age, ', 薪资为：', e_salary);
	FETCH emp_result INTO e_id, e_name, e_age, e_salary;
	SELECT CONCAT('id=', e_id, ', name=', e_name, ', age=', e_age, ', 薪资为：', e_salary);
	FETCH emp_result INTO e_id, e_name, e_age, e_salary;
	SELECT CONCAT('id=', e_id, ', name=', e_name, ', age=', e_age, ', 薪资为：', e_salary);
	FETCH emp_result INTO e_id, e_name, e_age, e_salary;
	SELECT CONCAT('id=', e_id, ', name=', e_name, ', age=', e_age, ', 薪资为：', e_salary);
	FETCH emp_result INTO e_id, e_name, e_age, e_salary;
	SELECT CONCAT('id=', e_id, ', name=', e_name, ', age=', e_age, ', 薪资为：', e_salary);
	
	CLOSE emp_result;
END $

DELIMITER ;
```

通过循环结构，获取游标中的数据

```mysql
```





### 4.7 存储函数

语法结构：

```mysql
CREATE FUNCTION function_name([param type ...])
RETURNS type
BEGIN
	...
END;
```

案例：

定义一个存储过程，请求满足条件的总记录数；

```mysql
DELIMITER $

CREATE function count_city(countryid int)
RETURNS int
BEGIN
	DECLARE cnum int;
	SELECT COUNT(*) INTO cnum FROM city WHERE country_id = countryId;
	return cnum;
END $

DELIMITER ;
```

调用：

```mysql
SELECT count_city(1);
SELECT count_city(2);
```

