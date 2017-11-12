# MySQL的select语法

## 介绍

SQL中最常用的当属`select`命令了，它被用于从一张或者多张表中获取数据，简单的使用例子例如是`select * from tab_name`，可以将一张表中的所有数据取出来；但又由于支持条件过滤、分组、排序、合并、嵌套查询等等特性，有些应用场景中的SQL可以说是非常复杂，下面我们就来整理一下SQL支持的语法都有哪些。

`select`完整的语法结构如下所示，可以说是非常庞大的。

```
SELECT
    [ALL | DISTINCT | DISTINCTROW ]
      [HIGH_PRIORITY]
      [STRAIGHT_JOIN]
      [SQL_SMALL_RESULT] [SQL_BIG_RESULT] [SQL_BUFFER_RESULT]
      [SQL_CACHE | SQL_NO_CACHE] [SQL_CALC_FOUND_ROWS]
    select_expr [, select_expr ...]
    [

     FROM table_references
      [PARTITION partition_list]
    [WHERE where_condition]
    [GROUP BY {col_name | expr | position}
      [ASC | DESC], ... [WITH ROLLUP]]
    [HAVING where_condition]
    [ORDER BY {col_name | expr | position}
      [ASC | DESC], ...]
    [LIMIT {[offset,] row_count | row_count OFFSET offset}]
    [PROCEDURE procedure_name(argument_list)]
    [INTO OUTFILE 'file_name'
        [CHARACTER SET charset_name]
        export_options
      | INTO DUMPFILE 'file_name'
      | INTO var_name [, var_name]]
    [FOR UPDATE | LOCK IN SHARE MODE]
    
    ]
```

如下是我总结的select语法流程图：

