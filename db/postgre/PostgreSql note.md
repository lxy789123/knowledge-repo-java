[TOC]

# 一、认识postgreSQL
## 1.1 postgre优势
- 支持自定义类型
- 支持重载运算符（+, -, *,  /)
- 支持数组

## 1.2 pgsql 数据库对象

***database***: 每个pg可以包含多个独立database

***schema***: 从属于database。新建database时，会创建默认schema（public schema）。如果创建table时，没有指定schema，会<u>默认放入public schema</u>，不推荐这样做。

***table***: 从属于schema。pgsql支持表继承，即父表子表；pgsql会在创建一张表时，默认为此表创建一种对应的自定义数据类型。

***view***: 虚拟表，一般是只读的。但在pgsql中，如果view是基于单实体表构建的，则支持view数据修改。

***扩展包***：类似于插件，可以包含函数、数据类型、数据类型转换器、用户自定义索引、表以及属性变量。

***函数***: 其他数据库把对数据库进行增删改(DML, Data Manipulation Language)操作的函数称为”存储过程( stored procedure)“，把不进行增删改的

# 二、数据类型

## 2.1 数值类型

|     类型名称     |   存储长度   |       描述       |               范围                |
| :--------------: | :----------: | :--------------: | :-------------------------------: |
|  smallint(int2)  | 2字节(16bit) |    小范围整型    |           -2^15~+2^15-1           |
|  integer(int4)   | 4字节(32bit) |       整型       |          -2^31 ~ +2^31-1          |
|   bigint(int8)   | 8字节(64big) |    大范围整型    |          -2^63 ~ +2^63-1          |
|     decimal      |     可变     |   用户指定精度   | 小数点前131072位；小数点后16383位 |
|     numberic     |     可变     |   用户指定精度   | 小数点前131072位；小数点后16383位 |
|       real       |    4字节     |   变长，不精确   |           6位十进制精度           |
| double precision |    8字节     |   变长，不精确   |          15位十进制精度           |
|   smallserial    |    2字节     | smallint自增序列 |             1~2^15-1              |
|      serial      |    4字节     | integer自增序列  |             1~2^31-1              |
|    bigserial     |    8字节     |  bigint自增序列  |             1~2^63-1              |

- decimal 和 numeric 是等效的。声明时按照如下格式

  ```sql
  NUMERIC(precision, scale)
  ```

  其中，recision是numeric数字里的全部位数，scale是指小数部分的位数。例如18.222的precision是5，scale是3；precision必须是正整数，scale可以是0或整数。性能弱于int。

- real和double precision都是浮点型。

- serial等严格意义上不是数据类型。

## 2.2 字符类型

### 2.2.1 字符类型列表

|           字符类型名称           |                  描述                  |
| :------------------------------: | :------------------------------------: |
| character varying(n), varchar(n) |         变长，字符最大数有限制         |
|      character(n), char(n)       | 定长，字符数没达到最大值则使用空白填充 |
|               text               |            变长，无长度限制            |

- varchar(n)在长度不足n时，只存储实际大小。长度不可超过n，否则报错。如果不声明长度，则可以存储任意长度。
- char(n)在长度不足n时，用空白填充。长度不可超过n，否则报错。如果不声明长度，则默认为char(1)。
- text类型和不声明长度的varchar没有差别。

### 2.2.2 字符类型函数

| 函数         | 功能                     | 示例                                                         |
| ------------ | ------------------------ | ------------------------------------------------------------ |
| char_length  | 计算字符串中的字符数     | char_length('abc')<br />结果为4                              |
| octet_length | 计算字符串占用字节数     | octet_length('abcd')<br />4                                  |
| position     | 指定字符在字符串中的位置 | position('a' in 'abcd')<br />1                               |
| substring    | 提取字符串中的子串       | substring('francs' from 3 for 4)<br />ancs - 从第三个字符开始的四个 |
| split_part   | 用分隔符拆分字符串       | split_part(string text, delimiter text, field int)<br />split_part('abc@def1@nb', '@', 2)<br />def1 |

## 2.3 时间/日期类型

| 字符类型名称  | 存储长度 | 描述                       |
| ------------- | -------- | -------------------------- |
| timestamp     | 8字节    | 包括日期和时间，不带时区   |
| timestampz    | 8字节    | 包括日期和时间，带时区     |
| date          | 4字节    | 日期，但不包含一天中的时间 |
| time[(p)] [without time zone] | 8字节 | 一天中的时间，不包括日期，不带时区 |
| time[(p)] with time zone | 12字节 | 一天中的时间，不包含日期，带时区 |
| interval [fields] [(p)] | 16字节 | 时间间隔 |


