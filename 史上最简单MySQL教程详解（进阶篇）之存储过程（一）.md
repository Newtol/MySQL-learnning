# 史上最简单MySQL教程详解（进阶篇）之存储过程（一）

## 什么是存储过程

SQL基本是一个命令实现一个处理的，是不能编写处理流程的。虽然通过子查询、多表连接等方式能实现一些高级的功能，但是具有很大的局限性。对于SQL本身是很难实现针对不同条件进行不同的处理或者循环等功能。即使能够实现，也是十分复杂或者对于性能有极大的影响。**存储过程（Stored Procedure）**就应运而生，它可以由SQL语句和各种条件判断、循环控制等语句组成，简单的说:存储过程像相互之间有联系的SQL语句组成的“小程序”。因为SQL的执行是需要经历一个：**解析->编译->执行**的过程的。存储过程是提前经过解析和编译存储在数据库中的，用户通过指定存储过程的名字并给定参数（如果该存储过程带有参数）来调用执行它。

## 存储过程的作用

+ 提高执行性能：

  因为存储过程是提前创建并保存在数据库中的，当执行命令时，就能免去解析和编译的过程，能减轻数据库的负担，提高执行性能。

+ 可减轻网络负担：

  因为存储过程的执行，只需要客户端传递对应的参数即可，并不需要多次传递SQL命令本身，可以大大的减轻网络负担，减少彼此之间的通信量，整体提高数据库性能。

+ 提高数据库的安全性：

  存储过程禁止了对表本身的访问，只赋予用户对相关存储过程的访问权限。同时存储过程将数据处理部分“黑匣子化”后，程序本身的可读性和简洁性都会有所增加。用户不需要考虑存储过程的内部实现，只需要知道应该调用哪个存储过程即可。

## 如何使用存储过程

### 创建存储过程

创建存储过程使用的是【CREATE】语句，具体语法如下：

```sql
CREATE PROCEDURE 存储过程名（
	参数1的种类 参数名1 参数1的数据类型，
	参数2的种类 参数名2 参数2的数据类型
）
BEGIN 
	数据处理过程
END
```

上面提到的参数种类，主要分为下面三种类型：

| 参数种类 | 说明                       |
| -------- | -------------------------- |
| IN       | 输入参数（可省略）         |
| OUT      | 输出参数                   |
| INOUT    | 既是输出参数，也是输入参数 |

