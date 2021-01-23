# Lecture#1 Course Introduction and the Relational Model

## the Relational Model

#### 基本概念
+ Data model（模式）：数据库中全体数据的逻辑结构和特征的描述，只涉及型，不涉及具体值。
+ Schema（实例）：模式的一个具体值，同一个模式可以有多个实例。

#### 主要术语
+ Relation（关系）：对应一张表格
+ Tuple（元组）：表中的一行称为一个元组 **通常不能是数组或者嵌套对象**
+ Attribute（属性）：表中的一列
+ Domain（域）：属性的取值范围
+ Primary Key（主键）：能唯一标识一个元组的属性or属性组
+ Foreign Key（外键）：

#### DML (Data Manipulation Languages)
*also known as query language* 

+ **Procedural** - user specifies what data is required and how to get those data
+ Non-Procedural (or declarative) - user specifies what data is required without specifying how to get those data

#### Relational Algebra
*(based on  set algebra)*

+ SELECT ==σ~predicate~(R).==
  从满足选择条件的关系中选择一个元组(tuples)的子集
  → 充当过滤器，仅保留满足选择条件要求的元组
  → 可以使用连接词/析取词组合多个条件
  
+ PROJECTION ==$π_{A1,A2,. . . ,An}$(R).==
  用仅包含指定属性的元组生成一个关系
  → 可以重新排列属性的顺序
  → 可以操作这些值

+ UNION ==(R ∪ S).==
  生成一个包含只在一个或两个输入关系中出现过的关系

+ INSERTION ==(R ∩ S).==
  生成仅包含在两个关系中都出现的元组的关系

+ DIFFERENCE  ==(R - S).==
  生成一个由仅在第一个关系而未在第二个关系中出现的关系
  
+ PRODUCT ==(R x S).==
  **(CROSS JOIN)**

  生成所有可能的组合

  <img src="/Users/hannahzhang/Desktop/截屏2021-01-23下午5.32.54.png" alt="截屏2021-01-23下午5.32.54" style="zoom:50%;" />

+ JOIN ==(R ⋈ S).==
  **(NATURAL JOIN)**
  与difference的不同：Join中可能存在部分属性不同

#### OBSERVATION

  *A better approach is to say the result you want, and let the DBMS decide the steps it wants to take to compute the query. SQL will do exactly this, and it is the de facto standard for writing queries on relational model databases.*

数据库发生改变时，更改SQL代码改变查询方式即可，不需要更改数据库代码*(DBMS程序代码)*









