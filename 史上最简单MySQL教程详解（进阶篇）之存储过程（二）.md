# 史上最简单MySQL教程详解（进阶篇）之存储过程（二）

## 前文回顾

在[史上最简单MySQL教程详解（进阶篇）之存储过程（一）](https://blog.csdn.net/m0_37888031/article/details/82691581)中，我们介绍了有关存储过程的一些基本知识，了解了存储过程的创建、使用和删除等。接下来，我们就将介绍一下使用存储过程中的输出参数的设置以及各种控制语句等。

## 定义输出参数

我们在上文介绍过，存储过程除了能够定义调用的参数外，还可以设置向调用方法的返回参数。首先，我们重新创建一个存储过程，这次我们加入输出参数【p_num】。

```sql
mysql> CREATE PROCEDURE sp_student2 (IN p_name VARCHAR(20),OUT p_num INT)
    -> BEGIN
    -> 	IF p_name IS NULL OR p_name = "" THEN
    -> 		SELECT * FROM student;
    -> 	ELSE
    -> 		SELECT * FROM student WHERE studentName LIKE p_name ;
    -> 	END IF;
    -> 	SELECT FOUND_ROWS() INTO p_num;
    -> END
    -> //
Query OK, 0 rows affected (0.00 sec)
mysql> DELIMITER ;
```

这样，我们就创建为名为【sp_student2】的存储过程，其中输入参数为【 p_name】，输出参数为【p_num】。

其中 【SELECT ...INTO ...】语句可以将SELECT语句中取得的结果设置到指定的变量中，具体语法如下：

```sql
SELECT 列名1... INTO 变量名1... FROM 表名 WHERE ...;
```

而【FOUND_ROWS】函数则可以取得前一条SELECT语句中检索出来的记录件数。

所以，【SELECT FOUND_ROWS() INTO p_num;】的含义就是将查询出来的结果的条数，存储在【 p_num】中。

例如：提示，当我们在使用OUT/INOUT参数时，在参数名的头部加上【@】

```sql
mysql> CALL sp_student2('路人%',@num);
+-----------+-------------+--------------+----------------+
| studentId | studentName | studentPhone | studentAddress |
+-----------+-------------+--------------+----------------+
| 4         | 路人甲      | 132          | 广州           |
| 5         | 路人乙      | 118          | 深圳           |
+-----------+-------------+--------------+----------------+
2 rows in set (0.04 sec)

Query OK, 1 row affected (0.04 sec)


```

当我们传入输出参数名后，我们就可以直接使用【SELECT】查询，例如：

```sql
mysql> SELECT @num;
+------+
| @num |
+------+
|    2 |
+------+
1 row in set (0.00 sec)
```

## 多重条件分支

上面我们介绍的都是单一的IF...ELSE...分支，但是实际情况中，我们可能会遇到多重分支的情况，那么在这种情况下，我们应该如何处理呢？

### 使用ELSEIF实现

在存储过程中，除了可以使用【IF...ELSE...】以外，也可以使用【IF...ELSEIF...ELSEIF...ELSE】的语句。

例如：

```sql
mysql> DELIMITER //
mysql> CREATE PROCEDURE sp_student3 (IN p_num INT)
    -> BEGIN
    -> 	IF p_num IS NULL OR p_num = "" THEN
    -> 		SELECT * FROM student;
    -> 	ELSEIF p_num = 1 THEN
    -> 		SELECT * FROM student WHERE studentName LIKE '王%';
    -> 	ELSEIF p_num = 2 THEN
    -> 		SELECT * FROM student WHERE studentName LIKE '张%';
    -> 	ELSE
    -> 		SELECT * FROM student WHERE studentName LIKE '路人%' ;
    -> 	END IF;
    -> END
    -> //
Query OK, 0 rows affected (0.00 sec)

mysql> DELIMITER ;

mysql> CALL sp_student3('1');
+-----------+-------------+--------------+----------------+
| studentId | studentName | studentPhone | studentAddress |
+-----------+-------------+--------------+----------------+
| 3         | 王五        | 135          | 上海           |
+-----------+-------------+--------------+----------------+
1 row in set (0.00 sec)

Query OK, 0 rows affected (0.00 sec)
```



### 使用CASE实现

当我们像上面一样，使用【ELSEIF】来实现多重条件，会显得比较的复杂和繁琐，我们还有一种更为简便的方式，那就是使用CASE来实现，例如：

```SQL
mysql> DELIMITER //
mysql> CREATE PROCEDURE sp_student4 (IN p_num INT)
    -> BEGIN
    -> 	CASE p_num
    -> 		WHEN 1 THEN
    -> 			SELECT * FROM student WHERE studentName LIKE '王%';
    -> 		WHEN 2 THEN
    -> 			SELECT * FROM student WHERE studentName LIKE '张%';
    -> 		WHEN 3 THEN
    -> 			SELECT * FROM student WHERE studentName LIKE '路人%';
    -> 		ELSE
    -> 			SELECT * FROM student WHERE studentName LIKE '李%';
    -> 	END CASE;
    -> END
    -> //
Query OK, 0 rows affected (0.00 sec)
mysql> DELIMITER ;

mysql> CALL sp_student4('5');
+-----------+-------------+--------------+----------------+
| studentId | studentName | studentPhone | studentAddress |
+-----------+-------------+--------------+----------------+
| 2         | 李四        | 137          | 北京           |
+-----------+-------------+--------------+----------------+
1 row in set (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

```

这样也能实现多重条件的判断，而且看上去更加的简洁。



## 定义本地变量

当我们回顾上面的两个例子，我们就会发现，有大量的重复代码：【SELECT * FROM student WHERE studentName LIKE... 】其中不同的只有检索的条件而已，那么我们应该如何优化这种情况呢？这个时候，**本地变量**就派上了用场。本地变量又称为局部变量，意为只能在存储过程中使用的变量，用于保存存储过程中的临时值。

变量的声明使用的是【DECLARE】，具体语法如下：

```SQL
DECLARE 变量名 数据类型[预期大小];
```

变量赋值使用的是【SET】，具体语法如下：

```sql
SET 变量名 = 值得;
```

例如：

```SQL
mysql> DELIMITER //
mysql> CREATE PROCEDURE sp_student5 (IN p_num INT)
    -> BEGIN
    -> 	DECLARE tmp VARCHAR(4);
    -> 	CASE p_num
    -> 		WHEN 1 THEN
    -> 			SET tmp = '王%';
    -> 		WHEN 2 THEN
    -> 			SET tmp = '张%';
    -> 		WHEN 3 THEN
    -> 			SET tmp = '李%';
    -> 		ELSE
    -> 			SET tmp = '路人%';
    -> 	END CASE;
    -> 	SELECT * FROM student WHERE studentName LIKE tmp;
    -> END
    -> //
Query OK, 0 rows affected (0.00 sec)

mysql> DELIMITER ;
mysql> CALL sp_student5('2');
+-----------+-------------+--------------+----------------+
| studentId | studentName | studentPhone | studentAddress |
+-----------+-------------+--------------+----------------+
| 1         | 张三        | 140          | 重庆           |
+-----------+-------------+--------------+----------------+
1 row in set (0.00 sec)

Query OK, 0 rows affected (0.01 sec)
```



## 使用循环语句

在存储过程中，可以使用【WHILE】和【REPEAT】两种命令实现循环。接下来，我们就利用这两种循环计算阶乘的方式来加深理解：

### WHILE 

```sql
mysql> DELIMITER //
mysql> CREATE PROCEDURE sp_test(IN p_num INT,OUT p_result INT)
    -> BEGIN
    -> 	SET p_result = 1;               			 //输出变量赋值为1
    -> 	WHILE p_num > 1 DO               			 //当p_num > 1时循环继续
    -> 		SET p_result = p_result * p_num ;		 //计算乘积
    -> 		SET p_num = p_num -1;  					//p_num值减1
    -> 	END WHILE;
    -> END
    -> //
Query OK, 0 rows affected (0.04 sec)

mysql> DELIMITER ;
mysql> CALL sp_test(5,@result);                //计算5的阶乘
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @result;                     
+---------+
| @result |
+---------+
|     120 |
+---------+
1 row in set (0.00 sec)

mysql> CALL sp_test(0,@result);             //看是否为初始值1,因为p_num为0时，未执行循环
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @result;
+---------+
| @result |
+---------+
|       1 |
+---------+
1 row in set (0.00 sec)
```

### REPEAT

```SQL
mysql> DELIMITER //
mysql> CREATE PROCEDURE sp_test2(IN p_num INT,OUT p_result INT)
    -> BEGIN
    -> 	SET p_result = 1;
    -> 	REPEAT
    -> 		SET p_result = p_result * p_num;
    -> 		SET p_num = p_num - 1;
    -> 	UNTIL p_num <=1 END REPEAT;
    -> END
    -> //
Query OK, 0 rows affected (0.00 sec)

mysql> DELIMITER ;

mysql> CALL sp_test2(5,@result);
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @result;
+---------+
| @result |
+---------+
|     120 |
+---------+
1 row in set (0.00 sec)

mysql> CALL sp_test2(0,@result);
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @result;
+---------+
| @result |
+---------+
|       0 |
+---------+
1 row in set (0.00 sec)
```



这时，就出现了一个非常有趣的现象，因为当【p_num】为0时，结果变成了0，这是为什么呢？原来，对于REPEAT来说，无论UNTIL的判断结果是TRUE还是FALSE，循环体都会被执行一次。

所以，在实际开发中，究竟是使用WHILE还是REPEAT就需要好好的思考和权衡了。