例子：
```sql
mydb=> SELECT now();
now
-------------------------------
2017-07-29 09:44:25.493425+08
(1 row)
```

类型转换：

- timestamp

```sql
mydb=> SELECT now()::timestamp without time zone;
now
----------------------------
2017-07-29 09:44:55.804403
(1 row)
```

- date
```sql
mydb=> SELECT now()::date;
now
------------
2017-07-29
(1 row)
```

- time with time zone:

```sql
mydb=> SELECT now()::time with time zone;
now
-------------------
09:45:57.13139+08
(1 row)
```

- interval: 间隔单位可以是hour/day/mouth/year
```sql
mydb=> SELECT now(), now()+interval'1 day';
now | ?column?
----------------------------------+-------------------------------
2017-07-29 09:47:26.026418+08 | 2017-07-30 09:47:26.026418+08
(1 row)

```

时间类型中的(p)是只时间精度，具体指秒后面小数点保留的位数；没有声明则默认为6，如：

```sql
mydb=> SELECT now(), now()::timestamp(0);
now | now
----------------------------------+---------------------
2017-07-29 09:59:42.688445+08 | 2017-07-29 09:59:43
(1 row)

```

## 2.4 布尔类型

| 字符类型名称 | 存储长度 | 描述                                                         |
| ------------ | -------- | ------------------------------------------------------------ |
| boolean      | 1字节    | TRUE可为：TRUE/t/y/yes/on/1<br />FALSE为：FALSE/f/false/n/no/off/0 |

## 2.5 网络地址类型

| 字符类型名称 | 存储长度  | 描述                                                         |
| ------------ | --------- | ------------------------------------------------------------ |
| cidr         | 7或19字节 | IPv4和IPv6网络<br />会检查格式是否正确:<br />address/y<br />address为网络地址，y为子网掩码 |
| inet         | 7或19字节 | IPv4和IPv6网络<br />会检查格式是否正确:<br />address/y<br />address为网络地址，y为子网掩码 |
| macaddr      | 6字节     | MAC地址                                                      |
| macaddr8     | 8字节     | MAC地址(EUI-64格式)                                          |

cidr和inet区别：

- cidr类型的输出默认带子网掩码信息，inet不一定。
- cidr对ip和子网掩码合法性进行检查，inet不会。

## 2.6 数组类型

pgsql支持一维和多维数组。

### 2.6.1 数组类型定义

```sql
create table test_array1 (
	id	integer,
    array_i	integer[],
    array_t	text[]
);
```

以上带"[]"都是一维数组

### 2.6.2 数组类型值输入

数组类型插入的两种方式：

- 花括号
```sql
mydb=> SELECT '{1,2,3}';
	?column?
------------
	{1,2,3}
(1 row)
```

往表中插入记录：

```sql
mydb=> INSERT INTO test_array1(id, array_i, array_t)
VALUES (1, '{1,2,3}','{"a","b","c"}')
INSERT 0 1
```

- array关键字

```sql
mydb=> INSERT INTO test_array1(id, array_i, array_t)
VALUES (2, array[4,5,6], array['d','e','f']);
INSERT 0 1
```

两次插入之后的结果：

```sql
mydb=> SELECT * FROM test_array1;
id | array_i | array_t
-------+---------+---------
1 | {1,2,3} | {a,b,c}
2 | {4,5,6} | {d,e,f}
(2 rows)
```

### 2.6.3 查询数组元素

- 查询所有元素值：

```sql
mydb=> SELECT array_i FROM test_array1 WHERE id=1;
array_i
---------
{1,2,3}
(1 row)
```

- 查询数组某个元素：

```sql
mydb=> SELECT array_i[1],array_t[3] FROM test_array1 WHERE id=1;
array_i | array_t
------------+---------
1 | c
(1 row)
```

### 2.6.4 数组元素的追加、删除、更新

- 追加：

```sql
mydb=> SELECT array_append(array[1,2,3],4);
array_append
--------------
{1,2,3,4}
(1 row)
```

或者：

```sql
mydb=> SELECT array[1,2,3] || 4;
?column?
-----------
{1,2,3,4}
(1 row)
```

- 删除：

```sql
mydb=> SELECT array[1,2,2,3],array_remove(array[1,2,2,3],2);
array | array_remove
------------+--------------
{1,2,2,3} | {1,3}
(1 row)
```

- 修改：

```sql
mydb=> UPDATE test_array1 SET array_i[3]=4 WHERE id=1 ;
UPDATE 1
```

