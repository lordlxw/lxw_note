## 一、基本概念

### 1、模式

> 数据库三级模式：模式、内模式和外模式
>
> 模式：是所有用户的公共数据视图，一个数据库只有一个模式。处于三级结构的中间层。
>
> 外模式：用户模式
>
> 当您使用 EF Core 进行数据写入时，实际上是直接向 MySQL 数据库的表中插入数据，不涉及到外模式的概念。主要用的是概念模式，并处理与数据库的“内模式”之间的映射和交互。
>
> 内模式：描述了数据库的实际物理存储和实现，包括数据存储格式、索引和数据分布等细节。是数据在数据库内部的表示方式，一个数据库只有一个内模式。
>
> 
>
> 数据库中**只有一个模式和一个内模式**



### 2、Mysql列约束

> PK、NN、UQ、BIN、UN、ZF、AI、Default
>
> 1. **PK：Primary Key（主键）**
>
>    - 表示该列是表的主键，用于唯一标识表中的每一行数据。主键是一个唯一且非空的标识符，每个表只能有一个主键。主键可以由单个列或多个列组成（复合主键）。
>
> 2. **NN：Not Null（非空约束）**
>
>    - 表示该列不允许为空，也就是说，该列必须包含有效的值，而不是 NULL。
>
> 3. **UQ：Unique（唯一约束）**
>
>    - 表示该列的值在整个表中必须是唯一的，不允许有重复的值。唯一约束允许一个或多个列具有唯一性。
>
> 4. **BIN：Binary（二进制）**
>
>    - 表示该列的数据类型是二进制类型，通常用于存储二进制数据，如图片、音频、视频等。
>
> 5. **UN：Unsigned（无符号）**
>
>    - 用于数值类型的列，表示该列的值不能为负数，只能是非负整数。如果将一个列声明为 `UNSIGNED`，则该列的取值范围将从0开始，而不是包含负数。
>
> 6. **ZF：Zero Fill（零填充）**
>
>    - 用于数值类型的列，在该列中的数字值前面使用零填充，以达到指定的位数。比如，如果一个列定义为 `INT(4) ZF`，那么存储在该列中的数值将被填充为固定的4位数，不足的部分用零填充。
>
> 7. **AI：Auto Increment（自动增量）**
>
>    - 表示该列是一个自动增量列，通常用于主键列。自动增量列的值会自动递增，每次插入一行数据时会自动分配一个新的唯一值。一般情况下，自动增量列会与主键一起使用，确保每行数据都有唯一的标识符。
>
>    这些约束简写可以在创建表时用于定义表中的列的属性，以确保数据的完整性、唯一性和约束条件。
>
> 



### 3、Mysql自带的系统级别数据库

> 1. **information_schema：** `information_schema` 数据库包含了 MySQL 数据库服务器的元数据信息，例如数据库、表、列、索引等的元数据。您可以查询 `information_schema` 数据库来获取数据库结构信息。
> 2. **performance_schema：** `performance_schema` 数据库是 MySQL 5.5.3 及以上版本引入的，它提供了关于数据库性能和资源消耗的监控信息。您可以使用 `performance_schema` 数据库来进行性能调优和分析。
> 3. **sys：** `sys` 数据库是 MySQL 8.0 及以上版本引入的，它是 `performance_schema` 数据库的补充，提供了更易于理解和使用的性能监控信息。`sys` 数据库中的视图和函数使性能数据更加可读和直观。
>
> 这三个数据库一定不能删除，一旦删除任意一个，MYSQL将不能正常工作



### 4、数据库常用语句

> 1. 创建数据库——Create Database 数据库名 IF NOT EXISTS Character Set=Gbk
> 2. 修改数据库（修改的是数据库参数）——Alter Databse 数据库名 Character Set=“XXX” Collater=“XXX” 
> 3. 删除数据库——Drop Databse [IF EXISTS] 数据库名
> 4. 查看命名匹配的数据库——SHOW DATABASES LIKE 'test00%';  %表示任意多个字符,_表示一个字符（中英文都是一样的，没有区分）





