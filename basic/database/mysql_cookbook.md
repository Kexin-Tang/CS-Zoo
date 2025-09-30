# MySQL cookbook

本文主要用于记录学习MySQL的相关内容和各种语法操作，方便大家~~临时抱佛脚~~

## 三大范式

### 第一范式 -- 列不可再分

> 每个列都是不可分割的原子值。

比如说保存有地址为 "*中国湖北省武汉市洪山区华中科技大学韵苑宿舍*" 就不满足第一范式，因为该地址可以继续划分，应该保存为 "*国家*" "*省*" "*市*" "*区*" "*地点*" 这种字段。 

### 第二范式 -- 完全依赖主键

> 必须在满足第一范式的基础上，其他各个列都必须完全依赖于主键。如果出现不完全依赖，也只能是联合主键的情况。

比如一个表涉及到 `房间号 | 联系人 | 联系人电话 | 身份证` 就是不符合第二范式的，因为 *联系人电话* 与 *身份证* 这两个属性和 *房间号* 没有关联，如果一个人定了多个房间，那么这些数据就会无意义的重复。

所以需要拆表，分成`房间号 | 联系人`和`联系人 | 联系人电话 | 身份证`这样两个表，这两个表中的属性都是完全依赖于主键。

### 第三范式 -- 不能有传递

> 必须在满足第二范式的基础上，每个列都跟主键有直接关系而不是间接关系。

比如一个表涉及`学号 | 学生 | 院系 | 入学年份 | 院系主任`，其中 *学号* 是和 *入学年份* 以及 *院系* 直接相关的，但是 *学号* &rarr; *院系* &rarr; *院系主任*，所以 *院系主任* 就是间接关系，不满足第三范式。

---

## 数据库基本操作

* 查看数据库
```bash
show databases;
```

* 切换数据库
```bash
use [dbName];
```

* 查看表
```bash
show tables;
```

* 创建数据库
```bash
create database [dbName];
```

* 创建表
```bash
create TABLE [tableName](
    id int PRIMARY KEY,
    name VARCHAR(20) NOT NULL
);
```
> *var()与varchar()的区别在于var()是定常的,哪怕存储的字符串没有达到"()"中数字的上限,var()依然会占用空格来填充空间.而varchar()则是不定长的,没有达到"()"中的上限则会自动去掉后面的空格*

* 查询表的架构
```bash
desc [tableName];
```

---

## CRUD操作

* 插入数据
```bash
INSERT INTO [tableName](name, age) VALUES('tom', 5);
```
> *上述语句设定了 name=tom, age=5, 其他未设置的字段为默认值/Null*

* 查询数据
```bash
SELECT (name, age) FROM [tableName] WHERE [case];
```
> *上述语句根据 [case] 查询 [tableName] 表中的name和age字段，使用`*`来代表所有字段*

* 删除数据
```bash
DELETE FROM [tablesName] WHRER [case];
```

* 更新数据
```bash
UPDATE [tableName] SET (name='jerry', age=10) WHERE [case];
```

---

## 约束

### 主键约束

主键：独一无二且可以唯一确定一条信息的标记，且该标记不能为空。建表时使用`PRIMARY KEY`来设置。

* 单一主键
```bash
CREATE TABLE pet(
    id INT PRIMARY KEY,
    name VARCHAR(20)
);
```

* 联合主键
> 复合主键只要所有的字段都不是相同的情况下可以允许其中的字段重复
```bash
CREATE TABLE pet(
    id INT,
    name VARCHAR(20),
    age INT,
    PRIMARY KEY(id, name)
);
```

### 自增约束

一般会配合主键一起使用，即该字段会在未设置的情况下自动的进行设置，值会依次递增。使用关键字`AUTO_INCREMENT`。

### 外键约束

使用`FOREIGN KEY [本表属性] REFERENCES [其他表]([其他表属性])`进行关联。

```bash
CREATE TABLE class(
    id INT PRIMARY KEY
);

CREATE TABLE student(
    id INT PRIMARY KEY,
    name VARCHAR(20) NOT NULL,
    classId INT NOT NULL UNIQUE,
    FOREIGN KEY(classId) REFERENCES class(id)
);
```