- 更新整个数组：

```sql
mydb=> UPDATE test_array1 SET array_i=array[7,8,9] WHERE id=1;
UPDATE 1
```

## 2.7 范围类型

| 类型      | 说明                        |
| --------- | --------------------------- |
| int4range | integer范围类型             |
| int8range | bigint范围类型              |
| numrage   | numeric范围类型             |
| tsrange   | 不带时区的timestamp范围类型 |
| tstzrange | 带时区的timestamp范围类型   |
| daterange | date范围类型                |

定义范围边界，对比以下两个示例：

```sql
mydb=> SELECT int4range(1,3);
int4range
-----------
[1,3)
(1 row)
```

没有指定边界。

```sql
mydb=> SELECT int4range(1,3,'[]');
int4range
-----------
[1,4)
(1 row)
```

指定上下皆为闭区间。

## 2.8 json/jsonb类型

### 2.8.1 json类型

示例1：

```sql
mydb=> SELECT '{"a":1,"b":2}'::json;
json
---------------
{"a":1,"b":2}
```

示例2：

先创建一张表

```sql
mydb=> CREATE TABLE test_json1 (id serial primary key,name json);
CREATE TABLE
```

插入数据

```sql
mydb=> INSERT INTO test_json1 (name)
VALUES ('{"col1":1,"col2":"francs","col3":"male"}');
INSERT 0 1
mydb=> INSERT INTO test_json1 (name)
VALUES ('{"col1":2,"col2":"fp","col3":"female"}');
INSERT 0 1
```

查询test_json1数据

```sql
mydb=> SELECT * FROM test_json1;
id | name
-------+------------------------------------------
1 | {"col1":1,"col2":"francs","col3":"male"}
2 | {"col1":2,"col2":"fp","col3":"female"}
```

### 2.8.2 查询json数据

通过"->"操作符可以查询json数据的键值：

```sql
mydb=> SELECT name -> 'col2' FROM test_json1 WHERE id=1;
?column?
----------
"francs"
(1 row)
```

如果想用文本格式返回json字段键值可以使用"->>"操作符:

```sql
mydb=> SELECT name ->> 'col2' FROM test_json1 WHERE id=1;
?column?
----------
francs
(1 row)
```

如果用户查询的对象具有层级结构，那么必须按照层级查询才能检索到对应的键值。
例一、查询第一级中的对象

```sql
select a.jon -> 'b'
from (
select '{"a":"aa","b":{"c":"c"}}'
) as a 
-------------------
{"c":"c"}
```
例二、直接查询第二级中对象
```sql
select a.jon -> 'c'
from (
select '{"a":"aa","b":{"c":"c"}}'
) as a 
-------------------
(NULL)
```

例三、先查询第一级，再查询第二级 

```sql
select a.jon -> 'b' -> 'c'
from (
select '{"a":"aa","b":{"c":"c"}}'
) as a 
-------------------
"c"
```

以上三个例子说明：

1. 查询默认从第一层开始
2. 查询不会自动递归，需要手动指定key进行对象检索
3. 查询结果为对象时，会整体返回


### 2.8.3 json与jsonb的差别

- json以文本形式存储，jsonb以二进制形式写入已经解析好的数据，检索时不需要重新解析。因此json写入性能更佳，jsonb检索性能更佳。

- jsonb输出的键的顺序和输入可能不一样，json则始终保持一致。

- jsonb会去掉数据中键值的空格。

- jsonb会删除重复的键，仅保留最后一个name键的值，而json数据会保留重复键值。

# 三、表

## 3.1 基本建表语句

```sql
create table logs (
log_id serial PRIMARY KEY,
user_name varchar(50),
description text,
log_ts timestamp with time zone NOT NULL DEFAULT current_timestamp
);
create index idx_logs_ts on logs using btree(log_ts);
```

pgsql 10 新增了`IDENTITY`关键字支持，它可以将字段定义为自增序列号类型，并且不再借助自动生成的序列号生成器。

```sql
create table logs (
log_id int GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
user_name varchar(50),
description text,
log_ts timestamp with time zone NOT NULL DEFAULT current_timestamp
)
```

`IDENTITY`优点：identity总是与所属表绑定的，其值的递增或者重置都是与表本身一体化管理的，不会受其他对象干扰。

## 3.2 继承表

​	子表会继承父表的所有字段，一旦父表结构发生了变化，子表的结构也会自动跟着变化。当查询父表时，pgsql会自动把子表的记录也取出来。

但继承不包括：

- 主表的主键约束

- 唯一性约束

- 索引