![select语法流程图](http://mdimg.fabuler.cn/1711/p001.png)

## 知识点

### `SELECT select_expr [, select_expr ...]`

其中的中括号表示可选的意思，所以只需要`SELECT select_expr`这两部分，`select`就可以正常工作啦，这两部分也是必选的。举个例子：

- 查询数字 `select 1` 结果：`1`
- 查询字符串 `select 'a'` 结果：`a`
- 查询计算结果 `select 1+1` 结果：`2`
- 查询当前时间 `select now()` 结果：`2017-11-11 15:23:11`

当然我们也可以查询多个`select_expr`，这时的语法结构是`SELECT select_expr [, select_expr ...]`。

### `FROM table_references`

大部分情况下，我们需要指定数据源，即表名，`FROM table_references`，注意这里是`table_references`，而不是`table_name`，因为`table_references`可以代指多张表的组合。组合的方法查考本文后边提到的`JOIN`语法。我们暂时只考虑一张表。

假设我们有一张表，表名为`student`，字段有`id, name, age, create_time`，

- 查询`id, name`两列，`select id, name from student`
- 查询所有的列，`select id, name, age, create from student`，更简单的，`select * from student`，其中`*`指代`student`表中的所有列
- 给每个学生的数字id加上20110000，代表学号，`select id+20110000 as student_number, name from student`

上述例子中用到了`as`关键词，代表别称，可以给`select_expr`指定别名，且此时`as`是可以省略的（但是非常不建议省略，`select c1 c2 from t1`等价于`select c1 as c2 from t1`，而不是`select c1, c2 from t1`，查询时带上`as`是好习惯）。

注意：用`as`指定的别名，不能用在`where`中，因为`where`先于`select_expr`执行。

### 缺省数据库和缺省表名

如果使用`use database_name;`指定了缺省的数据库，那么就可以缺省使用表名了；否则，需要显示指定数据的数据库`select * from database_name.table_name;`。

如果一条查询语句只涉及到一张表，`select id,name from student;`是可以正常执行的；但如果有两张表，其中有相同的字段名，例如有两个学生表`t1`和`t2`，那么查询时需要显式指定查询的字段属于哪张表，`select t1.id, t1.name, t2.id, t2.name from t1, t2`，以避免冲突。

### [GROUP BY {col_name | expr | position} [ASC | DESC], ... [WITH ROLLUP]]

`group by`用来对`select xxx from yyy where zzz;`的结果做聚类，`select age from student where id<100 group by age;`，是对id小于100的学生做年龄聚类，返回的结果是没有重复的，这和使用`distinct`关键词是没有区别的，`select distinct age from student where id<100;`。

上述使用`group by`的方法意义不大，一般我们会对每个group做统计，例如统计每个类别中学生的数量，`select age,count(*) as count from student where id<100 group by age;`。

输出的结果，缺省情况下`age`升序排列，即`group by age asc`，也可以使用`desc`是得输出结果按照`age`降序排列，`group by age desc`。（`group by`缺省升序排列这个特性已经废弃了，建议显式说明排序方式）

此外，也可以按照`count`降序排列，`select age,count(*) as count from student where id<100 group by age order by count desc;`

`group by position`的使用方式已经废弃了，不建议使用，已被SQL标准移除。

### [ORDER BY {col_name | expr | position} [ASC | DESC], ...]

`order by`可以对查询结果进行排序，排序指标可以有一个或者多个，例如：

- 按照id降序排列 `select * from student order by id desc;`
- 按照age降序排列，如果age相同，按照id升序排列 `select * from student order by age desc, id asc;`

使用`group by`和`order by`做排序时，如果被排序的值较长，只会根据值的前若干部分进行排序，这个长度由系统变量`max_sort_length`确定，例如这个值是1024（bytes）。

`order by position`的使用方式已经废弃了，不建议使用，已被SQL标准移除。

### [HAVING where_condition]

`having`用于对查询结果的过滤，和`where`类似。

- `select * from student where id<=10;`
- `select * from student having id<=10;`

以上两个sql语句都是可以正常执行的，但是不建议使用`having`替代`where`。

`having`几乎是在MySQL服务端将结果发送给客户端之前执行的，非常靠后（`limit`在`having`之后），且不会使用优化手段，执行速度比`where`要慢很多。就用上边这里例子来说，`having id<=10`会先全表遍历，然后再返回前10行，而`where id<=10`，通过使用主键等索引，只需要扫描10行就能取到最终的结果。

SQL标准要求`having`只能使用`group by`中的字段。但是MySQL支持使用`select`的字段列表。

### [LIMIT {[offset,] row\_count | row\_count OFFSET offset}]

`limit`用于限制返回结果的数量。`offset`的起始值是0。

- 查询前5个学生的数据 `select * from student limit 5;`
- 查询第6到10个学生的数据 `select * from student limit 5,10;`
- 查询第11个之后的所有的学生的数据，用了一个比较大的整数 `select * from student limit 10,18446744073709551615;`

为了兼容PostgreSQL，MySQL也支持`limit row_count offset offset_count`这样的语法。

### PROCEDURE

`PROCEDURE`在MySQL 5.7.18被设置为废弃状态，并会在MySQL 8.0中移除。

指定了一个procedure，用于处理结果集中的数据，参考[https://dev.mysql.com/doc/refman/5.7/en/procedure-analyse.html](https://dev.mysql.com/doc/refman/5.7/en/procedure-analyse.html)。

### 嵌套查询

举个例子，`select * from (select 1,2,3) as t1`

其中子查询`select 1,2,3`生成的结果表，又称为导出表。该结果表必须设定一个别称，用作表名，即`as t1`。

### JOIN

在MySQL中`JOIN`、`CROSS JOIN`和`INNER JOIN`是等价的；但在标准SQL中，是不等价的，`INNER JOIN`会和`ON`搭配使用，而`CROSS JOIN`不是。

`INNER JOIN`和`,`是等价的，都会生成两张表的笛卡尔积，即第一张表中的每一行会和第二张表中的每一行组合生成新行。

`,`的优先级低于`INNER JOIN`，所以这样的语句是错误的`select * from t1, t1 as t2 join t1 as t3 on t1.c1=t3.c1;`，需要都使用`select * from t1 join t1 as t2 join t1 as t3 on t1.c1=t3.c1;`

`USING(column_list)`中`column_list`指代的字段必须同时存在于两张表中，例如`a LEFT JOIN b USING (c1, c2, c3)`

`NATURAL [LEFT] JOIN`等价于`t1 INNER JOIN t2 USING(all_same_column_list)`，其中`all_same_column_list`代表`t1`和`t2`表中所有名称相同的字段

`RIGHT JOIN`和`LEFT JOIN`，建议使用`LEFT JOIN`

### UNION

`UNION`的语法

```
SELECT ...
UNION [ALL | DISTINCT] SELECT ...
[UNION [ALL | DISTINCT] SELECT ...]
```

`UNION`用于合并多个`select`查询结果。

第一个`select`结果中的列名称会作为总结果的列名称。

多个`select`结果中对应列的类型应该相同；否则总结果会统筹考虑所有`select`结果的值。

默认情况下，如果两个`select`的结果有相同的行，只会保留相同行中的一个。如果不希望这样的事发生，需要使用关键词`ALL`，即`select ... union all select ...;`。

如果单个`select`中使用了`order by`或者`limit`，需要用括号括起来：

```
(SELECT a FROM t1 WHERE a=10 AND B=1 ORDER BY a LIMIT 10)
UNION
(SELECT a FROM t2 WHERE a=11 AND B=2 ORDER BY a LIMIT 10);
```

可以对合并后的结果做`order by`和`limit`操作，例如：

```
(SELECT a FROM t1 WHERE a=10 AND B=1)
UNION
(SELECT a FROM t2 WHERE a=11 AND B=2)
ORDER BY a LIMIT 10;
```

可以使用一个固定值来连接两个表，标识数据是属于哪张表的

```
(SELECT 1 AS sort_col, col1a, col1b, ... FROM t1)
UNION
(SELECT 2, col2a, col2b, ... FROM t2) ORDER BY sort_col;
```

## 附录

`INNER JOIN` `LEFT JOIN` `RIGHT JOIN`比较

表t1，表t2如下所示

```
mysql> select * from t1;
+------+------+
| c1   | c2   |
+------+------+
|    1 |    1 |
|    2 |    3 |
|    3 |    5 |
+------+------+

mysql> select * from t2;
+------+------+
| c1   | c2   |
+------+------+
|    1 |    2 |
|    2 |    4 |
|    4 |    6 |
+------+------+
```

```
mysql> select * from t1 inner join t2;
+------+------+------+------+
| c1   | c2   | c1   | c2   |
+------+------+------+------+
|    1 |    1 |    1 |    2 |
|    2 |    3 |    1 |    2 |
|    3 |    5 |    1 |    2 |
|    1 |    1 |    2 |    4 |
|    2 |    3 |    2 |    4 |
|    3 |    5 |    2 |    4 |
|    1 |    1 |    4 |    6 |
|    2 |    3 |    4 |    6 |
|    3 |    5 |    4 |    6 |
+------+------+------+------+
```

```
mysql> select * from t1 inner join t2 on t1.c1=t2.c1;
+------+------+------+------+
| c1   | c2   | c1   | c2   |
+------+------+------+------+
|    1 |    1 |    1 |    2 |
|    2 |    3 |    2 |    4 |
+------+------+------+------+

mysql> select * from t1 left join t2 on t1.c1=t2.c1;
+------+------+------+------+
| c1   | c2   | c1   | c2   |
+------+------+------+------+
|    1 |    1 |    1 |    2 |
|    2 |    3 |    2 |    4 |
|    3 |    5 | NULL | NULL |
+------+------+------+------+

mysql> select * from t1 right join t2 on t1.c1=t2.c1;
+------+------+------+------+
| c1   | c2   | c1   | c2   |
+------+------+------+------+
|    1 |    1 |    1 |    2 |
|    2 |    3 |    2 |    4 |
| NULL | NULL |    4 |    6 |
+------+------+------+------+
```

`c1`是`t1`和`t2`的共有字段

```
mysql> select * from t1 inner join t2 using (c1);
+------+------+------+
| c1   | c2   | c2   |
+------+------+------+
|    1 |    1 |    2 |
|    2 |    3 |    4 |
+------+------+------+

mysql> select * from t1 left join t2 using (c1);
+------+------+------+
| c1   | c2   | c2   |
+------+------+------+
|    1 |    1 |    2 |
|    2 |    3 |    4 |
|    3 |    5 | NULL |
+------+------+------+

mysql> select * from t1 right join t2 using (c1);
+------+------+------+
| c1   | c2   | c2   |
+------+------+------+
|    1 |    2 |    1 |
|    2 |    4 |    3 |
|    4 |    6 | NULL |
+------+------+------+
```

## 参考

- [https://dev.mysql.com/doc/refman/5.7/en/select.html](https://dev.mysql.com/doc/refman/5.7/en/select.html)
- [https://dev.mysql.com/doc/refman/5.7/en/join.html](https://dev.mysql.com/doc/refman/5.7/en/join.html)
- [https://dev.mysql.com/doc/refman/5.7/en/union.html](https://dev.mysql.com/doc/refman/5.7/en/union.html)