[TOC]

#  史上最简单MySQL教程详解（进阶篇）之索引及其失效场合总结

## 什么是索引及其作用

在大型数据库中，一张表通常要容纳几十万甚至是上百万的数据，这些表一旦涉及到表连接等复杂操作后，当用户检索这么大的数据量时，如果要将数据库中所有的数据都与想要查询的数据进行比较的话，是十分慢的。所以为了有效的提高查询速率，避免对于数据库进行全盘扫面，我们就需要使用到：索引。

MySQL官方对索引的定义为：索引（Index）是帮助MySQL高效获取数据的数据结构，它的存在形式是文件。通俗的说：索引更像是我们在图书馆借书的过程中使用的图书目录表。它帮我们将图书根据书名或者是图书的类别进行了分类，让我们可以更加迅速的找到我们所需要的图书，避免了大范围的寻找。

### 索引的种类

索引是在MYSQL的存储引擎层中实现的，而不是在服务层实现的。所以每种存储引擎的索引都不一定完全相同，也不是所有的存储引擎都支持所有的索引类型。MYSQL目前提供了一下4种索引。

- B-Tree 索引：最常见的索引类型，大部分引擎都支持B树索引。
- HASH 索引：只有Memory引擎支持，使用场景简单。
- R-Tree 索引(空间索引)：空间索引是MyISAM的一种特殊索引类型，主要用于地理空间数据类型。
- Full-text (全文索引)：全文索引也是MyISAM的一种特殊索引类型，主要用于全文索引，InnoDB从MYSQL5.6版本提供对全文索引的支持。

### 各存储引擎对于索引的支持

| 索引           | MyISAM引擎 | InnoDB引擎 | Memory引擎 |
| -------------- | ---------- | ---------- | ---------- |
| B-Tree 索引    | 支持       | 支持       | 支持       |
| HASH 索引      | 不支持     | 不支持     | 支持       |
| R-Tree 索引    | 支持       | 不支持     | 不支持     |
| Full-text 索引 | 不支持     | 暂不支持   | 不支持     |

### 简单介绍索引的实现

大多数数据库都使用了B树(Balance Tree,平衡树)的结构来保存索引。B树就是如下图所示的一样，枝叶扩散开来的树状结构。