check约束会被继承，同时子表可另外建立自己的check约束。

```sql
create table logs_2011 (PRIMARY KEY (log_id)) INHERITS (logs);
create index idx_logs_2011_log_ts ON logs_2011 using btree(log_ts);
alter table logs_2011
	add constraint chk_y2022 
	check(log_ts >= '2011-1-1'::timestamptz and log_ts < '2012-1-1'::timestamptz)
```

## 3.3 原生分区表支持

&emsp;尽管分区表和继承表从内部实现机制看非常像，只不过ddl语法不同。不过还是有以下的区别。

- 分区表使用声明式的`create table .. partition by range ..`语法，后台回默认创建一个分区表组
- 使用分区表时，如果对主表插入数据，记录会按照分区规则被路由到相应的分区中。但使用表继承时情况就不是这样，你需要直接把数据插入正确的子表中，或者在主表上挂载触发器来把记录 路由到子表中。
- 分区表的所有子分区都必须**具备完全相同的字段结构**，而表继承中的子表完全可以比父表拥有更多字 段。
- 分区表的每个分区都隶属于一个共同的分区表组，这意味着它们**只能有一个父表**，然而表继承功 能中的一张子表可以继承自多个父表。
- **分区表的父表**是一个逻辑对象而非物理实体，因此它上面**不可以定义主键、唯一键或者索引**，但是每个子分区可以定义这些。表继承机制中的情况与此不同：父表和每个子表都可以有主键，而 且主键只需在本表内部唯一即可，并不一定要在所有子表范围内都唯一。
- 与表继承机制中的父表不同，分区表的**父表不能存储自己的记录**。所有针对父表插入的记录都会 被路由到相应的子分区中，如果**没有符合条件的子分区则会报错**。

&emsp;下面是一个创建分区表的示例，首先创建分区表的主表：

```sql
create table logs(
log_id int generated by default as identity,
user_name varchar(50),
description text,
log_ts timestampz not null default current_timestamp
) PARTITION BU RANGE (log_ts);
```

&emsp;创建子分区表：

```sql
create talb logs_2011 partition of logs for values from ('2011-1-1') to ('2012-1-1');
create index idx_logs_2011_log_ts on logs_2011 using btree(log_ts);
alter table logs_2011 add constraint pk_logs_2011 primary key (log_id);
```

&emsp;注意：

- 子分区之间的数据存储范围不能有重叠，否则创建语句会失败。
- 子分区表的主键并不需要在所有子分区表中全局唯一。

&emsp;如果要创建一个只有单边边界的分区表，可以用如下关键字：

```sql
create table logs_gt_2011 partition of logs for values from ('2012-1-1') to (unbounded)
```

`unbounded`表示无边界，即`2012-1-1`时间之后的所有数据都可以存进该表。

# 四、约束机制

## 4.1 外键约束

&emsp;创建外键约束

```sql
alter table facts add constraint fk_fact_1 foreign key (fact_type_id)
referrences lu_fact_types (fact_type_id) on update cascade on delete restrict;
create index fki_fackts_1 on facts (fact_type_id);
```

- 在facts和lu_fact_types表之间定义了一个外键约束。若在facts中插入的fact_type_id值在lu_fact_types中不存在，则禁止插入。
- 若lu_fact_types中的fact_type_id被修改，则facts中的fact_type_id会自动联动修改；若facts中的fact_type_id值还存在，则lu_fact_types表中的fact_type_id不可被删除。
- PostgreSql在建立主键和唯一约束时，会自动为其添加索引。而外键约束不会，需要手动添加。
- 外键约束在表继承不会被继承

## 4.2 唯一性约束

```sql
alter table logs_2011 ADD CONSTRAINT uq Unique (user_name, log_ts);
```

- 建立唯一性约束时将在后台自动创建唯一索引。
- 唯一索引允许输入空值，**唯一约束不允许**。
- 唯一约束在表继承时不会被继承。

## 4.3 check约束 

&emsp;表示给表的一个或者多个字段加上一个条件，表中的每一行记录都必须满足条件。
&emsp;表继承时，check约束也会被子表继承，但**主键、外键、唯一性**三种约束**不会继承**。

```sql
ALTER TABLE logs ADD CONSTRAINT chk CHECK (user_name = lower(user_name))
```

&emsp;上面的语句限制了`user_name`字段只能是小写。

## 4.4 排他性约束
&emsp;传统的唯一性约束只在比较算法中使用“等于”运算符，即保证了指定字段值在本表的任意两行记录中都不相等。排他性约束可以使用更多运算符来进行比较运算，该类约束特别适用于解决有关**时间安排**的问题。<br/>
&emsp;排他性约束一般基于`GiST`类型的索引。<br/>&emsp;下面是一个创建排他性约束的例子:

