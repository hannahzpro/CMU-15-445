# Advanced SQL

### Relational Languages

#### SQL的组成

+ Data Manipulation Language (DML)

  数据操作语言：insert, select, update......

+ Data Definition Language  (DDL)

  通过定义schemas来创建表存储数据

+ Data Control Language (DCL)

  安全性授权（控制哪些人可以读取哪些数据）

==注意==：*Relational algebra is based on sets (unordered, no duplicates). SQL is based on bags (unordered, allows duplicates).*  **SQL是无序的，可以有重复元素**



*数据库模型如下：*

<img src="/Users/hannahzhang/Desktop/截屏2021-01-23下午8.12.55.png" alt="截屏2021-01-23下午8.12.55" style="zoom: 33%;" />

### Aggregates（聚合函数）

返回select语句执行得到的结果の函数，输入多个Tuple，输出单个结果

==**聚合函数只能出现在select子句中作输出列**==

从一堆元组返回单个值得函数：

→ `AVG(col)`→ 返回所有值の平均值

→ `MIN(col)`→ 返回最小の行值

→ `MAX(col)`→ 返回最大の行值

→ `SUM(col)`→ 返回所有行值の总和

→ `COUNT(col)`→ 返回行数

 

e.g 数出student表中login字段中以@cs结尾的学生数量

```sql
SELECT COUNT(login) AS cnt
	FROM student WHERE login LIKE '%@cs'
```

*Tuple数与login无关，改写如下*

```sql
SELECT COUNT(*) AS cnt
	FROM student WHERE login LIKE '%@cs'
```

**'*'代表Tuple中所有属性**

**每数一个Tuple，Tuple的数量就+1，代码改写如下：**

```sql
SELECT COUNT(1) AS cnt
	FROM student WHERE login LIKE '%@cs'
```



#### Mutiple Aggregates

Get the number of students and their average GPA that have a "@cs" login.

```sql
SELECT AVG(gpa), COUNT(sid)
	FROM student WHERE login LIKE '%@cs'
```



#### Distinct Aggregates 

COUNT, SUM, AVG support **DISTINCT**

student表中，login字段中以"@cs"结尾的不重复的学生的个数

```sql
SELECT COUNT(DISTINCT login)
	FROM student WHERE login LIKE '%@cs'
```



#### Group By

将元组投影到子集中并针对每个子集计算聚合

==**SELECT输出子句中的非聚合值必须出现在GROUP BY子句中**==

```sql
SELECT AVG(s.gpa), e.cid, s.name
	FROM enrolled AS e, student AS s
 WHERE e.sid = s.sid
 GROUP BY e.cid, s.name
```



#### HAVING

*（由于aggregation还没有完成，所以无法在WHERE语句中使用聚合函数结果过滤tuple）* 因此，需要使用HAVING语句根据聚合计算过滤结果。

```sql
SELECT AVG(s.gpa) AS avg_gpa, e.cid
	FROM enrolled AS e, student AS s
 WHERE e.sid = s.sid
 GROUP BY e.cid
 HAVING avg_gpa > 3.9
```

### String Operations—字符串处理

==String相关函数可以出现在查询中的任何地方==



#### 字符串匹配

`LIKE`用于字符串匹配。

字符串匹配运算符:

→ `'%'` 匹配任意长度的字符串（包括空字符串）

→`'_'` 仅仅匹配一个字符

e.g.

查询某学生名字的前五个字母

```sql
SELECT SUBSTRING(name, 0, 5) AS abbrv_name
	FROM student WHERE sid = 53688
```

找出学生名字转化成大写后开头是KAN的数据

```sql
SELECT * FROM student AS s
	FROM UPPER(e.name) LIKE 'KAN%'
```



#### 字符串拼接

SQL-92:

```sql
SELECT name FROM student 
	WHERE login = LOWER(name) || '@cs'
```

MSSQL:

```sql
SELECT name FROM student
	WHERE login = LOWER(name) + '@cs'
```

MySQL

```sql
SELECT name FROM student 
	WHERE login = CONCAT(LOWER(name), '@cs')
```