### 5、存储引擎及数据类型

> 存储引擎有什么用：
>
> 使用存储引擎可以加快查询的速度，并且每一种引擎都存在不同的含义，Mysql的数据类型是数据的一种属性，可以决定数据的存储格式、有效范围和相应的限制。
>
> 存储引擎的核心：
>
> 如何存储数据、如何为存储的数据建立索引、如何更新和查询数据



#### 存储引擎相关命令

> 显示Mysql中支持的存储引擎——Show Engines;  更高级的展示 + \G
>
> 查询默认的存储引擎——Show Variables like 'storage_engine%'
>
> 查看某个表的存储引擎——SHOW TABLE STATUS LIKE 'users';





#### InnoDB

> Mysql上第一个提供外键约束的表引擎，有对事务处理的能力。
>
> 特点：自动增长列值不能为空，且值必须唯一。自增列必须为主键
>
> 优点：提供良好的事务管理、崩溃修复能力和并发控制。
>
> 缺点：读写率稍差，占用的数据空间相对较大。
>
> 适用场景：更新密集的表，Mysql中唯一支持事务（金融软件必须），自动灾难修复



#### MyISAM

> 基于ISAM发展起来的
>
> MyISAM存储引擎的表存储成三个文件，文件名和表明相同，扩展名有：frm、MYD和MYI
>
> frm：存储表结构
>
> MYD：存储数据，MYData
>
> MYI：存储索引，MYIndex
>
> 优点：占用空间小，处理速度快
>
> 缺点：不支持事务完整性和并发性



#### Memory

> 比较特殊，使用存储在内存中的内容来创建表，而且所有数据也都放在内存中。
>
> 默认使用Hash索引、其速度要比使用B树（Btree）索引快。
>
> 缺点：数据存储在内存上，一旦内存出现异常就会影响到数据完整性。重启或关机，数据会消失。





> 注意！！！
>
> **存储引擎是和表绑定的，而不是和数据库绑定的。**
>
> 在 MySQL 中，每个表都可以使用不同的存储引擎。这意味着同一个数据库中的不同表可以选择不同的存储引擎来存储数据，根据具体的需求选择最适合的存储引擎。



### 6、Mysql数据类型

> MySQL 支持多种数据类型，以下是常见的 MySQL 数据类型：
>
> **整数类型：**
> 1. TINYINT
> 2. SMALLINT
> 3. MEDIUMINT
> 4. INT / INTEGER
> 5. BIGINT
>
> **小数类型：**
>
> 6. FLOAT
> 7. DOUBLE
> 8. DECIMAL / NUMERIC
>
> **日期和时间类型：**
> 9. DATE
> 10. TIME
> 11. DATETIME
> 12. TIMESTAMP
> 13. YEAR
>
> **字符串类型：**
> 14. CHAR——占用的存储空间固定，与指定的长度有关。例如，定义 `CHAR(10)` 的列，无论实际存储的字符串长度是多少，都会占用 10 个字符的存储空间
>
> 15. VARCHAR——占用的存储空间是实际存储的内容长度加上一个额外的字节用于存储长度信息。例如，存储一个 10 字符长度的字符串，实际存储空间可能是 11 字节（10 字符内容 + 1 字节长度信息）。
>
>     1. **性能：**
>        - 由于 `CHAR` 类型是固定长度，它的存储和检索速度通常比 `VARCHAR` 类型更快，因为 MySQL 不需要解析存储的长度信息。
>        - `VARCHAR` 类型在存储和检索时可能需要稍微多一些的处理开销，因为需要读取长度信息。
>     2. **使用场景：**
>        - `CHAR` 类型适合用于存储固定长度的字符数据，例如国家代码、性别代码等。
>        - `VARCHAR` 类型适合用于存储可变长度的字符数据，例如用户名、地址等，长度不固定的字段。
>
> 16. BINARY
>
> 17. VARBINARY
>
> 18. BLOB
>
> 19. TEXT
>
>     Text类型适合存储长文本，而BLOB类型适合存储二进制数据（如文本、声音、图像等)
>
>     区分大小写用BLOB，不区分用TEXT
>
> 20. ENUM
>
> 21. SET
>
> **其他类型：**
>
> 22. BOOLEAN / BOOL
> 23. JSON
>
> 这些数据类型可以用于创建表中的列，每个数据类型都有其特定的用途和范围。根据数据的性质和存储需求，选择合适的数据类型是数据库设计的重要一环。同时，不同的数据类型还有不同的存储空间和效率开销，所以在使用时需要仔细考虑表的设计和数据的存储要求。