```sql
CREATE TABLE schedules(
    id serial primary key,
    room int,
    time_slot tstzrange);
ALTER TABLE schedules ADD CONSTRAINT ex_schedules
EXCLUDE USING gist (room WITH =, time_slot WITH &&);
```

​	排他性约束也会自动创建索引。

# 五、索引

## 5.1 PostgreSQL原生支持的索引类型

| 索引类型                | 结构                                     | 适用情况                                                     | 数据类型                                                     |
| ----------------------- | ---------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| B-树                    | B-树                                     | 等值查询和范围查询                                           | 无特定                                                       |
| BRIN                    | block range index，块范围索引            | 针对超大表做索引<br/>比B树索引和其他索引占用空间更小，建立更快<br/>查询速度较慢，不能用于确保唯一主键 |                                                              |
| GiST                    | generalized search tree，通用搜索树      | 括全文检索以及空间数据、科学 数据、非结构化数据和层次化数据的搜索。<br/>不能用于保障字段的唯一性。<br/>但如果把该类索引用于排他性约束就可以实现唯一性保障。 | **几何类型**，支持位置搜索（包含、相交、在上下左右等），按距离排序。<br/>**范围类型**，支持位置搜索（包含、相交、在左右等）<br/>**IP类型**，支持位置搜索（包含、相交、在左右等）<br/>**空间类型**（PostGIS），支持位置搜索（包含、相交、在上下左右等），按距离排序<br/>**标量类型**，支持按距离排序<br/> |
| GIN                     | generalized inverted index，通用逆序索引 | 主要适用于 PostgreSQL 内置的全文搜索引擎以 及二进制 json 数据类型，其底层数据结构类似于**B+树** | 数组、全文检索、TOKEN                                        |
| SP-GiST                 | Space-Partitioned GiST，空间分区GiST索引 | 比GiST在特定领域<br/>数据算法效率更高                        | **几何类型**，支持位置搜索（包含、相交、在上下左右等），按距离排序。<br/>**范围类型**，支持位置搜索（包含、相交、在左右等）<br/>**IP类型**，支持位置搜索（包含、相交、在左右等）<br/> |
| 哈希                    | Hash                                     | 等值查询                                                     | -                                                            |
| 基于B-树的GiST和GIN索引 |                                          | 既能够支 持 GiST 和 GIN 索引特有的运算符，又具有 B-树索引对于“等于”运算符的良好支持 |                                                              |

## 5.2 B-树结构

&emsp;B（Balance）-树，其结构如下图：

![B-树](.\attachment\B-树.png)

- 是一种多路的平衡搜索树。

- 和普通的二叉平衡树不同，B树每个节点可以存储多个数据，而且每个节点不只有两个子节点。

- B树中的每个节点都存放着索引和数据(此“数据”并非数据库表中的“数据”)，搜索可能在非叶子结点结束。

- 一般一棵树的高度在3层左右，3层就可以满足百万级别的数据量。
## 5.3 PostgreSQL的B-tree索引

![B树索引](.\attachment\B树索引.png)

&emsp;B-tree的叶节点存储的key为索引值，value为数据的ctid。ctid指示的是物理行信息，指的是一条记录位于哪个数据块的位移上面。

&emsp;`索引下推`：对于联合索引，例如`(a,b)`，如果查询条件是 `a>? and b = B`, 此时无法使用完整的索引查询，只能用到a层级。数据库引擎通常的做法是利用`(a,b)`查出满足`a>?`条件的叶节点（在pgsql中**ctid**即表示物理地址，在mysql中就是主键索引的值），再通过地址或主键索引查出所有满足`a>?`条件的完整数据，再在这些完整的数据集合上进行`b=B`的筛选。索引下推就是在查出`a>?`的叶节点的时候，就筛选出所有`b=B`条件的叶节点，再通过地址或主键索引查出完整数据。这样做的好处是减少物理查询或者回表的次数，减少内存开销。

# 六、执行计划

`<PostgreSQL修炼之道>`

## 6.1 explain命令

&emsp;语法

``` 
EXPLAIN [(option [, ...])] statement
EXPLAIN [ANALYZE] [VERBOSE] statement
```

&emsp;可选options：

```
ANALYZE [boolean]
VERBOSE [boolean]
COSTS [boolean]
BUFFERS [boolean]
FORMAT {TEXT | XML | JSON | YAML}
```

