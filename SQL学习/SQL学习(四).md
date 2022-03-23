# SQL学习（四）

学习来自datawhale

详情请参考：[ Follow me，从 0 到 1 掌握 SQL，决胜秋招。 (github.com)](https://github.com/datawhalechina/wonderful-sql)

## 第四章 集合运算

### 4.1 表的加减法

在标准 SQL 中, 分别对检索结果使用 `UNION`, `INTERSECT,` `EXCEPT` 来将检索结果进行并,交和差运算, 像`UNION`,`INTERSECT`, `EXCEPT`这种用来进行集合运算的运算符称为集合运算符。

#### 4.1.2 表的加法--union

例：

```sql
SELECT product_id, product_name
  FROM product
 UNION
SELECT product_id, product_name
  FROM product2;
```

 **UNION 等集合运算符通常都会除去重复的记录**

使用 UNION 对两个查询结果取并集, 和在一个查询中使用 WHERE 子句, 然后使用 OR 谓词连接两个查询条件, 能够得到相同的结果。

对于同一个表的两个不同的筛选结果集, 使用 UNION 对两个结果集取并集, 和把两个子查询的筛选条件用 OR 谓词连接, 会得到相同的结果, 但倘若要将两个不同的表中的结果合并在一起, 就不得不使用 UNION 了。

如果需要重复行：UNION ALL

#### 4.1.3 隐式数据类型转换

通常来说, 我们会把类型完全一致, 并且代表相同属性的列使用 UNION 合并到一起显示, 但有时候, 即使数据类型不完全相同, 也会通过隐式类型转换来将两个类型不同的列放在一列里显示, 例如字符串和数值类型:

```sql
SELECT product_id, product_name, '1'
  FROM Product
 UNION
SELECT product_id, product_name,sale_price
  FROM Product2;
```

<img src="https://github.com/datawhalechina/wonderful-sql/raw/main/img/ch04/ch04.07result4.png?raw=true" alt="img" style="zoom:150%;" />

```sql
SELECT SYSDATE(), SYSDATE(), SYSDATE()
 
 UNION
 
SELECT 'chars', 123,  null
```

<img src="https://github.com/datawhalechina/wonderful-sql/raw/main/img/ch04/ch04.08result5.png?raw=true" alt="img" style="zoom:150%;" />

#### 4.1.4 交运算--INTERSECT（MySQL8.0不支持）

```sql
SELECT product_id, product_name
  FROM Product
  
INTERSECT
SELECT product_id, product_name
  FROM Product2
```

需要用 inner join 来求得交集

```sql
SELECT p1.product_id, p1.product_name
  FROM Product p1
INNER JOIN Product2 p2
ON p1.product_id=p2.product_id
```

对于同一个表的两个查询结果而言, 他们的交INTERSECT实际上可以等价地将两个查询的检索条件用AND谓词连接来实现。

#### 4.1.5 差集，补集与表的减法

MySQL 8.0 还不支持 EXCEPT 运算

使用NOT IN实现：

```sql
-- 使用 NOT IN 子句的实现方法
SELECT * 
  FROM Product
 WHERE product_id NOT IN (SELECT product_id 
                            FROM Product2)
```

EXCEPT运算有相同的效果

#### 4.1.6 对称差

两个集合A,B的对称差是指那些仅属于A或仅属于B的元素构成的集合。

两个集合的对称差等于 A-B并上B-A

```sql
-- 使用Product表和Product2表的对称差来查询哪些商品只在其中一张表
-- 使用 NOT IN 实现两个表的差集
SELECT * 
  FROM Product
 WHERE product_id NOT IN (SELECT product_id FROM Product2)
UNION
SELECT * 
  FROM Product2
 WHERE product_id NOT IN (SELECT product_id FROM Product)
```

### 4.2 连结 (join)

连结(JOIN)就是使用某种关联条件(一般是使用相等判断谓词"="), 将其他表中的列添加过来, 进行“添加列”的集合运算。

#### 4.2.1 内连结 (INNER JOIN)

```sql
-- 内连结
FROM <tb_1> INNER JOIN <tb_2> ON <condition(s)>
```

其中 INNER 关键词表示使用了内连结

要点一: 进行连结时需要在 FROM 子句中使用多张表。

要点二:必须使用 ON 子句来指定连结条件。

要点三: SELECT 子句中的列最好按照 表名.列名 的格式来使用。

```sql
-- 找出每个商店里的衣服类商品的名称及价格等信息
SELECT  SP.shop_id,SP.shop_name,SP.product_id 
       ,P.product_name, P.product_type, P.purchase_price
  FROM shopproduct  AS SP 
 INNER JOIN Product AS P 
    ON SP.product_id = P.product_id
 WHERE P.product_type = '衣服';
```

结合 GROUP BY 子句使用内连结, 需要根据分组列位于哪个表区别对待。

```sql
-- 每个商店中, 售价最高的商品的售价分别是多少?
SELECT SP.shop_id
      ,SP.shop_name
      ,MAX(P.sale_price) AS max_price
  FROM shopproduct AS SP
 INNER JOIN product AS P
    ON SP.product_id = P.product_id
 GROUP BY SP.shop_id,SP.shop_name
```

#### 4.2.2 自连结 (SELF JOIN)

自连结可以是外连结也可以是内连结

```sql
-- 找出每个商品种类当中售价高于该类商品的平均售价的商品
-- 关联子查询
SELECT product_type, product_name, sale_price
  FROM product AS P1
 WHERE sale_price > (SELECT AVG(sale_price)
                       FROM product AS P2
                      WHERE P1.product_type = P2.product_type
                      GROUP BY product_type);】
-- 自连结
SELECT  P1.product_id
       ,P1.product_name
       ,P1.product_type
       ,P1.sale_price
       ,P2.avg_price
  FROM Product AS P1
 INNER JOIN 
   (SELECT product_type,AVG(sale_price) AS avg_price 
      FROM Product 
     GROUP BY product_type) AS P2 
    ON P1.product_type = P2.product_type
 WHERE P1.sale_price > P2.avg_price;
```

#### 4.2.3 自然连结 (NATURAL JOIN)

自然连结并不是区别于内连结和外连结的第三种连结, 它其实是内连结的一种特例--当两个表进行自然连结时, 会按照两个表中都包含的列名来进行等值内连结, 此时无需使用 ON 来指定连接条件。

```sql
SELECT *  FROM shopproduct NATURAL JOIN Product
```

#### 4.2.4 使用连结求交集

```sql
-- 使用内连结求 Product 表和 Product2 表的交集
SELECT P1.*
  FROM Product AS P1
 INNER JOIN Product2 AS P2
    ON (P1.product_id  = P2.product_id
   AND P1.product_name = P2.product_name
   AND P1.product_type = P2.product_type
   AND P1.sale_price   = P2.sale_price
   AND P1.regist_date  = P2.regist_date)
```

**注意：两个缺失值用等号进行比较, 结果不为真**

#### 4.2.5 外连结 (OUTER JOIN)

内连结会丢弃两张表中不满足 ON 条件的行,和内连结相对的就是外连结. 外连结会根据外连结的种类有选择地保留无法匹配到的行。

按照保留的行位于哪张表,外连结有三种形式: 左连结, 右连结和全外连结。

左连结会保存左表中无法按照 ON 子句匹配到的行, 此时对应右表的行均为缺失值; 右连结则会保存右表中无法按照 ON 子句匹配到的行, 此时对应左表的行均为缺失值; 而全外连结则会同时保存两个表中无法按照 ON子句匹配到的行, 相应的另一张表中的行用缺失值填充。

三种外连结的对应语法分别为:

```sql
-- 左连结     
FROM <tb_1> LEFT  OUTER JOIN <tb_2> ON <condition(s)>
-- 右连结     
FROM <tb_1> RIGHT OUTER JOIN <tb_2> ON <condition(s)>
-- 全外连结
FROM <tb_1> FULL  OUTER JOIN <tb_2> ON <condition(s)>
```

由于连结时可以交换左表和右表的位置, 因此左连结和右连结并没有本质区别.

```sql
-- 统计每种商品分别在哪些商店有售, 需要包括那些在每个商店都没货的商品。
SELECT SP.shop_id
       ,SP.shop_name
       ,SP.product_id
       ,P.product_name
       ,P.sale_price
  FROM Product AS P
  LEFT OUTER JOIN ShopProduct AS SP
    ON SP.product_id = P.product_id;
```

**● 外连结要点 1: 选取出单张表中全部的信息**

**● 外连结要点 2：使用 LEFT、RIGHT 来指定主表**

使用外连结从ShopProduct表和Product表中找出那些在某个商店库存少于50的商品及对应的商店.

```sql
SELECT P.product_id
      ,P.product_name
      ,P.sale_price
       ,SP.shop_id
      ,SP.shop_name
      ,SP.quantity 
  FROM Product AS P
  LEFT OUTER JOIN-- 先筛选quantity<50的商品
   (SELECT *
      FROM ShopProduct
     WHERE quantity < 50 ) AS SP
    ON SP.product_id = P.product_id
```

先写个子查询,用来从ShopProduct表中筛选quantity<50的商品, 然后再把这个子查询和主表连结起来。（防止缺失值造成影响）

MySQL8.0 目前还不支持全外连结, 不过我们可以对左连结和右连结的结果进行 UNION 来实现全外连结。

#### 4.2.6 多表连结

（1）多表进行内连结

连结第三张表的时候, 也是通过 ON 子句指定连结条件。

即使想要把连结的表增加到 4 张、5 张……使用 INNER JOIN 进行添加的方式也是完全相同的。

（2）多表进行外连结

正如之前所学发现的, 外连结一般能比内连结有更多的行, 从而能够比内连结给出更多关于主表的信息, 多表连结的时候使用外连结也有同样的作用。

#### 4.2.7 ON 子句进阶--非等值连结

实际上, 包括比较运算符(<,<=,>,>=, BETWEEN)和谓词运算(LIKE, IN, NOT 等等)在内的所有的逻辑运算都可以放在 ON 子句内作为连结条件。

注 1: COUNT 函数的参数是列名时, 会忽略该列中的缺失值, 参数为 * 时则不忽略缺失值。

#### 4.2.8 交叉连接 CROSS JOIN(笛卡尔积)

交叉连结的语法有如下几种形式:

```sql
-- 1.使用关键字 CROSS JOIN 显式地进行交叉连结
SELECT SP.shop_id
       ,SP.shop_name
       ,SP.product_id
       ,P.product_name
       ,P.sale_price
  FROM ShopProduct AS SP
 CROSS JOIN Product AS P;
--2.使用逗号分隔两个表,并省略 ON 子句
SELECT SP.shop_id
       ,SP.shop_name
       ,SP.product_id
       ,P.product_name
       ,P.sale_price
  FROM ShopProduct AS SP , Product AS P;
```

#### 4.2.9 连结的特定语法和过时语法

使用过时语法的内连结

```sql
SELECT SP.shop_id
       ,SP.shop_name
       ,SP.product_id
       ,P.product_name
       ,P.sale_price
  FROM shopproduct SP, product P
 WHERE SP.product_id = P.product_id
   AND SP.shop_id = '000A';
```

ps. 我才发现过去一直使用的都是过时的QAQ

第一,使用这样的语法无法马上判断出到底是内连结还是外连结（又或者是其他种类的连结）。

第二,由于连结条件都写在 WHERE 子句之中,因此无法在短时间内分辨出哪部分是连结条件,哪部分是用来选取记录的限制条件。

第三,我们不知道这样的语法到底还能使用多久.每个 DBMS 的开发者都会考虑放弃过时的语法,转而支持新的语法.虽然并不是马上就不能使用了,但那一天总会到来的。

### 4.3 练习题

1、找出 product 和 product2 中售价高于 500 的商品的基本信息。

```sql
 SELECT *
 FROM product 
 WHERE sale_price > 500
 UNION 
 SELECT *
 FROM product2
 WHERE sale_price > 500;
```

2、借助对称差的实现方式, 求product和product2的交集。

```sql
SELECT * 
  FROM Product
 WHERE product_id IN (SELECT product_id FROM Product2);
```

3、每类商品中售价最高的商品都在哪些商店有售 ？

```sql
-- 第三步，求出各id的商品在哪些商店有售
SELECT s.product_id, shop_name, product_type
FROM ( -- 第二步，求出每类商品最高售价对应的商品id
		SELECT product_id, m.product_type
		FROM (SELECT * FROM product UNION SELECT * FROM product2) all_product
		JOIN ( -- 第一步，求出各类商品分别对应的最高售价
				SELECT MAX(sale_price) MAX, product_type 
				FROM (SELECT * FROM product UNION SELECT * FROM product2) all_product
				GROUP BY product_type
			  ) m
		ON all_product.product_type = m.product_type
		WHERE sale_price = MAX
	   ) id
LEFT JOIN ShopProduct s
ON id.product_id = s.product_id;
```

4、分别使用内连结和关联子查询每一类商品中售价最高的商品

```sql
-- 关联子查询
SELECT * 
FROM (SELECT * FROM product UNION SELECT * FROM product2) p1
WHERE sale_price = (
		SELECT MAX(sale_price)
		FROM (SELECT * FROM product UNION SELECT * FROM product2) p2
                WHERE p2.product_type = p1.product_type
		GROUP BY product_type
 );
-- 自连结
select product_id
from (select * from product union select * from product2) all_product
join (select max(sale_price) max, product_type 
	 from (select * from product union select * from product2) all_product
	 group by product_type) m
on all_product.product_type = m.product_type
where sale_price = max;
```

5、用关联子查询实现：在 product 表中，取出 product_id, product_name, sale_price, 并按照商品的售价从低到高进行排序、对售价进行累计求和。

```sql
select product_id, product_name, sale_price, 
	(
		select sum(sale_price)
        from product p2
        where p2.sale_price <= p1.sale_price
	) sum
from product p1
order by sale_price;
```

