# SQL 学习（一）

学习来自datawhale

详情请参考：[ Follow me，从 0 到 1 掌握 SQL，决胜秋招。 (github.com)](https://github.com/datawhalechina/wonderful-sql)

## 第一章：初识数据库

### 1.1 初识数据库

数据库（Database，DB）：数据集合
数据库管理系统（Database Management System，DBMS）

#### 1.1.1 DBMS种类

- 层次数据库（Hierarchical Database，HDB）

- 关系数据库（Relational Database，RDB）

  - Oracle Database：甲骨文公司的RDBMS
  - SQL Server：微软公司的RDBMS
  - DB2：IBM公司的RDBMS
  - PostgreSQL：开源的RDBMS
  - MySQL：开源的RDBMS

  如上是5种具有代表性的RDBMS，其特点是由行和列组成的二维表来管理数据，这种类型的 DBMS 称为关系数据库管理系统（Relational Database Management System，RDBMS）。

- 面向对象数据库（Object Oriented Database，OODB）

- XML数据库（XML Database，XMLDB）

- 键值存储系统（Key-Value Store，KVS），举例：MongoDB

### 1.2 初识 SQL

SQL 语句可以分为以下三类

- **DDL** ：DDL（Data Definition Language，数据定义语言） 用来创建或者删除存储数据用的数据库以及数据库中的表等对象。
  - CREATE ： 创建数据库和表等对象
  - DROP ： 删除数据库和表等对象
  - ALTER ： 修改数据库和表等对象的结构
- **DML** :DML（Data Manipulation Language，数据操纵语言） 用来查询或者变更表中的记录。

  - SELECT ：查询表中的数据

  - INSERT ：向表中插入新数据

  - UPDATE ：更新表中的数据

  - DELETE ：删除表中的数据
- **DCL** ：DCL（Data Control Language，数据控制语言） 用来确认或者取消对数据库中的数据进行的变更。除此之外，还可以对 RDBMS 的用户是否有权限操作数据库中的对象（数据库表等）进行设定。
  - COMMIT ： 确认对数据库中的数据进行的变更
  - ROLLBACK ： 取消对数据库中的数据进行的变更
  - GRANT ： 赋予用户操作权限
  - REVOKE ： 取消用户的操作权限

#### 1.2.1  数据库的创建（ CREATE DATABASE 语句）

```sql
CREATE DATABASE < 数据库名称 > ;
```

#### 1.2.2 表的创建（ CREATE TABLE 语句）

```sql
CREATE TABLE < 表名 >
( < 列名 1> < 数据类型 > < 该列所需约束 > ,
  < 列名 2> < 数据类型 > < 该列所需约束 > ,
  < 列名 3> < 数据类型 > < 该列所需约束 > ,
  < 列名 4> < 数据类型 > < 该列所需约束 > ,
  .
  .
  .
  < 该表的约束 1> , < 该表的约束 2> ,……);
```

（练习时，遇到了一个奇怪的bug，没有识别到primary是关键字，报1064错误）

#### 1.2.3 表的删除和更新

删除表：

```sql
DROP TABLE < 表名 > ;
```

需要特别注意的是，删除的表是无法恢复的，只能重新插入，请执行删除操作时要特别谨慎。

添加列：

```sql
ALTER TABLE < 表名 > ADD COLUMN <列名> < 列的定义 >;
```

删除列：

```sql
ALTER TABLE < 表名 > DROP COLUMN < 列名 >;
```

删除表中特定的行：

```sql
DELETE FROM product WHERE COLUMN_NAME='XXX';
```

清空表内容：

```sql
TRUNCATE TABLE TABLE_NAME;
```

优点：速度快

数据的更新：

```sql
UPDATE <表名>
   SET <列名> = <表达式> [, <列名2>=<表达式2>...];  
 WHERE <条件>;  -- 可选，非常重要。
 ORDER BY 子句;  --可选
 LIMIT 子句; --可选
```

#### 1.2.4 插入数据

```sql
INSERT INTO <表名> (列1, 列2, 列3, ……) VALUES (值1, 值2, 值3, ……);  
```

#### 1.2.5 索引

MySQL索引的建立对于MySQL的高效运行是很重要的，索引可以大大提高MySQL的检索速度。

创建表时可以直接创建索引

```sql
CREATE TABLE mytable(  
 
ID INT NOT NULL,   
 
username VARCHAR(16) NOT NULL,  
 
INDEX [indexName] (username(length))  
 
);  
```

```sql
-- 方法1
CREATE INDEX indexName ON table_name (column_name)

-- 方法2
ALTER table tableName ADD INDEX indexName(columnName)
```

索引分类

- 主键索引

建立在主键上的索引被称为主键索引，一张数据表只能有一个主键索引，索引列值不允许有空值，通常在创建表时一起创建。

- 唯一索引

建立在UNIQUE字段上的索引被称为唯一索引，一张表可以有多个唯一索引，索引列值允许为空，列值中出现多个空值不会发生重复冲突。

- 普通索引

建立在普通字段上的索引被称为普通索引。

- 前缀索引

前缀索引是指对字符类型字段的前几个字符或对二进制类型字段的前几个bytes建立的索引，而不是在整个字段上建索引。前缀索引可以建立在类型为char、varchar、binary、varbinary的列上，可以大大减少索引占用的存储空间，也能提升索引的查询效率。

- 全文索引

利用“分词技术”实现在长文本中搜索关键字的一种索引。

语法：
```sql
SELECT * FROM article WHERE MATCH (col1，col2，...) AGAINST (expr [ search _ modifier ])
```

- 单列索引

建立在单个列上的索引被称为单列索引。

- 联合索引（复合索引、多列索引）

建立在多个列上的索引被称为联合索引，又叫复合索引、组合索引。

### 1.3 练习题

1、

```sql
CREATE TABLE Addressbook
(regist_no INTEGER NOT NULL,
`name` VARCHAR(128) NOT NULL,
address VARCHAR(256) NOT NULL,
tel_no CHAR(10),
mail_address CHAR(20),
PRIMARY KEY (regist_no)
);
```

注意：name需要用``包括。

2、

```sql
ALTER TABLE Addressbook ADD COLUMN postal_code CHAR(8) NOT NULL;
```

3、

```sql
DROP TABLE Addressbook;
```

4、

不可以，数据库表一旦删除，就不能恢复。