![这里写图片描述](https://img-blog.csdn.net/20180828130612258?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3ODg4MDMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70) 

B-TREE 每个节点都是一个二元数组: 一个是保存着关键字key,另一个则是指针 pointer。指针则是指向数据所存储的位置。节点就由这些数据相互关联起来而组成。

最上层的节点被称为根节点，最下面的节点被称为叶子节点。两者之间的节点被称为中间节点。B树中最显著的一个特点就是：根节点到各个叶子节点的距离都相等。也就意味着，检索任何值都经过相同数目的节点，能提高检索效率。

查找的原理：B-树的搜索，从根结点开始，对结点内的关键字（有序）序列进行二分查找，如果命中则结束，否则进入查询关键字所属范围的子结点；重复，直到所对应的儿子指针为空，或已经是叶子结点；因此，B-Tree的查找过程是一个顺指针查找结点和在结点的关键字中进行查找的交叉进行的过程。例如：根据上图，我们需要查找[K],首先我们去根节点检索，发现它位于左节点[EH],又去[EH]中查找发现它位于中间节点，就查找到了[K]这个数据。避免了再去检索右边部分的节点，有效的提高了检索的效率。



## 索引的设置与分析

我们这里主要讲解的是使用最多的B-tree索引。其他类型的索引，小伙伴们可以自行学习。

### 普通索引

这是最基本的索引类型，而且它没有唯一性之类的限制。普通索引可以通过以下几种方式创建：                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     （1）创建索引:使用【CREATE INDEX】命令，语法如下： 

```sql
CREATE INDEX 索引名 ON 表名(列名);
```

（2）修改表:使用【ALTER TABLE】命令，语法如下：

```sql
ALTER TABLE 表名ADD INDEX 索引名 (列名);
```

（3）创建表时指定索引：使用【CREATE TABLE 】命令，语法如下：

```sql
CREATE TABLE 表名 ( [...], INDEX 索引名 (列名) );
```

### 唯一索引(Unique Index )

表示唯一的，不允许重复的索引，如果该字段信息保证不会重复例如身份证号用作索引时，可设置为unique。UNIQUE索引创建语法和普通索引大体上无异，只是增加了关键词：UNIQUE。
（1）创建索引：

```sql
CREATE UNIQUE INDEX 索引名 ON 表名(列名);
```

（2）修改表：

```sql
ALTER TABLE 表名ADD UNIQUE 索引名 (列名);
```

（3）创建表时指定索引：

```sql
CREATE TABLE 表名( [...], UNIQUE 索引名 (列名) );
```

### 丛生索引（Clustered Index）

丛生索引是伴随着主键的定义而产生的一种特别唯一性索引，相当于聚合索引，是查找最快的索引。因为每张表只能有一个主键，所以每张表只能有一个丛生索引。但是需要注意注意的是，丛生索引无法通过【CREATE INDEX】语句创建，而是通过定义主键来实现：
（1）主键一般在创建表的时候指定，语法为：

```SQL
CREATE TABLE 表名( [...], PRIMARY KEY (列的列名) )；
```


（2）修改表的方式加入主键：

```SQL
ALTER TABLE 表名ADD PRIMARY KEY (列的列名)；
```

#### 丛生索引与其他索引的区别：

+ 通常索引在叶子节点中保存指向实际数据的指针，而丛生索引的叶子节点保存的是实际数据。
+ 不需要为保存索引而使用专门的硬盘空间，节约了资源
+ 不需要检索索引后再访问数据表，提高了检索的效率。
+ 创建丛生索引时需要对表中的数据进行排序，在进行数据插入、更新、删除时比一般的索引需要耗费更多的时间。

### 查看索引信息

我们使用【SHOW INDEX】命令来查看表中所有创建完成的索引信息，语法如下：

```sql
SHOW INDEX FROM 表名；
```

例如：我们对之前已经创建好的[【student】表中的](https://blog.csdn.net/m0_37888031/article/details/80632268)【name】字段创建索引，并用【SHOW INDEX】语句查看相关信息：

```sql
mysql> CREATE INDEX index_name ON student(name);
Query OK, 5 rows affected (0.10 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SHOW INDEX FROM student \G
*************************** 1. row ***************************
        Table: student
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 1
  Column_name: studentId
    Collation: A
  Cardinality: 5
     Sub_part: NULL
       Packed: NULL
         Null:
   Index_type: BTREE
      Comment:
Index_comment:
*************************** 2. row ***************************
        Table: student
   Non_unique: 1
     Key_name: collegeId
 Seq_in_index: 1
  Column_name: collegeId
    Collation: A
  Cardinality: NULL
     Sub_part: NULL
       Packed: NULL
         Null: YES
   Index_type: BTREE
      Comment:
Index_comment:
*************************** 3. row ***************************
        Table: student
   Non_unique: 1
     Key_name: index_name
 Seq_in_index: 1
  Column_name: name
    Collation: A
  Cardinality: NULL
     Sub_part: NULL
       Packed: NULL
         Null:
   Index_type: BTREE
      Comment:
Index_comment:
3 rows in set (0.00 sec)
```

#### 【SHOW INDEX】执行结果说明：

| 参数名        | 说明                            |
| ------------- | ------------------------------- |
| Table         | 表名                            |
| Non_unique    | 是否允许重复（1:允许，0：禁止） |
| Key_name      | 索引名                          |
| Seq_in_index  | 索引内的域序号（从1开始）       |
| Column_name   | 域名                            |
| Collation     | 排序（A:升序，Null:不排序）     |
| Cardinality   | 索引内的非重复值的数目          |
| Sub_part      | 作为索引部分的域内的字符数      |
| Packed        | 关键字的压缩方式                |
| Null          | 是否允许为Null                  |
| Index_type    | 索引的类型                      |
| Comment       | 备注                            |
| Index_comment | 索引备注                        |

#### 注意事项：

+ 因为目前MySQL不支持函数索引，但是能对列的前面某一部分进行索引，例如标题title字段，可以只取title的前10个字符进行索引，这个特性可以大大缩小索引文件的大小，但前缀索引也有缺点，在排序Order By和分组Group By 操作的时候无法使用。用户在设计表结构的时候也可以对文本列根据此特性进行灵活设计，参数【Sub_part】正是针对这种情况产生。语法如下：

  ```sql
  CREATE INDEX 索引名 ON 表名(字段名(所取字符长度))
  ```

+ 加入我们在创建索引的时候，同时选择了多个域，语法如下：

  ```sql
  CREATE INDEX 索引名 ON 表名(列名1，列名2)
  ```

  这时，我们创建的索引称为复合索引。这是我们再通过【SHOW INDEX】命令来查看索引信息，就会发现【Key_name】一样，但是【Seq_in_index】显示的就是列的顺序了。

### 删除索引

如果需要删除索引，使用的是【DROP INDEX】命令，语法如下：

```sql
DROP INDEX 索引名 ON 表名；
```

## 分析索引优劣

当我们学会如何创建、修改和删除索引以后，我们就可以尝试着自己去分析索引的使用情况以及索引的好坏。我们使用【EXPLAIN】来确认索引的使用情况。（EXPLAIN是大多数数据库支持的命令，但是使用方法与返回的信息随着数据库的不同可能会有所不同），其的具体语法如下：

```sql
EXPLAIN 需要分析的SELECT语句
```

例如：我们删除之前在【student】表上创建的索引后，执行Select语句，结果如下：

```sql
mysql> EXPLAIN SELECT * FROM student WHERE phone= "135" \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: student
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 5
     filtered: 20.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

### 【EXPLAIN】命令执行返回参数说明：

| 参数名        | 说明                                           |
| ------------- | ---------------------------------------------- |
| id            | Select命令的顺序号（通常为1，子查询时从2开始） |
#### 参数【select_type】的返回值说明

| 值                 | 说明                                        |
| ------------------ | ------------------------------------------- |
| SIMPLE             | 单纯的SELECT命令                            |
| PRIMARY            | 最外层的SELECT命令                          |
| UNION              | 由UNION语句连接的Select命令                 |
| DEPENDENT UNION    | 由UNION语句连接的Select命令（依赖外部查询） |
| SUBQUERY           | 子查询中的SELECT语句                        |
| DEPENDENT SUBQUERY | 子查询中的SELECT语句（依赖外部查询）        |
| DERIVED            | 派生表（FROM 语句的子查询）                 |

#### 参数【type】的返回值说明

| 值          | 说明                                                     |
| ----------- | -------------------------------------------------------- |
| system      | 只存在一条数据的表（系统表）                             |
| const       | 拥有PRIMARY KEY/UNION 制约的索引（唯一）                 |
| eq_ref      | 连接时由PRIMARY KEY/UNION 列进行的等值检索               |
| f           | 非UNION列进行的等值检索                                  |
| ref_or_null | ref中加入了【~OR 列名 IS NULL】的检索                    |
| range       | 使用索引检索一定范围的数据）（=、>、<、IS Null等运算符） |
| index       | 全索引扫描                                               |
| ALL         | 全表扫描                                                 |

#### 参数【EXTRA】的返回值说明：



| 参数                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| Using where             | 列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候。 |
| Using temporary         | 表示MySQL需要使用临时表来存储结果集，常见于排序和分组查询    |
| Using join buffer       | 说明获取连接条件时没有使用索引，并且需要连接缓冲区来存储中间结果，可能需要添加索引来改进能。 |
| Impossible where        | 说明where语句会导致没有符合条件的数据。                      |
| Select tables optimized | 说明仅通过使用索引，优化器可能仅从聚合函数结果中返回一行     |
| Using filesort          | 无法利用索引完成排序                                         |

我们在经历过上面的介绍以后，再去查看我们之前执行的结果，我们可以看到遍历的次数和实际表中的数据（数据量也是5）是相等的，显然检索的效率不是很高。如果在大数据量下，这样的遍历次数对于查询速度的影响几乎是灾难的，所以，我们再重新加入所以后再试一次:

```sql
mysql> CREATE INDEX index_phone ON student(phone);
Query OK, 5 rows affected (0.10 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> EXPLAIN SELECT * FROM student WHERE phone= "135" \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: student
   partitions: NULL
         type: ref
possible_keys: index_phone
          key: index_phone
      key_len: 767
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)
```

我们发现，遍历的次数已经降到了只有一次，而且filtered也达到了100%，效率得到了很大程度上的提升。同样的，如果我们使用索引后，查询速度未得到改善，我们则需要反思我们的所以设置是否合理，这也就是最简单的分析索引优劣的方法了。



## 索引失效场合总结

### 进行后方一致/部分一致检索的场合

当我们在使用【LIKE】关键词进行模糊检索时，只有在前方一致的检索时能使用上索引，而后方一致或者部分一致是不能使用的。例如下面的两句查询语句都是无效的：

```SQL
SELECT * FROM student WHERE name LIKE "%张%"
SELECT * FROM student WHERE name LIKE "%张"
```

### 使用了IS NOT NULL、<>比较运算符的场合

当我们使用了IS NOT NULL、<>比较运算符时，也是不能使用索引的。例如下面的两句查询语句都是无效的：

```sql
SELECT * FROM student WHERE name IS NOT NULL 
SELECT * FROM student WHERE name <> "张三"
```

### 对列使用了运算/函数的场合

当我们在查询语句中使用了运算或者函数时，索引也是无效的。对于MySQL所具有的函数和运算符可参考这篇博文：[史上最简单MySQL教程详解（基础篇）之运算符和常用数据库函数](https://blog.csdn.net/m0_37888031/article/details/80551151)

### 复合索引的第一列没有包含在WHERE条件语句中的场合

例如我们在【student】表创建了基于【name】和【phone】的复合索引，且【name】作为第一列，那么下面的两句查询语句都是无效的：

```
Select * FROM student WHERE phone = "135"；
Select * FROM student WHERE name = "张三" OR phone = "139"；
```



## 参考文献

+ 人云思云，[MYSQL-索引](https://segmentfault.com/a/1190000003072424)

+ 美团技术团队，[MySQL索引原理及慢查询优化](https://blog.csdn.net/m0_37888031/article/details/80551151)

+ 《MySQL高效编程》

**转载说明： 支持转载，但请保留原作者，原文链接，微信公众号二维码 ** 

<center>![这里写图片描述](https://img-blog.csdn.net/20180816004046416?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3ODg4MDMx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70) **扫码关注作者个人技术公众号,不定期将有学习资源分享** </center>  