*MySQL还可以直接拼接*：

```sql
SELECT 'hello' ' world'
```



#### DATE/TIME OPERATIONS

在SQL中一般支持三种数据类型。

- **date：**日历日期，包括年（四位），月和日。
- **time：**一天中的时间，包括小时，分和秒。可以用变量**time(p)**来表示秒的小数点后的数字位数（默认是0）。 通过制定 **time with timezone,**还可以把时区信息连同时间一起存储。
- **timestamp: date 和 time**的组合。 可以用变量timestamp(p)来表示秒的小数点后的数字位数（这里默认值为6）。如果指定with timezone，则时区信息也会被存储

日期和时间类型的值可按如下方式说明：

+ **date：‘2018-01-17’**

+ **time：‘10:14:00’**

+ **timestamp：‘2018-01-17 10:14:00.45’**

*demo：get the # of days since the beginning of the year:*

```sql
SELECT NOW();
```

（返回时间戳'2018-08-29 12:30:31'） 

```sql
SELECT EXTRACT(DAY FROM DATE('2018-08-29'));
```

（返回数值29）

```sql
SELECT DATE('2018-08-29') - DATE('2018-01-01') AS days
```

Postgres: days = 240

MySQL: days = 728

**将日期转换成UNIX中的时间戳**

```sql
SELECT ROUND(UNIX_TIMESTAMP((DATE('2018-08-29'))-UNIX_TIMESTAMP(DATE('2018-01-01')))/(60*60*24),0) AS days
//ROUND 表示四舍五入，这里四舍五入到整数
```

MySQL: days = 240

**MySQL中`DAYDIFF`函数也能起到相同的作用**

```sql
SELECT DAYDIFF(DATE('2018-08-29') - DATE('2018-01-01')) AS days
```

MySQL: days = 240



### Output Redirection

将查询结果存储在另一个表中：

+ 尚未定义表

+ 已存在的表格，且具有相同的列数



SQL-92中：使用`INTO`关键字，执行过程中自动创建table储存结果

```sql
SELECT DISTINCT cid INTO CourseIds
	FROM enrolled
```

MySQL中：使用`CREAT TABLE`，在其内部定义SELECT语句

```sql
CREATE TABLE CourseIds (
SELECT DISTINCT cid FROM enrolled);
```

将结果插入已有table中：

```sql
INSERT INTO CourseIds
(SELECT DISTINCT cid FROM enrolled);
```

### Output Control

+ **ORDER BY <column*> [ASC|DESC]**

  通过一个或多个列中的值对输出元组进行排序。
  ```sql
  SELECT sid, grade FROM enrolled
		WHERE cid = '15-721'
		ORDER BY grade
	```

	==默认升序排列==，**降序排列**表示`DESC`，**升序排列**表示为`ASC`

	```sql
	SELECT sdi, grade FROM enrolled
		WHERE cid = '15-721'
		ORDER BY grade DESC, sid ASC
	```

+ **LIMIT <count> [offset]**

  +  限制输出元组的数量
  + 可以设置一个偏移
  
  e.g. 只返回10条记录
  
  ```sql
  SELECT sid, name FROM student
  	WHERE login LIKE '%@cs'
  	LIMIT 10
  ```
  
   ```sql
  SELECT sid, name FROM student
  	WHERE login LIKE '%@cs'
  	LIMIT 10 OFFSET 20
   ```
  



### NESTED QUERIES（嵌套查询）

包含其他查询的查询通常难以优化，内部查询可以（几乎）出现在查询中的任何位置。

e.g.“Get the names of students that are enrolled in ’15-445’.”

```sql
SELECT name FROM student 
	WHERE sid IN (
    SELECT sid FROM enrolled
  		WHERE cid = '15-445'
  )
```

==先查询student表==

#### 操作符

+ `ALL`→ 必须满足子查询中所有行的表达式

+ `ANY`→ 在子查询中必须满足至少一行的表达式。

+ `IN`→ 类似于 ‘=ANY()’ .

+ `EXISTS`→ 至少返回一行