### 7、数据表相关命令

> 查看表结构——Show Columns; //在当前表下
>
> 查看表结构——SHOW COLUMNS FROM TB1 FROM DB;//具体绝对路径
>
> 使用Describe查看——Describe 数据表名;   or   Describe 数据表名 列名
>
> 修改表结构——ALTER TABLE 数据表名 操作
>
> 例1：修改字段名
>
> ALTER TABLE DB_TEST(库名).userA(表名) CHANGE COLUMN user userName VARCHAR(30) NOT DEFAULT NULL;  //将DB_TEST数据库中的userA表中的user列改为userName
>
> 例2：删除字段名
>
> ALTER TABLE_A DROP email //从表A中删除email字段
>
> 例3：修改表名：
>
> ALTER TABLE tb_A RENAME AS tb_B
>
> 例4：创建表备份
>
> CREATE tb_back Like TABLE_C；//创建一份数据表TABLE_C的备份tb_back；
>
> 



### 8、基础运算符

> In——判断数据是否存在于某个集合中，X1 In (值1、值2、...、值N)
>
> Like——匹配字符串，X1 Like S1，若x1与s1匹配，返回1，否则返回0
>
> REGEXP——正则匹配 X1 REGEXP '匹配方式'，满足返回1，否则返回0



### 9、存储过程语法及例子

> 有IF、CASE（WHEN）、WHILE（CONDITION）、LOOP（LEAVE退出循环）、ITERATE和LEAVE语句，进行流程控制。
>
> 注意：存储过程不支持for循环
>
> ```例子：获取指定日期区间的用户工作时间
> CREATE DEFINER=`root`@`%` PROCEDURE `GetWorkHoursBetweenDates`(IN start_date DATE, IN end_date DATE)
> BEGIN
>  -- 如果未提供 end_date 参数，默认使用当前日期
>  IF end_date IS NULL THEN
>      SET end_date = CURDATE();
>  END IF;
> 
>  -- 查询每个用户在指定日期范围内每天的工作时长
>  SELECT
>      a.UserName,
>      d.Date AS '日期',
>      ROUND(SUM(d.Time) / 3600, 2) AS '工作时长（小时）'
>  FROM
>      dailylog d
>  JOIN
>      currentusermodels a ON a.ID = d.UserID
>  WHERE
>      d.Date BETWEEN start_date AND end_date
>  GROUP BY
>      a.UserName,
>      d.Date
>  ORDER BY
>      a.UserName;
> END
> ```
>
> 调用存储过程：CALL 存储过程名。
>
> **存储过程和存储函数是在数据库服务器端进行存储可执行的，可以减少客户端和服务器端的数据传输。**



### 10、表数据有关的操作

> Delete——删除数据（逐行删除）
>
> Truncate——删除数据（从表中快速删除所有数据，直接清空表），使用这个会将auto_increment计数器重新设置为初始值。Truncate使用的系统和事务日志资源少。



### *11、查询语句进阶

#### 内连接=等同连接（求交集）

> SELECT students.name AS student_name, courses.name AS course_name FROM students JOIN courses ON students.course_id = courses.course_id;
>
> 等同于
>
> SELECT students.name AS student_name, courses.name AS course_name FROM students,courses where students.course_id = courses.course_id
>
> 第一个查询使用了显式的 INNER JOIN 语法，它是较新的 ANSI SQL 标准中的写法，用于进行表的等同连接。
>
> 第二个查询使用了传统的隐式连接写法，即在 FROM 子句中列出多个表，并在 WHERE 子句中指定连接条件来连接这些表。