再删除外键相关的内容时需要注意：
1. 主表中没有的数据,在附表中,是不可以使用的；
2. 主表中记录的数据现在正在被附表所引用,那么主表中正在被引用的数据不可以被删除；
3. 若要想删除,先将附表中的数据删除在删除主表数据；

### 唯一约束

约束修饰的列的值不可以重复。使用关键字`UNIQUE`。与主键最大的区别就是该约束修饰的列可以为NULL。

### 非空约束

约束修饰的列的值不能为空。使用关键字为`NOT NULL`。

### 默认约束

约束修饰的列的值如果没有被设置，则可以为预先定义的值。使用关键字为`DEFAULT [value]`。

---

## Alter

`alter`是可以在表已经创建完成后，对表的构造进行设置。

* 修改表名
```bash
ALTER TABLE [tableName] RENAME TO [newTableName];
```

* 删除属性
> 当表中只剩下一个字段时，无法删除
```bash
ALTER TABLE [tableName] DROP [属性名];
```

* 添加属性
```bash
ALTER TABLE [tableName] ADD [属性名] [类型];
ALTER TABLE [tableName] ADD PRIMARY KEY([属性名]);
```

* 修改属性
```bash
ALTER TABLE [tableName] MODIFY [属性名] [类型];
ALTER TABLE [tableName] CHANGE [原属性名] [新属性名] [类型];
```

---

## Where

### 常规查找

在SQL中，WHERE用于设定筛选的条件，其中可以使用`AND`,`OR`等逻辑运算符，同样可以使用`>`, `<`,`=`等比较运算符。

```bash
# 选出pet表中所有age<10并且sex='m'的数据
SELECT * FROM pet WHERE age<10 AND sex='m';
```

### 模糊查找

有的时候，我们不希望全部匹配，而是找出所有包含某一小段数据的时候，就需要使用关键字`LIKE`。

1. `%`表示任意0个或多个字符。
```bash
# 查找学生表中学号开头为U2017的数据，可以是U2017, U20170, U201700, U2017abc等
SELECT * FROM student WHERE studentId LIKE 'U2017%';
```
2. `_`表示一个字符。
```bash
# 查找学生表中学号为U2017x的
SELECT * FROM student WHERE studentId LIKE 'U2017_';
```

3. `[]`表示括号内所列字符中的一个。
```bash
# 查找学生表中学号为U2017或者是M2017的
SELECT * FROM student WHERE studentId LIKE '[UM]2017';
# 查找学生表中学号为U20170~U20179的
SELECT * FROM student WHERE studentId LIKE 'U2017[0-9]';
```

4. `[^]`表示不在括号所列之内的单个字符。
```bash
# 查找学生表中学号不为U2017或者是M2017的，如A2017, B2017等
SELECT * FROM student WHERE studentId LIKE '[^UM]2017';
```

### 常见的条件

* 区间(包含边界)
```bash
# 查询a<score<b的内容
SELECT * FROM [tableName] WHERE score BETWEEN a AND b;
```

* 去重
```bash
# 查询country，并将结果去重
SELECT DISTINCT country FROM [tableName];
```

* 存在区间里的值
```bash
# 查询score为a或b或c的内容
SELECT * FROM [tableName] WHERE score IN(a, b, c);
```

* 记录次数
```bash
SELECT COUNT(*) FROM [tableName] WHERE [case];
```

* 计算平均数
```bash
# 计算满足条件的记录中score的平均分
SELECT AVG(score) FROM [tableName] WHERE [case];
```

* 求和
```bash
# 计算满足条件的记录中score的总分
SELECT SUM(score) FROM [tableName] WHERE [case];
```

* 最大/最小
```bash
# 查找分最高的学生姓名和id
SELECT name, id FROM student WHERE score=(SELECT MAX(score) FROM student)
```

* limit 和 offset
```bash
# 从[开始的位置]取出[数据个数]个记录
SELECT * FROM [tableName] LIMIT [开始的位置] [数据个数];
```

---

## Union

UNION 操作符用于连接两个以上的 SELECT 语句的结果组合到一个结果集合中。默认情况下 UNION 是会自带参数 DISTINCT 并删除重复元素的