```sql
SELECT name FROM student 
	WHERE sid = ANY (
    SELECT sid FROM enrolled
  		WHERE cid = '15-445'
  )
```

==**内嵌查询可以放在SELECT语句的输出部分处：**==

```sql
SELECT (SELECT name FROM student AS S
       WHERE S.sid = E.sid) AS sname
	FROM enrolled AS E
 WHERE cid = '15-445'
```

==先查询enrolled表==

e.g.*Find student record with the highest id that is enrolled in at least one course.*

```sql
SELECT sid, name FROM student
	WHERE sid => ALL(
  	SELECT sid FROM enrolled
)
```

```sql
SELECT sid, name FROM student
	WHERE sid IN (
  	SELECT MAX(sid) FROM enrolled
)
```

```sql
SELECT sid, name FROM student
	WHERE sid IN (
 		SELECT sid FROM enrolled
  		ORDER BY sid DESC LIMIT 1
)
```

e.g.*Find all courses that has no students enrolled in it.*

```sql
SELECT * FROM course 
	WHERE NOT EXISTS(
  	SELECT * FROM enrolled
    	WHERE course.cid = enrolled.cid
)
```

### WINDOW FUNCTIONS（窗口函数）

对一组相关的元组执行“滑动”计算。
像聚合一样，但元组不会分组为单个输出元组。

```sql
SELECT … FUNC-NAME(...) OVER (...)
	FROM tableName
```
`FUNC-NAME` : 汇总功能 、特殊功能
`OVER`: 切分数据

+ 汇总功能：aggregates

+ 特殊功能：

  + `ROW_NUMBER()`→ `ROW_NUMBER()`函数作用就是将`select`查询到的数据进行排序，每一条数据加一个序号，它不能用做于学生成绩的排名，一般多用于分页查询， 比如查询前10个，查询10-100个学生。

    ```sql
    SELECT *, ROW_NUMBER()OVER() AS row_num
    	FROM enrolled
    ```

  + `RANK()`→ `RANK()`函数，排名函数，可以对某一个字段进行排名，这里为什么和`ROW_NUMBER()`不一样呢，`ROW_NUMBER()`是排序，当存在相同成绩的学生时，`ROW_NUMBER()`会依次进行排序，他们序号不同，而`Rank()`则不一样出现相同的，他们的排名是一样的。

+ `OVER`：关键字指定在计算窗口函数时如何将元组组合在一起。 

  + 使用`PARTITION BY`指定组
  
    ```sql
    SELECT *, ROW_NUMBER()OVER(PARTITION BY cid) AS row_num
    FROM enrolled
   ORDER BY cid
    ```
    
   + 使用`order`排序
    
     ```sql
     SELECT *,
     	ROW_NUMBER()OVER(ORDER BY cid)
      FROM enrolled 
     ORDER BY cid
     ```
     
   + e.g
  
     Find the student with the highest grade for each course.
  
      ```sql
     SELECT * FROM (
       SELECT *, 
       			RANK()OVER(PARTITION BY cid
                                ORDER BY grade ASC)
             AS rank
          FROM enrolled) AS ranking
       WHERE ranking.rank = 1   
      ```
  
     
### COMMON TABLE EXPRESSIONS（公用表表达式）

提供一种编写用于较大查询的辅助语句的方法。

→可以将其视为仅用于一个查询的临时表。 嵌套查询和视图的替代方法。



```sql
WITH cteName AS (
  SELECT 1
)
SELECT * FROM cteName
```

可以将输出列绑定到AS关键字之前的名称。

#### CTE的递归

==Adding the RECURSIVE keyword after WITH allows a CTE to reference itself.==

Example: Print the sequence of numbers from 1 to 10.

```sql
WITH RECURSIVE cteSourse (counter) AS (
	(SELECT 1)
  UNION // 甚至可以不写union
  (SELECT counter + 1 FROM cteSourse
  WHERE counter < 10)
)
SELECT * FROM cteSourse
```

`UNION` 操作符用于合并两个或多个 SELECT 语句的结果集。

默认地，`UNION` 操作符选取不同的值，如果允许重复的值，使用`UNION ALL`
















