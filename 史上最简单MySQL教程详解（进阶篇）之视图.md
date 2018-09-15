[TOC]

# 史上最简单MySQL教程详解（进阶篇）之视图

## 为什么要用视图

在设计数据库的过程中，为了防止数据的冗长性（即：同样的数据在多个表中重复出现的情况），就需要去遵守我们在前面的文章[史上最简单MySQL教程详解（基础篇）之数据库设计范式及应用举例](https://blog.csdn.net/m0_37888031/article/details/80620267)中介绍的有关数据库设计的一些规范，这样虽然让表的设计更加合理，但是我们可能会使用到**多表连接**等方式查询，就会增加SQL语句的复杂度，从而增大服务器的负担，降低查询效率，增加响应时间。

例如：我们有如下几个字段（学号，学生姓名，电话号码，家庭住址，课程编号，课程名字，授课教师...），请设计表结构来实现如下需求：能够查询出所有的学号，电话号码，及其选则的所有课程名字和授课教师。

可能第一反应是：【student】表为（学号，学生姓名，电话号码，家庭住址）和【course】表（学号，电话号码，所选的课程编号、课程名字、授课教师），因为这样能够快速的查询出我们所需要的数据，但是，假设某个学生的电话号码变化了呢？是不是又需要更改很多的表？而且这样冗杂的数据，根据设计规范也是不允许的。

所以，为了防止上述情况的发生，我们可以设计成这样【student】表（学号，学生姓名，电话号码，家庭住址），【course】表（课程编号，课程名字，授课教师），【choose】(学号，课程编号)。

当我们像这样设计数据表时，就遵循了数据库的设计规范，但同时我们在查询的时候就需要联合【student】和【course】表才能查询。所以，当这样我们需要一次性查询多个表的数据，但是为了表的规范，又没有办法将需要查询的数据保存在一个表中的时候，就会使用到**视图（View）**。

## 视图的本质

视图（View）的本质是将我们的Select语句的检索结果用表的形式保存下来，所以有时候视图又称为假表或者伪表。这是因为视图本身其实是不包含数据的，仅仅从对象表中动态地抽取数据，并将数据组织在一起，看上去和我们平时使用的表一样。

## 视图的作用

（1）可以只公开表中的特定行或者列：通过限制用户对实际表的SELECT操作权限，而仅赋予用户对相应视图的SELECT操作权限，来达到限制用户只读取表中特定行或列的目的。（因为一般视图在使用过程中，抽取的正是表中的一些特定行或列，而非整张表）

（2）简化SQL查询的复杂度，增强可读性：使用了视图后，视图就已经成为了检索后的假想表，省去了编写复杂SQL查询语句的过程，直接查询视图中的内容即可。当连接或者子查询发生改变时，只需要修改视图的定义，就可以大大减少修改的范围。

（3）可以限制可插入或更新的范围：在定义视图时加入【WITH CHECK OPTION】命令后，使用【INSERT】或者【UPDATE】命令操作数据时，数据库都会进行强制检查，不符合视图定义的数据将被限制操作（即：操作后的数据是否还在视图中，如果操作后不存在，则禁止操作）。如果没有加入WITH CHECK OPTION】，则不会进行限制。

## 如何使用视图

我们先创建下面三个表：

1. student：

| 列名           | 解释 |
| -------------- | ---- |
| studentId      | 学号 |
| studentName    | 名字 |
| studentPhone   | 电话 |
| studentAddress | 地址 |

```sql
CREATE TABLE `student` (
  `studentId` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `studentName` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `studentPhone` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `studentAddress` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  PRIMARY KEY (`studentId`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```

2. course

   | 列名          | 说明     |
   | ------------- | -------- |
   | courseId      | 课程Id   |
   | courseName    | 课程名   |
   | courseTeacher | 授课教师 |

   ```sql
   CREATE TABLE `course` (
     `courseId` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
     `courseName` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
     `courseTeacher` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
     PRIMARY KEY (`courseId`)
   ) ENGINE=MyISAM DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
   ```

3. choose

   | 列名      | 说明   |
   | --------- | ------ |
   | courseId  | 课程ID |
   | studentId | 学号   |

   ```sql
   CREATE TABLE `chooese` (
     `courseId` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
     `studentId` varchar(255) COLLATE utf8_unicode_ci NOT NULL
   ) ENGINE=MyISAM DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
   
   ```

然后，我们随便填入几种数据，我们按照以往的方法，我们执行的SQL查询语句为：

```SQL
mysql> Select stu.studentName AS name,
	   stu.studentPhone AS phone ,
	   co.courseName AS coname ,
	   co.courseTeacher AS teacher 
FROM (			
		(
		choose AS choo INNER JOIN student AS stu ON choo.studentId = 						 stu.studentId)
      INNER JOIN course AS co ON co.courseId = choo.courseId);
+--------+-------+--------+---------+
| name   | phone | coname | teacher |
+--------+-------+--------+---------+
| 张三   | 139   | 英语   | 张老师  |
| 李四   | 137   | 英语   | 张老师  |
| 王五   | 135   | 数学   | 李老师  |
| 路人乙 | 118   | 语文   | 王老师  |
| 李四   | 137   | 语文   | 王老师  |
+--------+-------+--------+---------+
5 rows in set
```



### 创建视图

创建视图使用的是【CREATE VIEW】命令，具体语法如下：

```SQL
CREATE VIEW 视图名（列名1，列名2...）AS 查询语句 [WITH CHECK OPTION];
```

例如，我们给刚才创建的表创建视图：

```sql
 CREATE VIEW result(name,phone,coname,teacher) AS 
 Select stu.studentName 
 	AS name,stu.studentPhone AS phone ,
 	co.courseName AS coname ,
 	co.courseTeacher AS teacher FROM (	
        (
			choose AS choo INNER JOIN student AS stu ON choo.studentId = stu.studentId
		)INNER JOIN course AS co ON co.courseId = choo.courseId								) with check option;
Query OK, 0 rows affected
```

注意事项：创建视图时，SELECT命令有如下限制

+ 不能包含系统变量/用户变量的参照；
+ TEMPORARY类型的表
+ FROM语句后的子查询

使用【SHOW TABLES】即可查看我们刚才创建的视图

```sql
mysql> show tables;
+-----------------+
| Tables_in_test1 |
+-----------------+
| choose          |
| course          |
| result          |
| student         |
+-----------------+
4 rows in set
```

注意事项：

因为我们在使用【SHOW】命令查看的时候就会发现，视图和实际的表都是会显示的，这样就不容易分辨出究竟是视图还是实际的表，所以，建议对于视图来说，建议以**v_视图名**的方式来命名，这样就很容易识别。

### 修改视图

修改已经创建好的视图，使用【REPLACE】命令，具体语法如下：

```sql
CREATE OR REPLACE VIEW 视图名 AS SELECT [...] FROM [...];
```

例如：

```SQL
CREATE OR REPLACE VIEW v_result(name,phone,coname,teacher) AS 
 Select stu.studentName 
 	AS name,stu.studentPhone AS phone ,
 	co.courseName AS coname ,
 	co.courseTeacher AS teacher FROM (	
        (
			choose AS choo INNER JOIN student AS stu ON choo.studentId = stu.studentId
		)INNER JOIN course AS co ON co.courseId = choo.courseId								) with check option;
```

### 删除视图

删除已经创建的视图使用【DROP】命令，语法如下：

```sql
DROP VIEW 视图名;
```

例如：

```sql
mysql> DROP VIEW result;   //删除之前创建的result表
Query OK, 0 rows affected

mysql> SHOW TABLES;    //查看是否删除成功
+-----------------+
| Tables_in_test1 |
+-----------------+
| choose          |
| course          |
| student         |
| v_result        |
+-----------------+
4 rows in set
```

### 查看视图

查看视图所有列的信息和平时我们使用的表一样，使用【SHOW】命令，具体语法如下：

```sql
SHOW FIELDS FROM 视图名;
```

例如：

```sql
mysql> SHOW FIELDS FROM v_result;
+---------+--------------+------+-----+---------+-------+
| Field   | Type         | Null | Key | Default | Extra |
+---------+--------------+------+-----+---------+-------+
| name    | varchar(255) | NO   |     | NULL    |       |
| phone   | varchar(255) | NO   |     | NULL    |       |
| coname  | varchar(255) | NO   |     | NULL    |       |
| teacher | varchar(255) | NO   |     | NULL    |       |
+---------+--------------+------+-----+---------+-------+
4 rows in set
```

### 使用视图检索

使用视图检索的时候，和使用普通表一样，使用【SELECT】语句，例如：

```sql
mysql> SELECT * FROM v_result WHERE name = "李四";
+------+-------+--------+---------+
| name | phone | coname | teacher |
+------+-------+--------+---------+
| 李四 | 137   | 英语   | 张老师  |
| 李四 | 137   | 语文   | 王老师  |
+------+-------+--------+---------+
2 rows in set
```

### 变更视图数据

当我们对于视图中的数据进行插入、更新、删除等操作时，和实际的表方式相同，都使用的【INSERT】、【UPDATE】、【DELETE】语句。例如：

```sql
mysql> UPDATE v_result SET phone = '140' WHERE name = '张三';   //将张三的电话修改为‘140’
Query OK, 1 row affected
Rows matched: 1  Changed: 1  Warnings: 0
mysql> SELECT * FROM v_result ;    //视图中修改成功
+--------+-------+--------+---------+
| name   | phone | coname | teacher |
+--------+-------+--------+---------+
| 张三   | 140   | 英语   | 张老师  |
| 李四   | 137   | 英语   | 张老师  |
| 王五   | 135   | 数学   | 李老师  |
| 路人乙 | 118   | 语文   | 王老师  |
| 李四   | 137   | 语文   | 王老师  |
+--------+-------+--------+---------+
5 rows in set
mysql> SELECT * FROM student;    //实际表中的数据也修改成功
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

```



注意事项：在以下几种条件下不能进行插入/更新/删除操作。

+ 视图的列中含有统计函数的情况下；
+ 视图定义时使用了GROUP BY/HAVING语句，DISTINCT语句、UNION语句的情况下；
+ 视图定义时使用了子查询的情况下；
+ 进行跨越多个表进行数据的变更操作；

### WITH CHECK OPTION 分析

前面我们介绍过，【 WITH CHECK OPTION】可以用于限制可插入或更新的范围。如果我们在创建视图的时候使用了【 WITH CHECK OPTION】语句，那么当我们执行插入语句时,就会报错：【Can not modify more than one base table through a join view】,不能修改超过一个基础表以上的情况。例如：

```sql
mysql> INSERT INTO v_result(name,phone,coname,teacher) VALUES ('赵六','130','历史','赵老师');
1393 - Can not modify more than one base table through a join view 'test1.v_result'
```



所以为了避免不必要的混乱和不可预知的BUG,建议在创建视图时加上【 WITH CHECK OPTION】语句。



## 总结

视图是一个非常方便的功能，但是对于**性能**来说并非是一个最好的选择。视图可以简化复杂的SQL查询语句，但是并不能简化内部处理。另外，视图还可以在视图的基础上再次定义，但是这必将导致数据库性能的下降，所以还是酌情使用。