```bash
# 下面两个语句相同，用于筛选结果，并去除重复的
SELECT 列名称 FROM 表名称 UNION DISTINCT SELECT 列名称 FROM 表名称 ORDER BY 列名称；
SELECT 列名称 FROM 表名称 UNION SELECT 列名称 FROM 表名称 ORDER BY 列名称；
# 下面的语句用于筛选所有的结果(包括重复的)
SELECT 列名称 FROM 表名称 UNION ALL SELECT 列名称 FROM 表名称 ORDER BY 列名称；
```

---

## 排序

使用`ORDER BY [ASC/DESC]`来设定升序/降序排列，默认为升序。

```bash
# 在满足条件的记录中，按照[属性]升序排列
SELECT * FROM [tableName] WHERE [case] ORDER BY [属性] ASC;
# 先以[属性1]升序排列，如果有相同情况时，再以[属性2]降序
SELECT * FROM [tableName] WHERE [case] ORDER BY [属性1], [属性2] DESC;
```


---

## 分组

使用`GROUP BY`将数据先进行分组，然后根据分组后的内容执行函数。

```bash
# 查找满足条件的内容，将class按组结合起来，最后按class的字母序输出 班级名称-班级的平均分
SELECT class, AVG(score) FROM [tableName] WHERE [case] GROUP BY class ORDER BY class;
```

> 注意，必须要按照这个顺序`WHERE ... GROUP BY ... HAVING ... ORDER BY ...`。因为`WHERE`>`GROUP BY`>`HAVING`>`ORDER BY`。也很好理解，首先在乱序的表里把满足条件的记录保留，再是按照内容分组，再是挑选满足要求的组，再是按照顺序输出剩下来的组的内容。

---

## 存在

使用`HAVING`对分过组的数据进行过滤，所以必须要在`GROUP BY`的后面出现。

其作用类似于`WHERE`，不过`WHERE`是在分组前对数据进行过滤，而`HAVING`是对分组后的数据进行过滤。

```bash
# 挑选出至少两名学生选修并以3开头的课程60分以上的学生的平均分
SELECT courseId, AVG(score) FROM [tableName] WHERE score>=60 GROUP BY courseId HAVING COUNT(student)>=2 AND courseId LIKE "3%";
```

---

## 多表查询

假设有两个表 *student* 和 *course*
|  id   |   name   |
| :---: | :------: |
|   1   |  kexin   |
|   2   | xiaowang |
|   3   |   tom    |

|  id   | course | studentId |
| :---: | :----: | :-------: |
|   1   |  Math  |     1     |
|   2   |  Math  |     2     |
|   3   |  Math  |     3     |
|   4   |  Phy   |     2     |

那么如何同时查询*name*和*course*呢？
```bash
SELECT name, course FROM student, course WHERE student.id=course.studentId
```

上述的输出为
|   name   | course |
| :------: | :----: |
|  kexin   |  Math  |
| xiaowang |  Math  |
|   tom    |  Math  |
| xiaowang |  Phy   |

---

## 嵌套查询

嵌套查询通常运用于要求输出某个列，但是限定的条件来自于多个表。与多表查询不同，*多表查询要求输出的列是多个表中的，但嵌套查询一般输出的列是在同一个表中的*。

```bash
# 查找xiaoming选修的所有课程
SELECT course FROM course WHERE studentId=(SELECT id FROM student WHERE name="xiaoming");
```

> 在嵌套查询中，还有几个常见的关键字——`ALL`, `ANY`, `IN`和`SOME`。
> * 其中`ALL`和`ANY`需要搭配比较运算符，表示`op所有的查询结果`/`op任意的查询结果`
> * `IN`相当于`=ANY`，即只要在查询结果中出现即可
> * `SOME`

---

## 连接

```bash
# 创建两个表，如下
CREATE TABLE person(
    id INT PRIMARY KEY,
    name VARCHAR(20),
    cardId INT
);

CREATE TABLE card(
    id INT PRIMARY KEY,
    name VARCHAR(20)
);
```

|  id   |  name  |
| :---: | :----: |
|   1   |  饭卡  |
|   2   | 建行卡 |
|   3   | 农行卡 |
|   4   | 工商卡 |
|   5   | 邮政卡 |