#### 全连接=两个表的笛卡尔积

> SELECT students.name AS student_name, courses.name AS course_name
> FROM students
> FULL JOIN courses ON students.course_id = courses.course_id;
>
> 全连接（Full Join）： 全连接会返回连接表中所有的行，并将不满足连接条件的位置填充为 NULL 值。在这个例子中，我们使用全连接来获取所有学生以及他们选修的课程，即使有些学生没有选修课程，或有些课程没有学生选择
>
> ````markdown
> 全连接（Full Join）确实涉及到笛卡尔积的概念，但它并不完全等同于笛卡尔积。
> 
> 首先，让我们回顾一下笛卡尔积是什么：
> 
> **笛卡尔积**：
> 在数学中，给定两个集合 A 和 B，它们的笛卡尔积（Cartesian Product）是一个集合，其中的元素由所有可能的有序对 (a, b) 组成，其中 a 属于 A，b 属于 B。换句话说，笛卡尔积是将一个集合中的每个元素与另一个集合中的每个元素进行组合。
> 
> 例如，如果集合 A = {1, 2}，集合 B = {a, b}，则它们的笛卡尔积为：{(1, a), (1, b), (2, a), (2, b)}。
> 
> **全连接**：
> 在 SQL 中，全连接是一种连接类型，它返回连接表中所有的行，并将不满足连接条件的位置填充为 NULL 值。这就意味着，无论是否存在匹配条件，全连接都会返回两个连接表中的所有行。
> 
> 如果我们有两个表 A 和 B，通过全连接将它们连接在一起，那么结果将包含 A 和 B 中所有可能的组合，而不仅仅是 A 和 B 共有的数据。
> 
> 举个例子：
> 
> 假设我们有两个表 A 和 B：
> 
> 表 A：
> 
> | ID | Name_A |
> |----|--------|
> | 1  | Alice  |
> | 2  | Bob    |
> 
> 表 B：
> 
> | ID | Name_B |
> |----|--------|
> | 1  | Apple  |
> | 3  | Orange |
> 
> 现在我们执行全连接操作：
> 
> ```sql
> SELECT A.ID, A.Name_A, B.ID, B.Name_B
> FROM A
> FULL JOIN B ON A.ID = B.ID;
> ```
> 
> 结果：
> 
> | ID | Name_A | ID | Name_B |
> |----|--------|----|--------|
> | 1  | Alice  | 1  | Apple  |
> | 2  | Bob    | NULL | NULL |
> | NULL | NULL | 3  | Orange |
> 
> 在上面的结果中，全连接返回了 A 和 B 表的所有行，其中：
> - 第一行表示 ID 为 1 的行在两个表中都有匹配，所以返回了该行的数据。
> - 第二行表示 ID 为 2 的行只在表 A 中有匹配，而在表 B 中没有匹配，所以表 B 部分的列被填充为 NULL。
> - 第三行表示 ID 为 3 的行只在表 B 中有匹配，而在表 A 中没有匹配，所以表 A 部分的列被填充为 NULL。
> 
> 总结：
> 全连接涉及到笛卡尔积的概念，因为它返回了连接表中所有可能的组合。但全连接和笛卡尔积并不完全等同，全连接在实际使用中会将不满足连接条件的位置填充为 NULL 值，从而保留了两个连接表中的所有行。而笛卡尔积仅仅是组合了两个集合中的元素，并不关心是否有其他连接条件。
> ````
>
> 



#### 左连接

> LEFT JOIN 关键字从左表（Websites）返回所有的行，即使右表（access_log）中没有匹配。



#### 右连接

> RIGHT JOIN 关键字从右表（table2）返回所有的行，即使左表（table1）中没有匹配。如果左表中没有匹配，则结果为 NULL。
>
> RIGHT JOIN 关键字从右表（access_log）返回所有的行，即使左表（Websites）中没有匹配。



#### GroupBy分组查询

> 通过关键字Group By可以将数据划分到不同组中，实现对记录进行分组查询。在查询时，所查询的列必须包含分组的列中，目的是使查询的数据没有矛盾。
>
> ==要求GROUP BY子句中的列必须是SELECT中列出的列或者是聚合函数，否则会报错。==





