# SQL学习（三）

学习来自datawhale

详情请参考：[ Follow me，从 0 到 1 掌握 SQL，决胜秋招。 (github.com)](https://github.com/datawhalechina/wonderful-sql)

## 第三章 复杂一点的查询

### 3.1 视图

视图是一个虚拟的表

1. 通过定义视图可以将频繁使用的SELECT语句保存以提高效率。
2. 通过定义视图可以使用户看到的数据更加清晰。
3. 通过定义视图可以不对外公开数据表全部字段，增强数据的保密性。
4. 通过定义视图可以降低数据的冗余。

#### 3.1.1 创建视图

```sql
CREATE VIEW <视图名称>(<列名1>,<列名2>,...) AS <SELECT语句>
```

其中SELECT 语句需要书写在 AS 关键字之后。可以单表，也可以多表（select语句中）。

需要注意的是在一般的DBMS中定义视图时不能使用ORDER BY语句。

#### 3.1.2 修改视图

```sql
ALTER VIEW <视图名> AS <SELECT语句>
```

其中视图名在数据库中需要是唯一的，不能与其他视图和表重名。

#### 3.1.3 更新视图内容

对于一个视图来说，如果包含以下结构的任意一种都是不可以被更新的：

- 聚合函数 SUM()、MIN()、MAX()、COUNT() 等。
- DISTINCT 关键字。
- GROUP BY 子句。
- HAVING 子句。
- UNION 或 UNION ALL 运算符。
- FROM 子句中包含多个表。

如果原表可以更新，那么 视图中的数据也可以更新。反之亦然。

#### 3.1.4 删除视图

```sql
DROP VIEW <视图名1> [ , <视图名2> …]
```

注意：需要有相应的权限才能成功删除。

### 3.2 子查询

子查询指一个查询语句嵌套在另一个查询语句内部的查询

子查询就是将用来定义视图的 SELECT 语句直接用于 FROM 子句当中。而且由于子查询是一次性的，所以子查询不会像视图那样保存在存储介质中， 而是在 SELECT 语句执行之后就消失了。

举个例子：

```sql
SELECT stu_name
FROM (
         SELECT stu_name, COUNT(*) AS stu_cnt
          FROM students_info
          GROUP BY stu_age) AS studentSum;
```

#### 3.2.1 标量子查询

标量就是单一的意思

返回表中具体的**某一行的某一列**

在类似以下的需求的地方使用：

1. 查询出销售单价高于平均销售单价的商品
2. 查询出注册日期最晚的那个商品

举例：

```sql
--通过标量子查询语句查询出销售单价高于平均销售单价的商品
SELECT product_id, product_name, sale_price
  FROM product
 WHERE sale_price > (SELECT AVG(sale_price) FROM product);
```

由于标量子查询的特性，导致标量子查询不仅仅局限于 WHERE 子句中，通常任何可以使用单一值的位置都可以使用。也就是说， 能够使用常数或者列名的地方，无论是 SELECT 子句、GROUP BY 子句、HAVING 子句，还是 ORDER BY 子句，几乎所有的地方都可以使用。

#### 3.2.2 关联子查询

查询与子查询之间存在着联系

类似以下需求时使用：

```sql
--选取出各商品种类中高于该商品种类的平均销售单价的商品
SELECT product_type, product_name, sale_price
  FROM product AS p1
 WHERE sale_price > (SELECT AVG(sale_price)
   FROM product AS p2
                      WHERE p1.product_type =p2.product_type
   GROUP BY product_type);
```

1. 首先执行不带WHERE的主查询
2. 根据主查询讯结果匹配product_type，获取子查询结果
3. 将子查询结果再与主查询结合执行完整的SQL语句

### 3.3 小结与练习题

视图和子查询是数据库操作中较为基础的内容，对于一些复杂的查询需要使用子查询加一些条件语句组合才能得到正确的结果。但是无论如何对于一个SQL语句来说都不应该设计的层数非常深且特别复杂，不仅可读性差而且执行效率也难以保证，所以尽量有简洁的语句来完成需要的功能。

1、

```sql
CREATE VIEW ViewPractice5_1(product_name, sale_price, regist_date)
AS
  SELECT product_name, sale_price, regist_date
  FROM product
  WHERE sale_price >= 1000 
       AND regist_date = '2009-09-20'
```

2、报错，不能插入。原因是在视图中插入数据时，原表中也会进行，但原表中有其他字段是有非空要求的。

3、

```sql
SELECT product_id, product_name, product_type, sale_price, 
		(SELECT AVG(sale_price) FROM product) AS sale_price_avg
FROM product;
```

4、

```sql
CREATE VIEW AvgPriceByType
AS
  SELECT product_id, product_name, product_type, sale_price,
  		(SELECT AVG(sale_price) FROM product AS p2 WHERE p1.product_type = p2.product_type) AS sale_price_avg_type
  FROM product AS p1;
```

### 3.4 函数

- 算术函数 （用来进行数值计算的函数）
- 字符串函数 （用来进行字符串操作的函数）
- 日期函数 （用来进行日期操作的函数）
- 转换函数 （用来转换数据类型和值的函数）
- 聚合函数 （用来进行数据聚合的函数）

#### 3.4.1 算术函数

- ABS -- 绝对值

  语法：`ABS( 数值 )`

  当 ABS 函数的参数为`NULL`时，返回值也是`NULL`。

- MOD -- 求余数

  语法：`MOD( 被除数，除数 )`

  只能对整数列求余数

- ROUND -- 四舍五入

  ROUND( 对象数值，保留小数的位数 )

  注意：当参数 **保留小数的位数** 为变量时，可能会遇到错误，请谨慎使用变量。

#### 3.4.2 字符串函数

- CONCAT -- 拼接

  语法：`CONCAT(str1, str2, str3)`

- LENGTH -- 字符串长度

  语法：`LENGTH( 字符串 )`

- LOWER -- 小写转换

  LOWER 函数只能针对英文字母使用，它会将参数中的字符串全都转换为小写。

  类似的， UPPER 函数用于大写转换。

- REPLACE -- 字符串的替换

  语法：`REPLACE( 对象字符串，替换前的字符串，替换后的字符串 )`

- SUBSTRING -- 字符串的截取

  语法：`SUBSTRING （对象字符串 FROM 截取的起始位置 FOR 截取的字符数）`

  使用 SUBSTRING 函数 可以截取出字符串中的一部分字符串。截取的起始位置从字符串最左侧开始计算，索引值起始为**1**。

- SUBSTRING_INDEX -- 字符串按索引截取

  语法：`SUBSTRING_INDEX (原始字符串， 分隔符，n)`

  该函数用来获取原始字符串按照分隔符分割后，第 n 个分隔符之前（或之后）的子字符串，支持正向和反向索引，索引起始值分别为 **1** 和 **-1**。

- REPEAT -- 字符串按需重复多次

  语法：`REPEAT(string, number)`

#### 3.4.3 日期函数

- CURRENT_DATE -- 获取当前日期

- CURRENT_TIME -- 当前时间

- CURRENT_TIMESTAMP -- 当前日期和时间

- EXTRACT -- 截取日期元素

  语法：`EXTRACT(日期元素 FROM 日期)`

  使用 EXTRACT 函数可以截取出日期数据中的一部分，例如“年”，“月”，或者“小时”“秒”等。

  该函数的返回值并不是日期类型而是数值类型

#### 3.4.4 转换函数

- CAST -- 类型转换

  语法：`CAST（转换前的值 AS 想要转换的数据类型）`

- COALESCE -- 将NULL转换为其他值

  语法：`COALESCE(数据1，数据2，数据3……)`

  COALESCE 是 SQL 特有的函数。该函数会返回可变参数 A 中左侧开始第 1个不是NULL的值。

  参数个数是可变的，因此可以根据需要无限增加。

  在 SQL 语句中将 NULL 转换为其他值时就会用到转换函数。

### 3.5 谓词

谓词就是返回值为真值的函数。包括`TRUE / FALSE / UNKNOWN`。

谓词主要有以下几个：

- LIKE
- BETWEEN
- IS NULL、IS NOT NULL
- IN
- EXISTS

#### 3.5.1 LIKE谓词 -- 用于字符串的部分一致查询

- `%`是代表“零个或多个任意字符串”的特殊符号。
- `_`下划线匹配任意 1 个字符

#### 3.5.2 BETWEEN谓词 -- 用于范围查询

BETWEEN包含临界值，是闭区间

#### 3.5.3 IS NULL、 IS NOT NULL -- 用于判断是否为NULL

#### 3.5.4 IN谓词 -- OR的简便用法

`IN(值1, 值2, 值3, ......)`

使用IN 和 NOT IN 时是无法选取出NULL数据的。

#### 3.5.5 EXIST 谓词

- EXIST谓词的使用方法

谓词的作用就是 **“判断是否存在满足某种条件的记录”**。

如果存在这样的记录就返回真（TRUE），如果不存在就返回假（FALSE）。

EXIST（存在）谓词的主语是“记录”。

举个例子：

```sql
SELECT product_name, sale_price
  FROM product AS p
 WHERE EXISTS (SELECT *
                 FROM shopproduct AS sp
                WHERE sp.shop_id = '000C'
                  AND sp.product_id = p.product_id);
```

### 3.6 CASE 表达式

语法：

```sql
CASE WHEN <求值表达式> THEN <表达式>
     WHEN <求值表达式> THEN <表达式>
     WHEN <求值表达式> THEN <表达式>
     .
     .
     .
ELSE <表达式>
END  
```

上述语句执行时，依次判断 when 表达式是否为真值，是则执行 THEN 后的语句，如果所有的 when 表达式均为假，则执行 ELSE 后的语句。 无论多么庞大的 CASE 表达式，最后也只会返回一个值。

- **应用场景1：根据不同分支得到不同列值**

假设现在 要实现如下结果：

A ：衣服
B ：办公用品
C ：厨房用具  

因为表中的记录并不包含“A ： ”或者“B ： ”这样的字符串，所以需要在 SQL 中进行添加。并将“A ： ”“B ： ”“C ： ”与记录结合起来。

```sql
SELECT  product_name,
        CASE WHEN product_type = '衣服' THEN CONCAT('A ： ',product_type)
             WHEN product_type = '办公用品'  THEN CONCAT('B ： ',product_type)
             WHEN product_type = '厨房用具'  THEN CONCAT('C ： ',product_type)
             ELSE NULL
        END AS abc_product_type
  FROM  product;
```

ELSE 子句也可以省略不写，这时会被默认为 ELSE NULL。但为了防止有人漏读，还是希望大家能够显式地写出 ELSE 子句。 此外， CASE 表达式最后的“END”是不能省略的

- **应用场景2：实现列方向上的聚合**

通常我们使用如下代码实现行的方向上不同种类的聚合

```sql
-- 对按照商品种类计算出的销售单价合计值进行行列转换
SELECT SUM(CASE WHEN product_type = '衣服' THEN sale_price ELSE 0 END) AS sum_price_clothes,
       SUM(CASE WHEN product_type = '厨房用具' THEN sale_price ELSE 0 END) AS sum_price_kitchen,
       SUM(CASE WHEN product_type = '办公用品' THEN sale_price ELSE 0 END) AS sum_price_office
  FROM product;
+-------------------+-------------------+------------------+
| sum_price_clothes | sum_price_kitchen | sum_price_office |
+-------------------+-------------------+------------------+
|              5000 |             11180 |              600 |
+-------------------+-------------------+------------------+
1 row in set (0.00 sec)
```

- 应用场景3：实现行转列

<img src="https://github.com/datawhalechina/wonderful-sql/blob/main/img/ch03/ch03.11casewhen1.png?raw=true" alt="img" style="zoom:150%;" />

```sql
-- CASE WHEN 实现数字列 score 行转列
SELECT name,
       SUM(CASE WHEN subject = '语文' THEN score ELSE null END) as chinese,
       SUM(CASE WHEN subject = '数学' THEN score ELSE null END) as math,
       SUM(CASE WHEN subject = '外语' THEN score ELSE null END) as english
  FROM score
 GROUP BY name;
+------+---------+------+---------+
| name | chinese | math | english |
+------+---------+------+---------+
| 张三 |      93 |   88 |      91 |
| 李四 |      87 |   90 |      77 |
+------+---------+------+---------+
2 rows in set (0.00 sec)
```

总结：

- 当待转换列为数字时，可以使用`SUM AVG MAX MIN`等聚合函数；
- 当待转换列为文本时，可以使用`MAX MIN`等聚合函数

### 3.7 练习题（二）

5、运算或者函数中含有 NULL 时，结果全都会变为NULL ？

答：不一定，如 SUM()， COALESCE() 等函数中含有 NULL 时结果不会全变为 NULL。

6、① 得到的结果为
​	product_name	purchase_price
​	打孔器	            320
​	擦菜板	            790

② 得到的结果为空

7、

```sql
 SELECT 
    COUNT(CASE WHEN sale_price <= 1000 THEN sale_price END) AS low_price,
    COUNT(CASE WHEN sale_price BETWEEN 1001 AND 3000 THEN sale_price END) AS mid_price,
    COUNT(CASE WHEN sale_price >= 3001 THEN sale_price END) AS high_price
 FROM product;
```

```sql
 SELECT 
       SUM(CASE WHEN sale_price <= 1000 THEN 1 ELSE 0 END) AS low_price,
       SUM(CASE WHEN sale_price BETWEEN 1001 AND 3000 THEN 1 ELSE 0 END) AS mid_price,
       SUM(CASE WHEN sale_price >= 3001 THEN 1 ELSE 0 END) AS high_price
  FROM product;
```

