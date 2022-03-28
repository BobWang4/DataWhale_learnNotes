# SQL学习（六）

## 第六章 决胜秋招

### Section A

#### 练习一：各部门工资最高的员工

```sql
SELECT Department.name Department, e.name Employee, e.salary Salary
FROM ((SELECT name, salary, departmentId,
             rank() over (PARTITION BY departmentId ORDER BY salary DESC) ranking
      FROM Employee
      ) e
      JOIN Department ON e.departmentId = Department.id)
WHERE ranking = 1;
```

#### 练习二：换座位

```sql
SELECT (CASE WHEN MOD(id, 2) != 0 AND id = (SELECT COUNT(*) FROM Seat) THEN id
             WHEN MOD(id, 2) != 0 THEN id + 1
        ELSE id - 1
        END) id, student
FROM Seat
ORDER BY id;
```

#### 练习三：分数排名

```SQL
SELECT score, dense_rank() over (ORDER BY score DESC) AS 'rank'
FROM Scores;
```

#### 练习四：连续出现的数字

```sql
SELECT DISTINCT Num  ConsecutiveNums
FROM LOGS
WHERE (id + 1, Num) IN (SELECT * FROM LOGS) 
AND (id - 1, Num) IN (SELECT * FROM LOGS);
```

#### 练习五：树节点

```sql
SELECT id, 
       (CASE WHEN t1.p_id IS NULL THEN 'Root'
             WHEN t1.id IN (SELECT t2.p_id FROM tree t2) THEN 'Inner'
       ELSE 'Leaf'
       END) TYPE
FROM tree t1;
```

#### 练习六：至少有五名直接下属的经理

```sql
SELECT e1.Name 
FROM Employee e1
WHERE 5 <= (SELECT COUNT(*) FROM Employee e2 WHERE e1.Id = e2.ManagerId);
```

#### 练习七：查询回答率最高的问题

```sql
SELECT question_id survey_log
FROM survey_log
GROUP BY question_id
HAVING SUM(answer_id IS NOT NULL)/COUNT(*) =
(
    SELECT SUM(answer_id IS NOT NULL)/COUNT(*) prect
    FROM survey_log
    GROUP BY question_id
    ORDER BY prect DESC LIMIT 1
);
```

#### 练习八：各部门前3高工资的员工

```sql
SELECT d.name Department, s.name Employee, s.salary Salary 
FROM (SELECT departmentId, NAME, salary,
             DENSE_RANK() over(PARTITION BY departmentId ORDER BY salary DESC) AS ranking
      FROM  Employee e) s
JOIN Department d ON s.departmentId = d.id
WHERE s.ranking <= 3;
```

#### 练习九：平面上最近距离

```sql
SELECT ROUND(SQRT(MIN(POWER((p1.x-p2.x),2)+POWER((p1.y-p2.y),2))),2) AS shortest
FROM point_2d p1,point_2d p2
WHERE p1.x<>p2.x  OR p1.y<>p2.y;
```

#### 练习十：行程和用户

```sql
SELECT
    Request_at 'Day', ROUND(AVG(STATUS!='completed'), 2) 'Cancellation Rate'
FROM Trips t 
    JOIN Users u1 ON (t.Client_id = u1.Users_id AND u1.Banned = 'No')
    JOIN Users u2 ON (t.Driver_id = u2.Users_id AND u2.Banned = 'No')
WHERE	
    Request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY 
    Request_at;
```

ps.有几道题不是自己做出来的，在网上学习到了很多，太难了QAQ

### Section B	

#### 练习一：行转列

```sql
SELECT name,
       SUM(CASE WHEN subject = 'chinese' THEN score ELSE null END) as chinese,
       SUM(CASE WHEN subject = 'math' THEN score ELSE null END) as math,
       SUM(CASE WHEN subject = 'english' THEN score ELSE null END) as english
  FROM score
 GROUP BY name;
```

#### 练习二：列转行

```sql
SELECT
    NAME,
    'chinese' AS SUBJECT ,
    MAX('chinese') AS score
FROM score GROUP BY NAME
UNION
SELECT
    NAME,
    'math' AS SUBJECT ,
    MAX('数学') AS score
FROM score GROUP BY NAME
UNION
SELECT
    NAME,
    '英语' AS SUBJECT ,
    MAX('英语') AS score
FROM score GROUP BY NAME;
```

#### 