#### 正则匹配查询

> 匹配以指定字符开头和结束的记录——Select * from computer_table where name REGEXP '^L..y$';
>
> //查找name字段中以L开头，以y结束的，中间包含两个字符的学生成绩。
>
> *代表任意多个字符，注意也可以是0个
>
> +，比如a+c，代表字母c前面至少有一个字母a

> 







### 12、常用函数

#### 日期和时间函数：

> - NOW()：返回当前日期和时间。
> - CURDATE() 或 CURRENT_DATE()：返回当前日期。 比如2023-07-08
> - CURTIME() 或 CURRENT_TIME()：返回当前时间。
> - DATE()：提取日期部分。
> - TIME()：提取时间部分。
> - DATE_FORMAT()：格式化日期和时间。
> - DATEDIFF('日期1','日期2');//计算相隔的天数





#### 系统信息函数

> VERSION()—— 返回版本号
>
> CONNECTION_ID()—— 返回服务器连接数，即迄今为止mysql服务的连接次数
>
> SHOW VARIABLES LIKE 'max_connections';——查看mysql最大连接数
>
> SHOW PROCESSLIST;——查看当前活动的连接数和连接 ID
>
> select user(),session_user(),current_user();——查询当前用户用户名
>
> select charset('aa'),collation('bb');——获取字符串的字符集和排序方式
>
> 

#### 加密函数

> select MD5('lxw');//使用MD5()函数对字符串lxw进行加密
>
> SELECT SHA('Hello World');
>
> 



#### 格式化函数

> Format(x,n)——将数字X进行格式化，保留小数点后n位







## 二、进阶部分

### 1、索引概述

> 数据库索引是一种数据结构，用于加快数据库表中数据的检索速度。它类似于书籍的索引，可以帮助数据库管理系统更快地找到符合特定条件的数据行，从而提高查询效率。

> 通过创建适当的索引，数据库可以避免全表扫描，而是通过索引快速定位到满足条件的数据行，从而提高数据库查询性能。当数据库表中的数据量较大时，索引的作用尤为显著，可以大幅减少查询时间，提高数据库的响应速度。然而，索引的创建也需要谨慎考虑，因为索引的存在会增加存储空间和维护成本。



> 创建普通索引：
>
> create table_A(id int(11) ,index(id));
>
> 创建唯一性索引：
>
> create table_A(id int(11) ,name varchar(50),UNIQUE index ADDRESS(id ASC));//创建一个table_A的表并指定该表的id字段上建立唯一索引。
>
> 在已建立的数据表中创建索引：
>
> create [UNIQUE|FULLTEXT|SPATIAL] INDEX index_name on table_name(属性)
>
> 创建单列索引：
>
> create INDEX 索引名 ON 数据表名(字段名（长度))//设置字段名称长度，可以优化查询速度，提高查询效率
>
> 创建多列索引：
>
> create INDEX 索引名 ON 数据表名（字段名1，字段名2,...）
>
> 删除索引：
>
> DROP INDEX index_name ON 表名;





> 当选择使用哪种类型的索引时，需要根据数据库表的结构和查询需求来确定。不同类型的索引适用于不同的场景，下面是一些常见的使用场景以及适合的索引类型的示例：
>
> 1. 唯一索引：唯一索引一般添加在主键上
>    - 场景：需要确保某个列的值在表中是唯一的，如用户的身份证号或邮箱地址。
>    - 索引类型：唯一索引。
> 2. 主键索引：
>    - 场景：需要为表中的每一行指定一个唯一标识符，以便快速定位和识别每一行。
>    - 索引类型：主键索引。
> 3. 聚集索引：
>    - 场景：经常进行范围查询、按顺序查询或分页查询的表。
>    - 索引类型：聚集索引。
> 4. 非聚集索引：
>    - 场景：经常进行查找或条件过滤的表。
>    - 索引类型：非聚集索引。
> 5. 复合索引：
>    - 场景：需要优化联合查询或按多个列查询的表。
>    - 索引类型：复合索引。
> 6. 全文索引：
>    - 场景：需要在大文本字段中进行全文搜索的表，如新闻文章或产品描述。
>    - 索引类型：全文索引。
> 7. 空间索引：
>    - 场景：处理包含地理空间数据的表，如地图数据或位置信息。
>    - 索引类型：空间索引。MYSQL中只要MyISAM可以用