&emsp;`ANALYSE`表示通过实际执行SQL语句来获得实际的执行计划，因此如果涉及修改表数据的语句，最好使用事务回归已经执行的语句：

````sql
BEGIN;
EXPALIN ANALYSE ...;
ROLLBACK;
```

&emsp;`VERBOSE`显示附加信息，如 计划数中每个节点输出的各个列，如果触发器被处罚，还会输出触发器的名称。默认FALSE。

&emsp;`COSTS`选项显示每个计划节点的启动成本和总成本，以及估计行数和每行宽度。默认TRUE。

&emsp;`BUFFERS`选项显示缓冲区使用的信息。只能与ANALYZE参数一起使用。显示信息包括共享块、本地块和临时块的读写块数信息。他们分别包含表和索引、临时表和临时索引、在排序和物化计划中使用的磁盘块。上层节点显示出来的块数包括其所有子节点使用的块数。默认FALSE。

&emsp;`FORMAT`选项指定输出格式，包含TEXT,XML,JSON或者YAML。默认TEXT。

## 6.2 输出结果的解释

&emsp;以以下语句为例：

```sql
osdba=# explain select * from testtab01;
QUERY PLAN
---------------------------------------------------------------
Seq Scan on testtab01 (cost=0.00..184.00 rows=10000 width=36)
(1 row)
```

&emsp;`Seq Scan on testtab01` 表示顺序扫描表`testtab01`，顺序扫描即全表扫描。

&emsp;`cost=0.00...184.00`中`...`前的数字表示“启动成本”，即返回第一行需要额的cost值；第二个数字表示返回所有数据的成本。

&emsp;`rows=10000`表示会返回10000行。

&emsp;`width=36`表示每行平均宽度为36**字节**。

&emsp;成本`cost`用于描述代价，不同cost值如下表：

| 操作                    | cost值 |
| ----------------------- | ------ |
| 顺序扫描一个数据块      | 1      |
| 随机扫描一个数据块      | 4      |
| 处理一个数据行的cpu代价 | 0.01   |
| 处理一个索引行的cpu代价 | 0.005  |
| 每个操作符的cpu代价     | 0.0025 |

&emsp;一个analyse示例：

```sql
osdba=# explain analyze select * from testtab01;
QUERY PLAN
--------------------------------------------------------------------------------
Seq Scan on testtab01 (cost=0.00..184.00 rows=10000 width=36) (actual
time=0.493..4.320 rows=10000 loops=1)
Total runtime: 5.653 ms
(2 rows)

```

实际时间的单位是**毫秒**。

## 6.3 扫描类型

[pg执行计划]: https://blog.csdn.net/Hehuyi_In/article/details/101782808

&emsp;pgsql支持的扫描类型如下：

- Seq Scan, 顺序扫描。
- Index Scan, 基于索引扫描，但不只是返回索引列的值。
- IndexOnly Scan, 基于索引的扫描，并且只返回索引列的值，简称覆盖索引。
- BitmapIndex Scan, 利用Bitmap结构扫描。
- BitmapHeap Scan, 把BitmapIndex Scan返回的Bitmap结构转换为元祖结构。
- Tid Scan, 用于扫描一个元祖TID数组。
- Subquery Scan, 扫描一个子查询。
- Function Scan, 处理含有函数的扫描。
- TableFunc Scan, 处理含有函数的扫描。
- Values Scan, 用于扫描Values链表的扫描。
- Cte Scan, 用于扫描WITH子句的结果集。
- NamedTuplestore Scan, 用于某些明明的结果集的扫描。
- WordTable Scan, 用于扫描Recursive Union的中间数据。
- Foreign Scan, 用于外键扫描。
- Custom Scan, 用于用户自定义的扫描。

### 6.3.1 全表扫描

&emsp;也称顺序扫描(Seq Scan)，在`Explain`命令输出中用`Seq Scan`表示：

```sql
postgres=> explain(ANALYZE,VERBOSE,BUFFERS) select * from class where st_no=2;
                                               QUERY PLAN
--------------------------------------------------------------------------------------------
 Seq Scan on public.class  (cost=0.00..26.00 rows=1 width=35) (actual time=0.136..0.141 rows=1 loops=1)
   Output: st_no, name
   Filter: (class.st_no = 2)
   Rows Removed by Filter: 1199
   Buffers: shared hit=11
 Planning time: 0.066 ms
 Execution time: 0.160 ms