|  id   | name  | cardId |
| :---: | :---: | :----: |
|   1   | 张三  |   1    |
|   2   | 李四  |   3    |
|   3   | 王五  |   6    |


### 内连接(join / inner join)

匹配两个表中共有的值所在的行。

<img src="https://img-blog.csdnimg.cn/2020070712591551.png">

```bash
SELECT * FROM person JOIN card ON card.id=person.cardId;
```

|  id   | name  | cardId |  id   |  name  |
| :---: | :---: | :----: | :---: | :----: |
|   1   | 张三  |   1    |   1   |  饭卡  |
|   2   | 李四  |   3    |   3   | 农行卡 |

### 外连接

#### 左连接(left join / left outer join)

用左表中所有行匹配右表中的行，右表没有的用NULL填充。

<img src="https://img-blog.csdnimg.cn/2020070712591564.png">

```bash
SELECT * FROM person LEFT JOIN card ON card.id=person.cardId;
```

|  id   | name  | cardId |  id   |  name  |
| :---: | :---: | :----: | :---: | :----: |
|   1   | 张三  |   1    |   1   |  饭卡  |
|   2   | 李四  |   3    |   3   | 农行卡 |
|   3   | 王五  |   6    | NULL  |  NULL  |

#### 右连接(right join / right outer join)

用右表中所有行匹配左表中的行，左表没有的用NULL填充。

<img src="https://img-blog.csdnimg.cn/20200707125915108.png">

```bash
SELECT * FROM person RIGHT JOIN card ON card.id=person.cardId;
```

|  id   | name  | cardId |  id   |  name  |
| :---: | :---: | :----: | :---: | :----: |
|   1   | 张三  |   1    |   1   |  饭卡  |
|   2   | 李四  |   3    |   3   | 农行卡 |
|   3   | NULL  |  NULL  |   2   | 建行卡 |
|   4   | NULL  |  NULL  |   4   | 工商卡 |
|   5   | NULL  |  NULL  |   5   | 邮政卡 |

#### 完全外连接(full join / full outer join)

查询的结果集包括左表和右表的所有行。如果某行在另一个表中没有匹配行时，则用NULL填充。

> MySQL没有 `full join`，但是可以使用`union`将`left join`和`right join`合起来实现。

<img src="https://img-blog.csdnimg.cn/2020070712591593.png">

---

## 事务

事务主要用于处理操作量大，复杂度高的数据。比如说，在人员管理系统中，你删除一个人员，你既需要删除人员的基本资料，也要删除和该人员相关的信息，如信箱，文章等等，这样，这些数据库操作语句就构成一个事务。

1. 在 MySQL 中只有使用了 Innodb 数据库引擎的数据库或表才支持事务。
2. 事务处理可以用来维护数据库的完整性，保证成批的 SQL 语句要么全部执行，要么全部不执行。
3. 事务用来管理 insert,update,delete 语句

### ACID：

1. **原子性**：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被 **回滚（Rollback）** 到事务开始前的状态，就像这个事务从来没有执行过一样。

2. **一致性**：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。

3. **隔离性**：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。

4. **持久性**：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

### 回滚与提交

在 MySQL 命令行的默认设置下，事务都是自动提交的，即执行 SQL 语句后就会马上执行 COMMIT 操作，而提交后就无法更改/撤销。因此要显式地开启一个事务务须使用命令 `BEGIN` 或 `START TRANSACTION`，或者执行命令 `SET AUTOCOMMIT=0`，用来禁止使用当前会话的自动提交。

```bash
SET AUTOCOMMIT=0;                       # 设置关闭自动提交，相当于开启事务
INSERT INTO [tableName] VALUES(...);    # 插入一条数据
ROLLBACK;                               # 撤回刚才的操作
INSERT INTO [tableName] VALUES(...);    # 再插入一条数据
COMMIT;                                 # 提交
ROLLBACK;                               # 此时无法撤回
```

### 事务控制语句

1. BEGIN 或 START TRANSACTION 显式地开启一个事务；

2. COMMIT 也可以使用 COMMIT WORK，不过二者是等价的。COMMIT 会提交事务，并使已对数据库进行的所有修改成为永久性的；