### 2、视图概述

> select Select_priv,Create_view_priv FROM mysql.user where user='root' //查看某个用户是否有创建视图、查询视图的权限。

> CREATE VIEW DailyWorkHoursView AS
> SELECT
>     a.UserName,
>     ROUND(SUM(Time) / 3600, 2) AS '当日有效工作时长（小时）',
>     CURRENT_DATE AS '日期'
> FROM
>     dailylog d
> JOIN
>     currentusermodels a ON a.ID = d.UserID
> WHERE
>     Date = CURDATE()
> GROUP BY
>     UserID
> ORDER BY ROUND(SUM(Time) / 3600, 2) DESC;
>
> 可以将这个存储过程改写为视图。视图是一个虚拟表，基于查询语句的结果生成数据。由于存储过程中的查询语句可以直接在视图中使用，所以将存储过程改写为视图是很简单的。
>
> 创建完视图后，你可以像查询表一样使用视图 DailyWorkHoursView 来获取当日有效工作时长的信息。



#### 视图和存储过程的区别

==视图只能select，而存储过程可以增删改查都行==

> 1. 数据表示和逻辑封装：
>    - 视图是用于表示数据的虚拟表，本身不存储数据，它是对基本表的一个抽象，用于简化查询和隐藏复杂逻辑。视图提供了一种数据的逻辑视图，可以将复杂的查询语句封装成一个简单的视图，方便查询和使用。
>    - 存储过程是一组预定义的 SQL 语句集合，用于在数据库服务器端执行复杂的逻辑操作。它不是用于表示数据，而是用于封装一系列操作或业务逻辑，可以接受参数并返回一个或多个结果集。
> 2. 查询灵活性：
>    - 视图返回的结果集是固定的，每次查询视图都会返回相同的结果。视图的查询逻辑是固定的，无法在查询时动态修改。
>    - 存储过程具有更大的灵活性，它可以根据传递的参数动态执行不同的逻辑。存储过程可以根据不同的参数值返回不同的结果，因此更适合处理一些动态需求或有条件分支的情况。
> 3. 调用方式：
>    - 视图可以像查询表一样直接查询，不需要调用特定的过程语法。
>    - 存储过程需要使用 CALL 或 EXECUTE 等过程调用语句来执行。
> 4. 用途：
>    - 视图适用于简化查询、隐藏复杂逻辑、提供数据的逻辑视图等场景。
>    - 存储过程适用于封装一系列操作、实现复杂的业务逻辑、提高数据库性能等场景。





### 3、高级应用

#### 1、触发器

> 由Insert、Update、Delete等事件来触发某些特定动作。
>
> 创建Mysql触发器：
>
> CREATE TRIGGER 触发器名 BEFORE|AFTER 触发事件 ON 表名 FOR EACH ROW 执行语句
>
> 查看triggers表中触发器信息
>
>  select * from information_schema.triggers;
>
> 删除触发器：
>
> DROP TRIGGER 触发器名



#### 2、事务

> START TRANSACTION;
>
> -- 执行一系列数据库操作
>
> -- 如果所有操作都成功，提交事务
> COMMIT;
>
> -- 如果发生错误或不满足某种条件，回滚事务
> ROLLBACK;