```

- `Seq Scan on public.class` 表明了这个节点的类型和作用对象，即在class 表上进行了全表扫描
- `(cost=0.00..26.00 rows=1 width=35) `表明了这个节点的代价估计
- `(actual time=0.136..0.141 rows=1 loops=1) `表明了这个节点的真实执行信息，当EXPLAIN 命令中的ANALYZE选项为on时，会输出该项内容
- `Output: st_no, name` 表明了SQL 的输出结果集的各个列，当EXPLAIN 命令中的选项VERBOSE 为on时才会显示
- `Filter: (class.st_no = 2) `表明了Seq Scan节点之上的Filter 操作，即全表扫描时对每行记录进行过滤操作，过滤条件为class.st_no = 2
- `Rows Removed by Filter: 1199 `表明了过滤操作过滤了多少行记录，属于Seq Scan 节点的VERBOSE 信息，只有EXPLAIN 命令中的VERBOSE 选项为on 时才会显示
- `Buffers: shared hit=11`表明了从共享缓存中命中了11 个BLOCK，属于Seq Scan 节点的BUFFERS 信息，只有EXPLAIN 命令中的BUFFERS 选项为on 时才会显示
- `Planning time: 0.066 ms`表明了生成查询计划的时间
- `Execution time: 0.160 ms`表明了实际的SQL 执行时间，其中不包括查询计划的生成时间

### 6.3.2 索引扫描

&emsp;索引扫描，就是在索引中找出需要的数据行的物理位置，然后再到表中数据块中把相应的数据读出来的过程。

&emsp;一个`Index Scan`的示例如下：

```sql
postgres=> explain(ANALYZE,VERBOSE,BUFFERS) select * from class where st_no=2;
                                                       QUERY PLAN
--------------------------------------------------------------------------------------------
 Index Scan using no_index on public.class  (cost=0.28..8.29 rows=1 width=35) (actual time=0.022..0.023 rows=1 loops=1)
   Output: st_no, name
   Index Cond: (class.st_no = 2)
   Buffers: shared hit=3
 Planning time: 0.119 ms
 Execution time: 0.060 ms
(6 rows)
```

- `Index Scan using no_index on public.class`表明是使用的public.class 表的no_index 索引对表进行索引扫描的。

- `Index Cond: (class.st_no = 2)`表明索引扫描的条件是class.st_no = 2。

  可以看出，使用了索引之后，对相同表的相同条件的扫描速度变快了。这是因为从全表扫描变为索引扫描，通过Buffers: shared hit=3 可以看出，需要扫描的BLOCK（或者说元组）少了，所以需要的代价也就小了，速度也就快了。

### 6.3.3 覆盖索引

&emsp;覆盖索引扫描(IndexOnly Scan)指所需的返回结果能被所扫描的索引全部覆盖，例如：

```sql

postgres=> explain(ANALYZE,VERBOSE,BUFFERS) select st_no from class where st_no=2;
                                                         QUERY PLAN
-------------------------------------------------------------------------------------------
 Index Only Scan using no_index on public.class  (cost=0.28..4.29 rows=1 width=4) (actual time=0.015..0.016 rows=1 loops=1)
   Output: st_no
   Index Cond: (class.st_no = 2)
   Heap Fetches: 0
   Buffers: shared hit=3
 Planning time: 0.058 ms
 Execution time: 0.036 ms