对于参数的数据类型，可参考这篇博文：[ 史上最简单MySQL教程详解（基础篇）之常用表操作和表参数介绍](https://blog.csdn.net/m0_37888031/article/details/80450393)



在创建存储过程之前呢，还有下面几个要点需要掌握。

#### DELIMITER改变分隔符

【DELIMITER】是MySQL用来改变监视器中分离符的命令。默认的分隔符是【;】，但是，存储过程本身就是**命令的集合**，所以一定还会含有其他的分隔符。那么它们彼此之间冲突和混淆，所以在创建存储过程之前，我们需要将默认符换成一个完全无关的符号，只要不与其他关键字发生歧义即可，通常使用的是【//】。在完成创建之后，我们再将其回复即可。但需要提醒的是：分隔符的改变只会在启动期间有效，重新启动后，会自动恢复到默认状态。

#### 可使用的控制语句

1. 简单分支

   ```SQL
   IF 条件表达式1
   	条件表达式1为TRUE时执行的命令
   [ELSEIF 条件表达式N
       条件表达式N为TRUE时执行的命令
   ]
   [ELSE
       全部为False时执行的命令
   ]
   END IF
   ```

2. 多重分支

   ```sql
   CASE 表达式1
   	WHEN 值1 THEN 表达式1 = 值1时执行的命令
   	...
   	WHEN 值N THEN 表达式1 = 值N时执行的命令
   	[ELSE 上述所有值以外执行的命令]
   END CASE
   ```

3. 循环控制（后置判断）

   ```SQL
   REPEAT
   	直至条件表达式为True时执行的命令
   UNTIL 
   	条件表达式 
   END REPEAT
   ```

4. 循环控制（前置判断）

   ```sql
   WHILE 条件表达式 DO
   	系列命令
   END WHILE
   ```

#### 开始创建存储过程

```sql
mysql> DELIMITER //
	-> CREATE PROCEDURE sp_student (IN p_name VARCHAR(20))
    -> BEGIN
    -> 	IF p_name IS NULL OR p_name = "" THEN 
    -> 		SELECT * FROM student ;
    -> 	ELSE 
    -> 		SELECT * FROM student WHERE studentName LIKE p_name;
    -> 	END IF ;
    -> END 
    -> //
Query OK, 0 rows affected
mysql> DELIMITER ;
```

这样我们就创建了一个【sp_student】的存储过程。如果我们传入参数【p_name】，那么他就会进行模糊查询，否则就无条件查询。

### 确认存储过程

我们确认存储过程使用的是【SHOW】语句，具体语法如下：

```SQL
SHOW PROCEDURE STATUS \G
```

例如：

```sql
mysql> SHOW PROCEDURE STATUS \G
*************************** 27. row ***************************
                  Db: test1
                Name: sp_student
                Type: PROCEDURE
             Definer: root@localhost
            Modified: 2018-09-13 12:03:13
             Created: 2018-09-13 12:03:13
       Security_type: DEFINER
             Comment:
character_set_client: utf8
collation_connection: utf8_general_ci
  Database Collation: utf8_unicode_ci
27 rows in set (0.00 sec)
```

结果集参数解释：

| 参数                 | 说明                                              |
| -------------------- | ------------------------------------------------- |
| Db                   | 所属数据库名                                      |
| Name                 | 存储过程名                                        |
| Type                 | 种类（PROCEDURE:存储过程/FUNCTION:函数）          |
| Definer              | 创建者                                            |
| Modified             | 最终更新时间                                      |
| Created              | 创建时间                                          |
| Security_type        | 安全种类(DEFINER：存储过程权限与创建用户权限一致) |
| Comment              | 备注                                              |
| character_set_client | 从客户端发送过来查询的字符编码                    |
| collation_connection | 当前连接中使用的校对顺序                          |
| Database Collation   | 数据库的校对顺序                                  |

除此之外，还能使用下面的语句确认:

```SQL
SHOW CREATE PROCEDURE 存储过程名\G
```

例如：

```sql
mysql> SHOW CREATE PROCEDURE sp_student\G
*************************** 1. row ***************************
Procedure: sp_student
sql_mode: STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER
    Create Procedure: CREATE DEFINER=`root`@`localhost` PROCEDURE `sp_student`(IN p_name VARCHAR(20))
BEGIN

IF p_name IS NULL OR p_name = "" THEN

SELECT * FROM student ;

ELSE

SELECT * FROM studnet WHERE studentName LIKE p_name;

END IF ;

END
character_set_client: utf8
collation_connection: utf8_general_ci
Database Collation: utf8_unicode_ci
1 row in set (0.00 sec)
```

这个会返回一些安全的设置信息，并且名称都会被单引号（‘ ’）括起来。

### 使用存储过程

使用存储过程，使用的是【CALL】命令，具体语法如下：

```sql
CALL 存储过程名（参数1，...）;
```

例如：

```sql
mysql> CALL sp_student('路人%');    //提交了参数
+-----------+-------------+--------------+----------------+
| studentId | studentName | studentPhone | studentAddress |
+-----------+-------------+--------------+----------------+
| 4         | 路人甲      | 132          | 广州           |
| 5         | 路人乙      | 118          | 深圳           |
+-----------+-------------+--------------+----------------+
2 rows in set

Query OK, 0 rows affected

mysql> CALL sp_student('');    //未提交参数
+-----------+-------------+--------------+----------------+
| studentId | studentName | studentPhone | studentAddress |
+-----------+-------------+--------------+----------------+
| 1         | 张三        | 140          | 重庆           |
| 2         | 李四        | 137          | 北京           |
| 3         | 王五        | 135          | 上海           |
| 4         | 路人甲      | 132          | 广州           |
| 5         | 路人乙      | 118          | 深圳           |
+-----------+-------------+--------------+----------------+
5 rows in set
Query OK, 0 rows affected

```

### 删除存储过程

删除已经创建的存储过程使用【DROP】语句，具体语法如下：

```sql
DROP PROCEDURE 存储过程名;
```

例如：

```sql
mysql> DROP PROCEDURE sp_student;
Query OK, 0 rows affected (0.00 sec)
```



参考文献：

[MySQL存储过程详解  ](http://blog.sina.com.cn/s/blog_52d20fbf0100ofd5.html)

《MySQL高效编程》