> 事务的孤立级别（Isolation Level）是数据库管理系统中一个重要的概念，用于控制并发事务的隔离程度。在多用户并发访问数据库的环境中，可能会出现多个事务同时对数据库进行读取和修改的情况。事务的孤立级别规定了一个事务对于其他事务的可见性和影响范围，以确保数据的一致性和隔离性。
>
> 常见的四个事务孤立级别，从低到高依次为：
>
> 1. 读未提交（Read Uncommitted）：最低级别的孤立性，一个事务可以读取另一个事务尚未提交的修改，可能导致脏读（Dirty Read）。
>
> 2. 读已提交（Read Committed）：保证一个事务只能读取另一个事务已经提交的修改，避免了脏读，但可能会出现不可重复读（Non-Repeatable Read）。
>
> 3. 可重复读（Repeatable Read）：确保在一个事务中多次读取同一数据，结果保持一致。其他事务对该数据的修改在当前事务中不可见，避免了不可重复读，但可能出现幻读（Phantom Read）。
>
> 4. 串行化（Serializable）：最高级别的孤立性，确保一个事务的执行完全不受其他事务的影响，同时阻止其他事务对事务涉及的数据进行读取和修改。
>
> 选择合适的事务孤立级别要根据具体业务需求和数据一致性的要求。较低的孤立级别可能会提高并发性能，但可能导致数据不一致和异常，而较高的孤立级别则会降低并发性能，但可以保证数据的一致性和隔离性。在设计数据库应用时，需要仔细考虑孤立级别，避免出现意外的数据访问和修改问题。

> 在 MySQL 中，事务的默认孤立级别是可重复读（Repeatable Read）。这意味着当你开始一个事务时，默认情况下，该事务将使用可重复读的孤立级别。
>
> 在可重复读的孤立级别下，一个事务可以多次读取同一数据，保持一致。其他事务对该数据的修改在当前事务中不可见，从而避免了不可重复读。但在可重复读的孤立级别下，仍然可能出现幻读，即在一个事务中多次查询同一范围的数据，但在其他事务插入新数据时，这些新数据可能会在当前事务中出现，导致幻读。
>
> Mysql中可以通过TRANSACTION ISOLATION LEVEL变量来修改事务孤立级。



##### 幻读

> 当使用可重复读（Repeatable Read）事务隔离级别时，可能会遇到幻读的情况。假设有一个简单的表格 `employees` 存储员工的信息，表结构如下：
>
> ```
> CREATE TABLE employees (
>     id INT PRIMARY KEY AUTO_INCREMENT,
>     name VARCHAR(50),
>     salary INT
> );
> ```
>
> 现在，我们有两个事务同时进行操作：
>
> **事务1：**
>
> ```sql
> START TRANSACTION;
> SELECT * FROM employees WHERE salary > 3000;
> -- 假设此时查询到两条记录
> -- 假设在这段时间内，事务2插入了新的记录
> -- 例如：INSERT INTO employees (name, salary) VALUES ('Alice', 4000);
> SELECT * FROM employees WHERE salary > 3000;
> -- 此时查询到三条记录，出现了幻读
> COMMIT;
> ```
>
> **事务2：**
>
> ```sql
> START TRANSACTION;
> INSERT INTO employees (name, salary) VALUES ('Bob', 3500);
> -- 在事务1的第二次查询之前，事务2插入了一条新的记录
> COMMIT;
> ```
>
> 在上面的例子中，事务1 在可重复读的隔离级别下，第一次查询 `salary > 3000` 的结果是两条记录，但在第二次查询之前，事务2 插入了一条新的记录，导致事务1 第二次查询的结果变成了三条记录。这就是幻读，因为在事务1 内部，同样的查询语句返回了不同的结果。
>
> 为了避免幻读，可以将事务隔离级别设置为读已提交（Read Committed）或使用其他机制如范围锁定等。这样，事务1 在第二次查询时，会看到事务2 插入的新记录并阻止幻读的发生。



#### 3、事件

> 就是定时器





#### 4、备份