3. ROLLBACK 也可以使用 ROLLBACK WORK，不过二者是等价的。回滚会结束用户的事务，并撤销正在进行的所有未提交的修改；

4. SAVEPOINT identifier，SAVEPOINT 允许在事务中创建一个保存点，一个事务中可以有多个 SAVEPOINT；

5. RELEASE SAVEPOINT identifier 删除一个事务的保存点，当没有指定的保存点时，执行该语句会抛出一个异常；

6. ROLLBACK TO identifier 把事务回滚到标记点；

7. SET TRANSACTION 用来设置事务的隔离级别。InnoDB 存储引擎提供事务的隔离级别有READ UNCOMMITTED、READ COMMITTED、REPEATABLE READ 和 SERIALIZABLE。

### 隔离级别

查看数据库隔离级别的语句(适用于MySQL8.0): `SELECT @@global.transaction_isolation;`。

修改数据库的隔离级别(适用与MySQL8.0): `SET GLOBAL TRANSACTION ISOLATION LEVEL [level];`。

1. **READ UNCOMMITTED** -- 读未提交的。如果事务a进行了操作但是未提交，另一个事务b可以看见a的操作，这样被称作**脏读**

2. **READ COMMITTED** -- 读已提交的。事务a多次读取同一数据,事务b在事务a多次读取的过程中,对数据作了更新并提交,导致事务a多次读取同一数据时,结果 不一致，这样被称作**不可重复读现象**

3. **REPEATABLE READ** -- 重复读。事务a和事务b同时操作一张表，其中事务a提交了一个新的信息，并且没有终止会话，但是此时事务b无法读取到最新的信息，需要等待事务a完全退出数据库，才能更新到数据，这样被称作**幻读**

4. **SERIALIZABLE** -- 串行化。事务a提交一条信息，事务b开始查询该表，事务a再次提交一条信息，此时事务a被阻塞，要等待事务b退出查询(比如事务b输入`commit`)，事务a才会继续执行。

> 上述四种隔离级别递增，但是性能也越来越差。

---

## 函数