(7 rows)
```

- `Index Only Scan using no_index on public.class` 表明使用public.class 表的no_index 索引对表进行覆盖索引扫描。
- `Heap Fetches` 表明需要扫描数据块的个数。

### 6.3.4 位图索引扫描

&emsp;位图索引扫描(Bitmap Index Scan)和索引扫描的过程类似，只是扫描得到的并非元祖(tuple,即数据行)，而是一个位图。

&emsp;以一个示例说明位图索引扫描的执行步骤：

![BitmapIndexScan示例](.\attachment\BitmapIndexScan示例.png)

&emsp;从图上可以看出，单次索引查找可能会读取同一个数据块多次。我们可以将数据块编码为bitmap，并将被命中的数据块赋值为1，则可以得到一个类似下图的位图。

![BitmapIndexScan示例-位图映射](.\attachment\BitmapIndexScan示例-位图映射.png)

&emsp;这样就可以得到一个数组`...101101...`，再根据这个数组访问相关数据块。一个数据块只会被访问一次，在该次访问中读出所有的数据tuple。

### 6.3.5 位图堆扫描

&emsp;通常作为位图索引扫描的父节点。针对单次查询使用多个索引的情况，可以利用位图进一步缩减数据块访问数量。例如：

```
表 t
字段 a,b
索引 ida(a),idb(b)
------------------------
查询语句:
select a,b
from t
where (a>100 or a < 200) and b = 30
```

&emsp;根据`位图索引扫描`可以得到各个索引的位图：

```
a<100   ...000000111000000...
or				||
a>200	...000000011100000...
and				&&
b=30	...000010110101000...
-------------------------------
result	...000000110100000...
```

&emsp;位图扫描是用内存和CPU开销来换取IO消耗。位图扫描通常在以下三个场景中出现：

- 部分b-tree索引，在不同列的and/or查询或者相同列的or查询（问题：为什么相同列的情况下不包含and场景）；
- brin索引。由于brin索引本身存储的就是连续块的元信息，所以无法精确查询，因此也必须首先构建heap page的bitmap串，并在`bitmap heap scan`阶段recheck条件。
- gin索引。它存储的是key，以及ctid（heap行号）组成的`posting list`或`posting tree`，虽然它理论上可以支持index scan，但PostgreSQL目前仅对GIN实施bitmap scan。

&emsp;一个示例：

```sql
bill=# create index idx_t11 on t1 using btree(c1);
CREATE INDEX
bill=# create index idx_t12 on t1 using btree(c2); 
CREATE INDEX
bill=# create index idx_t13 on t1 using btree(c3); 
CREATE INDEX
bill=# explain select * from t1 where c1 =10 and c2 =20 and c3 = 30;  
                                 QUERY PLAN                                  
-----------------------------------------------------------------------------
 Bitmap Heap Scan on t1  (cost=3.31..4.62 rows=1 width=12)
   Recheck Cond: ((c3 = 30) AND (c2 = 20))
   Filter: (c1 = 10)
   ->  BitmapAnd  (cost=3.31..3.31 rows=1 width=0)
         ->  Bitmap Index Scan on idx_t13  (cost=0.00..1.53 rows=10 width=0)
               Index Cond: (c3 = 30)
         ->  Bitmap Index Scan on idx_t12  (cost=0.00..1.53 rows=10 width=0)
               Index Cond: (c2 = 20)
(8 rows)
```



# 七、物理存储结构

[PostgreSQL存储结构]:https://blog.csdn.net/abcwywht/article/details/123758413
[PostgreSQL heap堆表 存储引擎实现原理]: https://blog.51cto.com/u_13456560/5823300

## 7.1 物理结构

&emsp;当我们使用PostgreSQL创建一个表时，PostgreSQL会在` $PGDATA/base `目录下生成一个**新的目录**，用来存储这个创建好的 db 的表数据，PostgreSQL不同数据库之间是完全**物理隔离**的。

&emsp;在postgreSQL中，一个表会对应一个逻辑文件，一个逻辑文件又分为若干物理段(relation segment)文件。除了最后一段外，物理段文件默认大小为40MB。以下是一个逻辑文件结构示意图：

![postgre存储结构-逻辑文件](.\attachment\postgre存储结构-逻辑文件.png)

## 7.2 文件页

&emsp;一个文件页也称为page，默认大小8KB。它的主要结构如下图所示：

![postgre存储结构-文件页（磁盘块）](.\attachment\postgre存储结构-文件页（磁盘块）.png)

&emsp;`PageHeader`又称页描述区，记载页的使用情况，如页分布格式版本，元数据空间和特殊空间的起始位置及文件页相关的事务日志记载点等信息。

&emsp;PageHeader的一些重要字段如下：

&emsp;pd_lsn: 一个8bit的unsigned int，与pg的WAL文件xlog相关，唯一标识上一个写入到这个page的请求。

&emsp;pd_checksum: 当前page的校验和，unit16。

&emsp;pd_flags: 当前page 的一些flag信息，比如是否有空的line pointer(可用的元组空间)，其内部的每一个元组是否对外可见等，uint16。

&emsp;pd_lower: 标识当前page 空闲空间的起始偏移位置。上图中自item2之后开始。

&emsp;pd_upper: 标识当前page空闲空间的结束偏移位置。上图中在Tuple2之前结束。

&emsp;pd_special: 特殊空间的起始偏移位置，其一般会存放在整个page的最后一段（上图Special Space）。

&emsp;pd_pagesize_version: 标识page大小和当前page版本信息。

&emsp;pd_prune_xid: 标识本页面可以回收的 最老的元组 id。

&emsp;`item`: 可变长度的数组，与`元组(tuple)`一一对应，用来存储每一个元组在当前page内部的起始偏移地址。

## 7.3 元祖

&emsp;元祖，即tuple，是实际存储的数据。它的结构如下：

# 附录

## 附录一、XXXXXXX