> mysqldump命令可以将数据库中的数据备份成一个文本文件。
>
> ```数据库备份
> `mysqldump` 是 MySQL 数据库中用于备份数据的命令行工具。它通过生成一个 SQL 脚本文件，将数据库中的数据和结构导出到该文件中，从而实现备份数据库的功能。`mysqldump` 的工作原理如下：
> 
> 1. 连接到数据库：`mysqldump` 首先通过指定的用户名和密码，连接到要备份的 MySQL 数据库。
> 
> 2. 获取数据库结构信息：一旦连接成功，`mysqldump` 开始获取数据库的结构信息，包括表的定义、字段、索引、触发器等。
> 
> 3. 生成 SQL 脚本：根据数据库的结构信息，`mysqldump` 生成一个 SQL 脚本文件。该脚本包含了一系列 SQL 语句，用于创建数据库、创建表格、插入数据等。
> 
> 4. 导出数据：`mysqldump` 在生成 SQL 脚本时，同时将数据库中的数据也导出到脚本中。数据导出使用 INSERT 语句，将每个表的数据逐行写入脚本。
> 
> 5. 保存到文件：生成的 SQL 脚本会保存到指定的文件中，这样就完成了数据库的备份过程。
> 
> `mysqldump` 支持许多选项，可以通过选项来指定备份的内容和方式，例如：
> 
> - `--databases`：备份指定的数据库。
> - `--tables`：备份指定的表格。
> - `--ignore-table`：指定要忽略的表格，不备份其中的数据。
> - `--where`：指定备份数据的条件。
> - `--single-transaction`：在备份过程中使用事务，确保数据一致性。
> - 等等。
> 
> 使用 `mysqldump` 可以轻松备份数据库，并在需要时进行恢复。备份的 SQL 脚本文件可以用于恢复数据库到备份时的状态。注意，`mysqldump` 在备份大型数据库时可能会导致一定的性能影响，因为它需要查询和导出整个数据库的数据。
> ```
>
> 语法：mysqldump -u root -pPassword test004 app > BackupScent.sql
>
> 将一个数据库备份成一个后缀名为.sql的文件，备份一个数据库
>
> 备份多个数据库：
>
> mysqldump -u 用户名 -p --databases dbname1，dbname2 > BackUpSql.sql
>
> 备份所有数据库：
>
> mysqldump -u 用户名 -p --all -databases > BackupAllSql.sql



#### 5、迁移

> mysqldump -h name1 -u root -p123456 -all-databases | mysql -h host2 -u root -password=password2
>
> 通过管道符迁移





### 4、Mysql性能优化

> 数据库管理员可以使用SHOW STATUS语句查询Mysql数据库的性能！
>
> ```sql
> SHOW STATUS LIKE 'VALUE';
> ```
>
> value是几个常用的统计参数，具体如下：
>
> 1. Connections：连接Mysql服务器的次数
> 2. Uptime：Mysql服务器的上线时间
> 3. Slow_queries：慢查询的次数
> 4. Com_select：查询操作的次数
> 5. Com_insert：插入操作的次数
> 6. Com_delete：删除操作的次数
>
> 



#### 分析查询语句

> Explain + select 语句
>
> 返回字段及意义：
>
> 1. id：select的位置
> 2. table：表名
> 3. type：连接类型
> 4. possible_keys：为了提高查找速度所用的索引
> 5. key：实际使用的键
> 6. rows：mysql在相应表中返回查询结果所检验的行数
> 7. Extra：其他信息，设计Mysql如何处理查询



#### 优化方式

> 将字段很多的表分解成多个表
>
> 增加中间表
>
> 如果一下子插入大量数据，索引的存在会降低插入记录的速度，为解决这种情况，在插入记录时应该先禁用索引，等到记录插入完毕再开启索引。
>
> 禁用唯一性检查：SET UNIQUE_CHECKS = 0;// 开启是1
>
> 一次性insert多条数据比每次插入一条数据要快，因为减少了与数据库之间的连接操作。



#### 分析表、优化表

> ANALYZE TABLE 表名1，【表名2】
>
> OPTIMIZE TABLE 表名1，【表名2】 可以消除删除和更新造成的磁盘碎片，从而减少空间的浪费。执行过程中表会阻塞。





#### 高速缓存

> 当用户通过SELECT语句查询数据时，该操作将结果集保存到一个特殊的高级缓存中，从而实现查询操作。
>
> ```sql
> SHOW VARIABLES LIKE '%query_cache%' //检验高速缓存是否开启。
> ```
>
> 