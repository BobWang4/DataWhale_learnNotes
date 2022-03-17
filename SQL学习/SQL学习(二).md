# SQL学习（二）

学习来自datawhale

详情请参考：[ Follow me，从 0 到 1 掌握 SQL，决胜秋招。 (github.com)](https://github.com/datawhalechina/wonderful-sql)

## 第二章：基础查询与排序

### 2.1 SELECT语句基础

```sql
SELECT <列名>, 
  FROM <表名>;
```

```sql
SELECT <列名>, ……
  FROM <表名>
 WHERE <条件表达式>;
```

- 在SELECT语句中使用DISTINCT可以删除重复行。
- 注释是SQL语句中用来标识说明或者注意事项的部分。分为1行注释"-- "和多行注释两种"/* */"

### 2.2 算术运算符和比较运算符

#### 2.2.1 算术运算符

+、-、*、/四个

#### 2.2.2 比较运算符

| 符号 | 含义     |
| ---- | -------- |
| =    | 等于     |
| <>   | 不等于   |
| >=   | 大于等于 |
| >    | 大于     |
| <=   | 小于等于 |
| <    | 小于     |

- 字符串类型的数据原则上按照字典顺序进行排序，不能与数字的大小顺序混淆。

- 希望选取NULL记录时，需要在条件表达式中使用IS NULL运算符。希望选取不是NULL的记录时，需要在条件表达式中使用IS NOT NULL运算符。

### 2.3 逻辑运算符

#### 2.3.1 NOT、AND、OR

#### 2.3.2 真值表

<img src="https://github.com/datawhalechina/wonderful-sql/blob/main/img/ch02/ch02.03true.png?raw=true" alt="img"  />

<img src="https://github.com/datawhalechina/wonderful-sql/blob/main/img/ch02/ch02.05true3.png?raw=true" alt="img"  />

复杂运算可以通过真值表来梳理逻辑关系

### 2.4 练习题（第一部分）

1、

```sql
SELECT product_name , regist_date
FROM product
WHERE regist_date > '2009-04-28';
```

2、

3条select语句均错误，NULL不能使用比较运算符，而是使用is null或者 is not null。

3、

```sql
SELECT product_name, sale_price, purchase_price
FROM product
WHERE sale_price - purchase_price >= 500;
```

4、

```sql
SELECT product_name, product_type, (sale_price * 0.9 - purchase_price) AS profit
FROM product
WHERE sale_price * 0.9 - purchase_price > 100 AND product_type IN ('办公用品', '厨房用具');
```

### 2.5 聚合

- COUNT：计算表中的记录数（行数）
- SUM：计算表中数值列中数据的合计值
- AVG：计算表中数值列中数据的平均值
- MAX：求出表中任意列中数据的最大值
- MIN：求出表中任意列中数据的最小值

常用法则：

- COUNT函数的结果根据参数的不同而不同。COUNT(*)会得到包含NULL的数据行数，而COUNT(<列名>)会得到NULL之外的数据行数。
- 聚合函数会将NULL排除在外。但COUNT(*)例外，并不会排除NULL。
- MAX/MIN函数几乎适用于所有数据类型的列。SUM/AVG函数只适用于数值类型的列。
- 想要计算值的种类时，可以在COUNT函数的参数中使用DISTINCT。
- 在聚合函数的参数中使用DISTINCT，可以删除重复数据。

### 2.6 分组 ：group by

```sql
SELECT <列名1>,<列名2>, <列名3>, ……
  FROM <表名>
 GROUP BY <列名1>, <列名2>, <列名3>, ……;
```

聚合键中包含NULL时，会将NULL作为一组特殊数据进行处理

GROUP BY的子句书写顺序有严格要求：1 SELECT → 2. FROM → 3. WHERE → 4. GROUP BY

其中前三项用于筛选数据，GROUP BY对筛选出的数据进行处理

GROUP BY 子句中不能使用SELECT 子句中定义的别名

### 2.7 为聚合结果指定条件：having

HAVING子句用于对分组进行过滤，可以使用数字、聚合函数和GROUP BY中指定的列名（聚合键）。

### 2.8 排序

```sql
SELECT <列名1>, <列名2>, <列名3>, ……
  FROM <表名>
 ORDER BY <排序基准列1>, <排序基准列2>, ……
```

默认为升序排列，降序排列为DESC(写在排序基准列之后)

 ORDER BY中列名可使用别名

在MySQL中，`NULL` 值被认为比任何 `非NULL` 值低，因此，当顺序为 ASC（升序）时，`NULL` 值出现在第一位，而当顺序为 DESC（降序）时，则排序在最后。

如果想指定存在 `NULL` 的行出现在首行或者末行，需要特殊处理。

- 将 `NULL` 值排在末行，同时将所有 `非NULL` 值按升序排列。

对于数字或者日期类型，可以在排序字段前添加一个负号（minus）来得到反向排序。（降序）

对于字符型或者字符型数字，可以使用 `IS NULL` 比较运算符。另外 `ISNULL( )` 函数等同于使用 `IS NULL` 比较运算符。

 ```sql
 ORDER BY <排序基准列> is NULL, <排序基准列> ASC
 ```

- 将 `NULL` 值排在首行，同时将所有 `非NULL` 值按倒序排列。

与上述相反

另一个小技巧，在固定长度的列排序时，可以使用coalesce函数排序

```sql
ORDER BY coalesce(<排序基准列>,'zzzz....')ASC
```

其中'zzzz...'表示定长的最后一个，如果为空，返回zzzzz，所以排最后

### 2.9 练习题

5、

```sql
SELECT product_id, SUM（product_name）
--本SELECT语句中存在错误。
  FROM product 
 GROUP BY product_type 
 WHERE regist_date > '2009-09-01';
```

where语句书写时应在group by之前；

group by与select子句的字段不同，即SELECT子句中如果出现列名，只能是GROUP BY子句中指定的列名（也就是聚合键）

6、

<img src="https://github.com/datawhalechina/wonderful-sql/blob/main/img/ch02/ch02.08test26.png?raw=true" alt="img"  />

```sql
 SELECT product_type, SUM(sale_price), SUM(purchase_price)
 FROM product
 GROUP BY product_type
 HAVING SUM(sale_price) > SUM(purchase_price) * 1.5;
```

7、

<img src="https://github.com/datawhalechina/wonderful-sql/blob/main/img/ch02/ch02.09test27.png?raw=true" alt="img"  />

```sql
 SELECT *
 FROM product
 ORDER BY - regist_date, sale_price;
```