可以参考[这里](https://www.runoob.com/mysql/mysql-functions.html)查询一些常用的函数。

---

## 触发器

监视某种情况，并触发某种操作，它是提供给程序员和数据分析员来保证数据完整性的一种方法，它是与表事件相关的特殊的存储过程，它的执行不是由程序调用，也不是手工启动，而是由事件来触发。

* 触发器四要素

1. 监视地点(table)
2. 监视事件(insert/update/delete) 
3. 触发时间(after/before) 
4. 触发事件(insert/update/delete)

```bash
CREATE TRIGGER triggerName after/before insert/update/delete ON 表名 FOR EACH ROW
begin
# sql语句;
end;
```

由于监视的时间只有三种，所以又将触发器分为三种：插入型、更改型和删除型

* new & old

比如有一个这样的业务场景，我在a表中增加一条数据(比如新增m个学生)，并希望b表中相应的数据能变化(比如班级人数+m)
```bash
CREATE TRIGGER tg0 AFTER INSERT ON a FOR EACH ROW
BEGIN
    UPDATE b SET studentNum=studentNum+new.num WHERE b.id=a.classId
END
```

1. 在 INSERT 型触发器中，NEW 用来表示将要（BEFORE）或已经（AFTER）插入的新数据；
2. 在 UPDATE 型触发器中，OLD 用来表示将要或已经被修改的原数据，NEW 用来表示将要或已经修改为的新数据；
3. 在 DELETE 型触发器中，OLD 用来表示将要或已经被删除的原数据。

> OLD 是只读的，而 NEW 则可以在触发器中使用 SET 赋值，这样不会再次触发触发器，造成循环调用。

---

## 索引

索引是对数据库表中一列或多列的值进行排序的一种结构。MySQL索引的建立对于MySQL的高效运行是很重要的，索引可以大大提高MySQL的检索速度。更多内容可以参考[这里](https://blog.csdn.net/wangfeijiu/article/details/113409719)。

### 优缺点

优点：索引大大减小了服务器需要扫描的数据量，从而大大加快数据的检索速度，这也是创建索引的最主要的原因。

缺点：创建索引和维护索引(比如增删改的时候除了维护数据，还要维护索引)要耗费时间，这种时间随着数据量的增加而增加；而且索引需要占物理空间，除了数据表占用数据空间之外，每一个索引还要占用一定的物理空间，如果需要建立聚簇索引，那么需要占用的空间会更大。

### 创建索引的准则

* 应该创建索引的位置

1. 在经常搜索的字段上设置索引
2. 在作为主键的列上，强制该列的唯一性和组织表中数据的排列结构
3. 在经常用在连接（JOIN）的列上，这些列主要是一外键，可以加快连接的速度
在经常需要根据范围（<，<=，=，>，>=，BETWEEN，IN）进行搜索的列上创建索引，因为索引已经排序，其指定的范围是连续的
4. 在经常需要排序（order by）的列上创建索引，因为索引已经排序，这样查询可以利用索引的排序，加快排序查询时间
5. 在经常使用在WHERE子句中的列上面创建索引，加快条件的判断速度

* 不应该创建索引的位置

1. text, img等数据量巨大的字段
2. 不常用的字段
3. 重复度过高的字段


### 索引结构

Mysql数据库中的常见索引结构有多种，常用Hash，B-树(B-tree)，B+树等数据结构来进行数据存储。树的深度加深一层，意味着多一次查询，对于数据库磁盘而言，就是多一次IO操作，导致查询效率低下。

#### B-Tree

即平衡树，类似于平衡二叉树，但是此处可以有m叉。

<img src="https://pic2.zhimg.com/80/v2-2c2264cc1c6c603dfeca4f84a2575901_720w.jpg">

在查询的时候，使用二分查找的思想，每次根据大小情况选择一条路径，走到下一个层次中。

#### B+Tree

B-Tree的升级版，查找的速度更快，且更加稳定(访问每一个元素的时间都差不多)。

1. B+树的非叶子节点不保存关键字记录的指针，只进行数据索引，这样使得B+树每个非叶子节点所能保存的关键字大大增加；
2. B+树叶子节点保存了父节点的所有关键字记录的指针，所有数据地址必须要到叶子节点才能获取到。所以每次数据查询的次数都一样；
3. B+树叶子节点的关键字从小到大有序排列，左边结尾数据都会保存右边节点开始数据的指针。

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/37/Bplustree.png/600px-Bplustree.png">

B+Tree的特点：

1. B+树的**层级更少**：相较于B树B+每个非叶子节点存储的关键字数更多，树的层级更少所以查询数据更快；

2. B+树查询**速度更稳定**：B+所有关键字数据地址都存在叶子节点上，所以每次查找的次数都相同所以查询速度要比B树更稳定;

3. B+树天然**具备排序功能**：B+树所有的叶子节点数据构成了一个有序链表，在查询大小区间的数据时候更方便，数据紧密性很高，缓存的命中率也会比B树高。

4. B+树全节点**遍历更快**：B+树遍历整棵树只需要遍历所有的叶子节点即可，，而不需要像B树一样需要对每一层进行遍历，这有利于数据库做全表扫描。

B树相对于B+树的优点是，如果经常访问的数据离根节点很近，而B树的非叶子节点本身存有关键字其数据的地址，所以这种数据检索的时候会要比B+树快。

#### Hash

利用哈希函数进行查找，注意，只能进行**等值查找**，无法进行范围查找


### 联合索引

即定义多个字段，将这多个字段共同作为索引。比如索引为(a,b,c)，那么现用a进行索引，如果出现无法索引的部分，再使用b，以此类推。

联合索引有许多规则：

1. **最左前缀原则**。一个联合索引（a,b,c）,如果有一个查询条件有a，有b，那么他则走索引，如果有一个查询条件没有a，那么他则不走索引。

> 比如索引为(name, age)，那么
> * `SELECT * FROM table WHERE name="..." AND age=...;`
> * `SELECT * FROM table WHERE name="...";`
> 
> 都会使用索引，而
> * `SELECT * FROM table WHERE age=...;`
> 
> 则不会使用索引，因为联合索引中定义了(name, age)，如果没有name，则不用索引进行查找。

2. 使用**唯一索引**。具有多个重复值的列，其索引效果最差。

一些联合索引的例题，可以点击[这里](https://zhuanlan.zhihu.com/p/115778804)。


---


## 视图

---
