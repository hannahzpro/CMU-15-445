# Lecture#4 Database Storage II

### Log Structured File Organization (日志结构的文件组)

DBMS只存储日志记录，而不是在页面中存储元组。

系统将日志记录追加到有关如何修改数据库的文件中：

→插入存储整个元组。

→删除将元组标记为已删除。

→更新仅包含已修改属性的增量。

![11](/Users/hannahzhang/Desktop/11.jpg)

提高读取效率：

+ 建立索引以使其跳至日志中的位置。
+ 定期压缩日志![20200508172834807](/Users/hannahzhang/Desktop/20200508172834807.png)

## Data Representation（数据表示）
元组本质上是字节序列。

DBMS的工作是将这些字节解释为属性类型和值。

DBMS的目录包含有关表的模式信息，系统使用这些表来确定元组的布局。


`INTEGER`/`BIGINT`/`SMALLINT`/`TINYINT`

→ C/C++ 原生数据类型

`FLOAT`/`REAL` vs. `NUMERIC`/`DECIMAL`

→ IEEE-754 浮点数标准/ 定点小数

`VARCHAR`/`VARBINARY`/`TEXT`/`BLOB`

→ Header中含有长度信息, 后面跟着字节数组。

`TIME`/`DATE`/`TIMESTAMP`

→ 自Unix时代以来的32/64位整数（微秒）

### 可变精度数

使用 C / C ++原生的数据类型的不精确，精度可变的数字类型。

→例如：`FLOAT`，`REAL` /`DOUBLE`

按照IEEE-754的规定直接存储。

通常比定点数快，但可能会有舍入误差。**硬件无法精确表示浮点数**



### 固定精度数

具有任意精度和小数位数的数值数据类型。 在无法接受舍入误差时使用。

→例如：`NUMERIC`，`DECIMAL`

通常以精确的，可变长度的二进制表示形式存储，并带有其他元数据。

→类似于`VARCHAR`，但不存储为字符串



#### POSTGRES: NUMERIC

实现类似于高精度加减法

![20200508172901853](/Users/hannahzhang/Desktop/20200508172901853.png)



### Large Values

需要保存的东西too large而不能放在一个page上时，有两种解决方案：

+ Overflow Page 

  大多数DBMS不允许元组超过单个页面的大小。

  为了存储大于页面的值，DBMS使用单独的溢出存储页面。

  **通过一个指针来指向保存了我们想要数据的overflow page**

  在实际使用中，不同数据库系统的overflow有不同的的规格：

  + Postgres：TOAST（> 2KB）

  + MySQL：溢出（> 1/2页大小）

  + SQL Server：溢出（>页面大小）

+ External Value Storage

  不会将该属性的数据保存在tuple内部，而是往里面存一个指针or文件路径，它们指向能早到该数据的本地磁盘，或者网络存储、外部设备等。

  **只可以读取，不能操作数据**

  视为BLOB类型。

  →Oracle：BFILE数据类型

  →Microsoft：FILESTREAM数据类型

  DBMS无法操纵外部文件的内容。

  →没有耐用性保护。

  →没有交易保护。

  

## System Catalogs（系统目录）

DBMS在其内部目录中存储有关数据库的元数据。

+ 表，列，索引，视图

+ 用户，权限

+ 内部统计

很多数据库系统都会将他们的Catalog用另一张表来保存

**内部会通过某种底层的方法来访问catalog**

+ 将对象抽象包装在元组周围。

+ 用于“引导”目录表的专用代码。

您可以查询DBMS的内部`INFORMATION_SCHEMA`目录以获取有关数据库的信息。

→ANSI标准的只读视图集，可提供有关数据库中所有表，视图，列和过程的信息

DBMS还具有非标准的快捷方式来检索此信息。



## Storage Models（储存模型）

关系模型没有指定我们必须将所有元组的属性一起存储在单个页面中。

对于某些workloads (工作情况)，这实际上可能不是最佳的布局

==两种workload==

### OLTP(联机事务处理)

思路：从外部拿到新数据后，将他们放入数据库系统中。

查询，读取/更新与数据库中**单个实体**有关的少量数据。

### OLAP(联机分析处理)
复杂查询读取跨多个实体的数据库的大部分内容。



![20200508173005130](/Users/hannahzhang/Desktop/20200508173005130.png)

**这种环境下，不会对数据进行更新**

HATP：OLTP和OLAP的结合





### N-ARY 存储模型(aka 行存储模型、NSM）

将单个Tuple中的所有属性去除，并将他们连续地储存在page中

（对于较大的可以使用overflow pages，但一般来说需要被对齐）

OLTP工作负载的理想选择，在这些工作负载中查询往往只对单个实体进行操作并插入大量工作负载。

![20200508173014907](/Users/hannahzhang/Desktop/20200508173014907.png)

有大量数据时，该储存模型不适用

优点：

+ 插入，更新，删除快速
+ 对于需要查询整个Tuple的操作，效率高

缺点：

+ 查找单个属性时，效率低

### DECOMPOSITION （aka 列存储模型、DSM）

DBMS在页面中连续存储所有元组的单个属性的值。

→也称为“列存储”



![20200508173027126](/Users/hannahzhang/Desktop/20200508173027126.png)

可以对重复的数据进行压缩

优点：

+ 在进行OLAP查询时，可以降低浪费的I/O的数量*（因为是只读数据）*
+ 更好的查询和数据压缩能力

缺点：

+ 插入，更新，删除慢

#### 元组ID

选择1：**固定长度的偏移**

→每个值的属性长度相同。

选择2：嵌入式元组ID

→每个值及其元组ID存储在列中

