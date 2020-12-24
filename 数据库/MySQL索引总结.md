索引原理

#### 一、索引的优缺点

##### 1）优点

- 索引大大减小了服务器需要扫描的数据量

- 索引可以帮助服务器避免排序和临时表

- 索引可以将随机IO变成顺序IO

- 索引对于InnoDB（对索引支持行级锁）非常重要，因为它可以让查询锁更少的元组。在MySQL5.1和更新的版本中，InnoDB可以在服务器端过滤掉行后就释放锁，但在早期的MySQL版本中，InnoDB直到事务提交时才会解锁。对不需要的元组的加锁，会增加锁的开销，降低并发性。 InnoDB仅对需要访问的元组加锁，而索引能够减少InnoDB访问的元组数。但是只有在存储引擎层过滤掉那些不需要的数据才能达到这种目的。一旦索引不允许InnoDB那样做（即索引达不到过滤的目的），MySQL服务器只能对InnoDB返回的数据进行WHERE操作，此时，已经无法避免对那些元组加锁了。如果查询不能使用索引，MySQL会进行全表扫描，并锁住每一个元组，不管是否真正需要。

- - 关于InnoDB、索引和锁：InnoDB在二级索引上使用共享锁（读锁），但访问主键索引需要排他锁（写锁）

##### 2）缺点

- 虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。因为更新表时，MySQL不仅要保存数据，还要保存索引文件。
- 建立索引会占用磁盘空间的索引文件。一般情况这个问题不太严重，但如果你在一个大表上创建了多种组合索引，索引文件的会膨胀很快。
- 如果某个数据列包含许多重复的内容，为它建立索引就没有太大的实际效果。
- 对于非常小的表，大部分情况下简单的全表扫描更高效；

索引只是提高效率的一个因素，如果你的MySQL有大数据量的表，就需要花时间研究建立最优秀的索引，或优化查询语句。

因此应该只为最经常查询和最经常排序的数据列建立索引。

MySQL里同一个数据表里的索引总数限制为16个。

#### 二、索引存储类型

##### 1）B-Tree索引

InnoDB使用的是B+Tree。

B+Tree：每一个叶子节点都包含指向下一个叶子节点的指针，从而方便叶子节点的范围遍历。

B-Tree通常意味着所有的值都是按顺序存储的，并且每一个叶子页到根的距离相同，很适合查找范围数据。

B-Tree可以对<，<=，=，>，>=，BETWEEN，IN，以及不以通配符开始的LIKE使用索引。

###### a）索引查询

可以利用B-Tree索引进行全关键字、关键字范围和关键字前缀查询，但必须保证按索引的最左边前缀(leftmost prefix of the index)来进行查询。

![img](https://pic3.zhimg.com/80/v2-307ae0886144c7bc2619691ff616a6a2_720w.jpg)

假设有如下一个表：

```sql
CREATE TABLE People (
  last_name varchar(50) not null,
  first_name varchar(50) not null,
  dob date not null,
  gender enum('m', 'f') not null,
  key(last_name, first_name, dob)
);
```

其**组合索引**包含表中每一行的last_name、first_name和dob列。其结构大致如下：

![img](https://pic3.zhimg.com/80/v2-a0669007f11ae9e57bd64394295f84aa_720w.jpg)

按索引的最左边前缀(leftmost prefix of the index)来进行查询：

1. 查询必须从索引的最左边的列开始，否则无法使用索引。例如，你不能直接利用索引查找在某一天出生的人。
2. 不能跳过某一索引列。例如，你不能利用索引查找last name为Smith且出生于某一天的人。
3. 存储引擎不能使用索引中范围条件右边的列。例如，如果你的查询语句为WHERE last_name="Smith" AND first_name LIKE 'J%' AND dob='1976-12-23'，则该查询只会使用索引中的前两列，因为LIKE是范围查询。



1. 匹配全值(Match the full value)：对索引中的所有列都指定具体的值。例如，上图中索引可以帮助你查找出生于1960-01-01的Cuba Allen。
2. 匹配最左前缀(Match a leftmost prefix)：你可以利用索引查找last name为Allen的人，仅仅使用索引中的第1列。
3. 匹配列前缀(Match a column prefix)：例如，你可以利用索引查找last name以J开始的人，这仅仅使用索引中的第1列。
4. 匹配值的范围查询(Match a range of values)：可以利用索引查找last name在Allen和Barrymore之间的人，仅仅使用索引中第1列。
5. 匹配部分精确而其它部分进行范围匹配(Match one part exactly and match a range on another part)：可以利用索引查找last name为Allen，而first name以字母K开始的人。
6. 仅对索引进行查询(Index-only queries)：如果查询的列都位于索引中，则不需要再多一次I/O回读元组。(覆盖索引：索引的叶子节点中已经包含要查询的数据，那么就没有必要再回表查询了，如果索引包含满足查询的所有数据，就称为覆盖索引。)

###### b）索引排序

也可以利用B-Tree索引进行索引排序（对查询结果进行ORDER BY），必须保证ORDER BY按索引的最左边前缀(leftmost prefix of the index)来进行。

MySQL中，有两种方式生成有序结果集：

- 使用filesort
- 按索引顺序扫描

如果explain出来的type列的值为“index”，则说明MYSQL使用了索引扫描来做排序。

**按索引顺序扫描：**

可以利用同一索引同时进行查找和排序操作：

- 当索引的顺序与ORDER BY中的列顺序相同，且所有的列是同一方向（全部升序或者全部降序）时，可以使用索引来排序。
- ORDER BY子句和查询型子句的限制是一样的：需要满足索引的最左前缀的要求，有一种情况下ORDER BY子句可以不满足索引的最左前缀要求，那就是前导列为常量时：WHERE子句或者JOIN子句中对前导列指定了常量。

![img](https://pic1.zhimg.com/80/v2-19d7202ee71287a606f644837d0c19cc_720w.jpg)

- 如果查询是连接多个表，仅当ORDER BY中的所有列都是第一个表的列时才会使用索引。其它情况都会使用filesort文件排序。

**使用filesort：**

当MySQL不能使用索引进行排序时，就会利用自己的排序算法(快速排序算法)在内存(sort buffer)中对数据进行排序；

如果内存装载不下，它会将磁盘上的数据进行分块，再对各个数据块进行排序，然后将各个块合并成有序的结果集（实际上就是外排序，使用临时表）。

对于**filesort**，MySQL有两种排序算法：

- **两次扫描算法(Two passes)**

先将需要排序的字段和可以直接定位到相关行数据的指针信息取出，然后在设定的内存（通过参数sort_buffer_size设定）中进行排序，完成排序之后再次通过行指针信息取出所需的Columns。

该算法是MySQL4.1之前采用的算法，它需要两次访问数据，尤其是第二次读取操作会导致大量的随机I/O操作。另一方面，内存开销较小。

- **一次扫描算法(single pass)**

该算法一次性将所需的Columns全部取出，在内存中排序后直接将结果输出。

从MySQL4.1版本开始使用该算法。它减少了I/O的次数，效率较高，但是内存开销也较大。如果我们将并不需要的Columns也取出来，就会极大地浪费排序过程所需要的内存。

在 MySQL 4.1 之后的版本中，可以通过设置 max_length_for_sort_data 参数来控制 MySQL 选择第一种排序算法还是第二种：当取出的所有大字段总大小大于 max_length_for_sort_data 的设置时，MySQL 就会选择使用第一种排序算法，反之，则会选择第二种。

当对连接操作进行排序时，如果ORDER BY仅仅引用第一个表的列，MySQL对该表进行filesort操作，然后进行连接处理，此时，EXPLAIN输出“Using filesort”；否则，MySQL必须将查询的结果集生成一个临时表，在连接完成之后进行filesort操作，此时，EXPLAIN输出“Using temporary;Using filesort”。

为了尽可能地提高排序性能，我们自然更希望使用第二种排序算法，所以在 Query 中仅仅取出需要的 Columns 是非常有必要的。

##### 2）聚簇索引(cluster index)

一个表只能有一个聚簇索引。

目前，只有solidDB和InnoDB支持聚簇索引，MyISAM不支持聚簇索引。一些DBMS允许用户指定聚簇索引，但是MySQL的存储引擎到目前为止都不支持。

###### a）InnoDB的聚簇索引：

1. InnoDB对主键建立聚簇索引。
2. 如果你不指定主键，InnoDB会用一个具有唯一且非空值的索引来代替。
3. 如果不存在这样的索引，InnoDB会定义一个隐藏的主键，然后对其建立聚簇索引。

InnoDB默认使用聚簇索引来组织数据，如果你用InnoDB，而且不需要特殊的聚簇索引，一个好的做法就是使用代理主键(surrogate key)——独立于你的应用中的数据。最简单的做法就是使用一个AUTO_INCREMENT的列，这会保证记录按照顺序插入，而且能提高使用primary key进行连接的查询的性能。应该尽量避免随机的聚簇主键，例如字符串主键就是一个不好的选择，它使得插入操作变得随机。

一般来说，DBMS都会以聚簇索引的形式来存储实际的数据，它是其它二级索引的基础：

- 聚簇索引（primary索引）：主键索引
- 非聚簇索引（second索引）：二级索引

###### b）聚簇索引结构：

聚簇索引的结构大致如下：

- **聚簇索引：**节点页只包含了索引列，叶子页包含了行的全部数据。聚簇索引“就是表”，因此可以不需要独立的行存储。

聚簇索引保证关键字的值相近的元组存储的物理位置也相近（所以字符串类型不宜建立聚簇索引，特别是随机字符串，会使得系统进行大量的移动操作）。

![img](https://pic3.zhimg.com/80/v2-f81e0a8e4cd2bf0839c765351f650ffa_720w.jpg)



- **二级索引：**叶子节点保存的不是指行的物理位置的指针，而是行的主键值。

这意味着通过二级索引查找行，存储引擎需要：1、找到二级索引的叶子节点获取对应的主键值，2、根据这个主键值去聚簇索引中查找到对应的行。这里需要两次B-Tree查找而不是一次。

覆盖索引对于InnoDB表尤其有用，因为InnoDB使用聚簇索引组织数据，如果二级索引中包含查询所需的数据，就不再需要在聚集索引中查找了。

###### c）聚簇索引（InnoDB）和二级索引（MyISAM）数据布局比较：

```text
CREATE TABLE layout_test (
  col1 int NOT NULL,
  col2 int NOT NULL,
  PRIMARY KEY(col1),
  KEY(col2)
);
```

- **MyISAM**

MyISAM按照插入的顺序在磁盘上存储数据：

左边为行号(row number)，从0开始。因为元组的大小固定，所以MyISAM可以很容易的从表的开始位置找到某一字节的位置。

![img](https://pic2.zhimg.com/80/v2-1a2aa27dbe33463dd42f0585d2504e3d_720w.jpg)



MyISAM建立的索引结构大致如下：

col1主键索引：

MyISAM不支持聚簇索引，索引中每一个叶子节点仅仅包含行号(row number)，且叶子节点按照col1的顺序存储。

![img](https://pic3.zhimg.com/80/v2-e5622d317fe702269cf05973dcc36b7e_720w.jpg)

col2非主键索引：

在MyISAM中，primary key和其它索引没有什么区别。Primary key仅仅只是一个叫做PRIMARY的唯一，非空的索引而已，叶子节点按照col2的顺序存储。

![img](https://pic3.zhimg.com/80/v2-1c672e22e1d629c651cda207c7b02352_720w.jpg)



- **InnoDB**

col1主键索引，即聚簇索引：

聚簇索引中的每个叶子节点包含主键的值，事务ID，用于事务和MVCC的回滚指针，和余下的列(如col2)。

![img](https://pic1.zhimg.com/80/v2-f20aa7e23a8662b4054b24bc7f86f66c_720w.jpg)

col2非主键索引，即二级索引：

InnoDB的二级索引的叶子包含主键的值，而不是行指针(row pointers)，这样的策略减小了移动数据或者数据页面分裂时维护二级索引的开销，因为InnoDB不需要更新索引的行指针。

![img](https://pic1.zhimg.com/80/v2-28008f3730c8546c1e11bbdac88c92dc_720w.jpg)



- **聚簇索引+二级索引表 与 非聚簇索引表 的对比**

![img](https://pic2.zhimg.com/80/v2-ee2277c868c80bac6d3526e647de8e41_720w.jpg)



##### 3）Hash索引

哈希索引基于哈希表实现，只有精确索引所有列的查询才有效。

对于每一行数据，存储引擎都会对所有的索引列计算一个哈希码。哈希索引将所有的哈希码存储在索引中，同时在哈希表中保存指向每个数据的指针。

MySQL中，只有Memory存储引擎显示支持hash索引，是Memory表的默认索引类型，尽管Memory表也可以使用B-Tree索引。

Memory存储引擎支持非唯一hash索引，这在数据库领域是罕见的：如果多个值有相同的hash code，索引把它们的行指针用链表保存到同一个hash表项中。



假设创建如下一个表：

```text
CREATE TABLE testhash (
  fname VARCHAR(50) NOT NULL,
  lname VARCHAR(50) NOT NULL,
  KEY USING HASH(fname)
) ENGINE=MEMORY;
```

包含的数据如下：

![img](https://pic4.zhimg.com/80/v2-c57ecf890c1e4acbe53560592df2a6e3_720w.jpg)

假设索引使用hash函数f( )，如下：

```text
f('Arjen') = 2323
f('Baron') = 7437
f('Peter') = 8784
f('Vadim') = 2458
```

此时，索引的结构大概如下：

![img](https://pic3.zhimg.com/80/v2-043d10a60fcc2a4a5ee9d696a7537cae_720w.jpg)

哈希索引中存储的是：哈希值+数据行指针

当你执行 SELECT lname FROM testhash WHERE fname='Peter'; MySQL会计算’Peter’的hash值，然后通过它来查询索引的行指针。因为f('Peter') = 8784，MySQL会在索引中查找8784，得到指向记录3的指针。

**Hash索引有以下一些限制：**

- 由于索引仅包含hash code和记录指针，所以，MySQL不能通过使用索引避免读取记录，即每次使用哈希索引查询到记录指针后都要回读元祖查取数据。
- 不能使用hash索引排序。
- Hash索引不支持键的部分匹配，因为是通过整个索引值来计算hash值的。
- Hash索引只支持等值比较，例如使用=，IN( )和<=>。对于WHERE price>100并不能加速查询。
- 访问Hash索引的速度非常快，除非有很多哈希冲突（不同的索引列值却有相同的哈希值）。当出现哈希冲突的时候，存储引擎必须遍历链表中所有的行指针，逐行进行比较，直到找到所有符合条件的行。
- 如果哈希冲突很多的话，一些索引维护操作的代价也会很高。当从表中删除一行时，存储引擎要遍历对应哈希值的链表中的每一行，找到并删除对应行的引用，冲突越多，代价越大。

InnoDB引擎有一个特殊的功能叫做“自适应哈希索引”，由Mysql自动管理，不需要DBA人为干预。默认情况下为开启，我们可以通过参数innodb_adaptive_hash_index来禁用此特性。

当InnoDB注意到某些索引值被使用得非常频繁时，它会在内存中基于缓冲池中的B+ Tree索引上再创建一个哈希索引，这样就上B-Tree索引也具有哈希索引的一些优点，比如快速的哈希查找。

- 只能用于等值比较，例如=， <=>，in ；
- 无法用于排序

InnoDB官方文档显示，启用自适应哈希索引后，读和写性能可以提高2倍，对于辅助索引的连接操作，性能可以提高5倍

##### 4）空间(R-Tree)索引

MyISAM支持空间索引，主要用于地理空间数据类型，例如GEOMETRY。

##### 5）全文(Full-text)索引

全文索引是MyISAM的一个特殊索引类型，它查找的是文本中的关键词，主要用于全文检索。

MySQL InnoDB从5.6开始已经支持全文索引，但InnoDB内部并不支持中文、日文等，因为这些语言没有分隔符。可以使用插件辅助实现中文、日文等的全文索引。详见：[12.9.5 Full-Text Restrictions](https://link.zhihu.com/?target=https%3A//dev.mysql.com/doc/refman/5.7/en/fulltext-restrictions.html)

#### 三、索引使用

##### 1）MySQL建立索引类型

- 单列索引，即一个索引只包含单个列，一个表可以有多个单列索引，但这不是组合索引。
- 组合索引，即一个索包含多个列。

索引是在存储引擎中实现的，而不是在服务器层中实现的。所以，每种存储引擎的索引都不一定完全相同，并不是所有的存储引擎都支持所有的索引类型。

###### a）普通索引

这是最基本的索引，它没有任何限制。普通索引（由关键字KEY或INDEX定义的索引）的唯一任务是加快对数据的访问速度。因此，应该只为那些最经常出现在查询条件(WHERE column = …)或排序条件(ORDER BY column)中的数据列创建索引。

它有以下几种创建方式：

- 创建索引

CREATE INDEX indexName ON mytable(username(length));

如果是CHAR，VARCHAR类型，length可以小于字段实际长度；如果是BLOB和TEXT类型，必须指定 length，下同。

- 修改表结构

ALTER mytable ADD INDEX [indexName] ON (username(length))

- 创建表的时候直接指定

CREATE TABLE mytable( ID INT NOT NULL, username VARCHAR(16) NOT NULL, INDEX [indexName] (username(length)) );

- 删除索引的语法：

DROP INDEX [indexName] ON mytable;

###### b）唯一索引

它与前面的普通索引类似，不同的就是：普通索引允许被索引的数据列包含重复的值。而唯一索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。

它有以下几种创建方式：

- 创建索引

CREATE UNIQUE INDEX indexName ON mytable(username(length))

- 修改表结构

ALTER mytable ADD UNIQUE [indexName] ON (username(length))

- 创建表的时候直接指定

CREATE TABLE mytable( ID INT NOT NULL, username VARCHAR(16) NOT NULL, UNIQUE [indexName] (username(length)) );

###### c）主键索引

它是一种特殊的唯一索引，不允许有空值。一个表只能有一个主键。

一般是在建表的时候同时创建主键索引：

CREATE TABLE mytable( ID INT NOT NULL, username VARCHAR(16) NOT NULL, PRIMARY KEY(ID) ); 当然也可以用 ALTER 命令。

与之类似的，外键索引

如果为某个外键字段定义了一个外键约束条件，MySQL就会定义一个内部索引来帮助自己以最有效率的方式去管理和使用外键约束条件。

###### d）组合索引

为了形象地对比单列索引和组合索引，为表添加多个字段：

CREATE TABLE mytable( ID INT NOT NULL, username VARCHAR(16) NOT NULL, city VARCHAR(50) NOT NULL, age INT NOT NULL );

为了进一步榨取MySQL的效率，就要考虑建立组合索引。就是将 name, city, age建到一个索引里：

ALTER TABLE mytable ADD INDEX name_city_age (name(10),city,age);

建表时，usernname长度为 16，这里用 10。这是因为一般情况下名字的长度不会超过10，这样会加速索引查询速度，还会减少索引文件的大小，提高INSERT的更新速度。

建立这样的组合索引，其实是相当于分别建立了下面三组组合索引：

usernname,city,age

usernname,city

usernname

为什么没有 city，age这样的组合索引呢？这是因为MySQL组合索引“最左前缀”的结果。简单的理解就是只从最左面的开始组合。并不是只要包含这三列的查询都会用到该组合索引。下面的几个SQL就会用到这个组合索引：

SELECT * FROM mytable WHREE username="admin" AND city="郑州"

SELECT * FROM mytable WHREE username="admin"

而下面几个则不会用到：

SELECT * FROM mytable WHREE age=20 AND city="郑州"

SELECT * FROM mytable WHREE city="郑州"

如果分别在 usernname，city，age上建立单列索引，让该表有3个单列索引，查询时和上述的组合索引效率也会大不一样，远远低于我们的组合索引。因为虽然此时有了三个索引，但MySQL只能用到其中的那个它认为似乎是最有效率的单列索引。

##### 2）建立索引的时机

一般来说，在WHERE和JOIN中出现的列需要建立索引，但也不完全如此，因为MySQL的B-Tree只对<，<=，=，>，>=，BETWEEN，IN，以及不以通配符开始的LIKE才会使用索引。

例如：

SELECT t.Name FROM mytable t LEFT JOIN mytable m ON t.Name=m.username WHERE m.age=20 AND m.city='郑州'

此时就需要对city和age建立索引，由于mytable表的userame也出现在了JOIN子句中，也有对它建立索引的必要。

##### 3）正确使用索引

使用（B-Tree）索引时，有以下一些技巧和注意事项：

##### 4）索引设计：

- 索引字段尽量使用数字型（简单的数据类型）

若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了

- 尽量不要让字段的默认值为NULL

在MySQL中，含有空值的列很难进行查询优化，因为它们使得索引、索引的统计信息以及比较运算更加复杂。

索引不会包含有NULL值的列，只要列中包含有NULL值都将不会被包含在索引中，复合索引中只要有一列含有NULL值，那么这一列对于此复合索引就是无效的。

所以我们在数据库设计时尽量不要让字段的默认值为NULL，应该指定列为NOT NULL，除非你想存储NULL。你应该用0、一个特殊的值或者一个空串代替空值。

- 前缀索引和索引选择性

对串列进行索引，如果可能应该指定一个前缀长度。

对于BLOB、TEXT或者很长的VARCHAR类型的列，必须使用前缀索引，因为MYSQL不允许索引这些列的完整长度。

前缀索引是一种能使索引更小、更快的有效办法，但另一方面也有其缺点：MySQL无法使用前缀索引做order by和group by，也无法使用前缀索引做覆盖扫描。

一般情况下某个前缀的选择性也是足够高的，足以满足查询性能。

例如，如果有一个CHAR(255)的列，如果在前10个或20个字符内，多数值是惟一的，那么就不要对整个列进行索引。

短索引不仅可以提高查询速度而且可以节省磁盘空间和I/O操作。在绝大多数应用里，数据库中的字符串数据大都以各种各样的名字为主，把索引的长度设置为10~15个字符已经足以把搜索范围缩小到很少的几条数据记录了。

通常可以索引开始的部分字符，这样可以大大节约索引空间，从而提高索引效率。但这样也会降低索引的选择性。

索引的选择性是指，不重复的索引值（基数）和数据表中的记录总数的比值。索引的选择性越高则查询效率越高，因为选择性高的索引可以让MYSQL在查找时过滤掉更多的行。唯一索引的选择性是1，这是最好的索引选择性，性能也是最好的。

决窍在于要选择足够长的前缀以保证较高的选择性，同时又不能太长（以便节约空间）。前缀应该足够长，以使得前缀索引的选择性接近于索引整个列。换句话说，前缀的“基数”应该接近于完整列的“基数”。

为了决定前缀的合适长度，需要找到最常见的值的列表，然后和最常见的前缀列表进行比较。例如以下查询：

select count(*) as cnt,city from sakila.city_demo group by city order by cnt desc limit 10;

select count(*) as cnt,left(city,7) as perf from sakila.city_demo group by city order by cnt desc limit 10;

直到这个前缀的选择性接近完整列的选择性。

计算合适的前缀长度的另一个方法就是计算完整列的选择性，并使前缀的选择性接近于完整列的选择性，如下：

select count(distinct city)/count(*) from sakila.city_demo;

select count(distinct left(city,7))/count(*) from sakila.city_demo;

- 使用唯一索引

考虑某列中值的分布。索引的列的基数越大，索引的效果越好。

例如，存放出生日期的列具有不同值，很容易区分各行。而用来记录性别的列，只含有“ M” 和“F”，则对此列进行索引没有多大用处，因为不管搜索哪个值，都会得出大约一半的行。

- 使用组合索引代替多个列索引

一个多列索引（组合索引）与多个列索引MySQL在解析执行上是不一样的，如果在explain中看到有索引合并（即MySQL为多个列索引合并优化），应该好好检查一下查询的表和结构是不是已经最优。

- 注意重复/冗余的索引、不使用的索引

MySQL允许在相同的列上创建多个索引，无论是有意还是无意的。大多数情况下不需要使用冗余索引。

对于重复/冗余、不使用的索引：可以直接删除这些索引。因为这些索引需要占用物理空间，并且也会影响更新表的性能。

##### 6）索引使用：

- 如果对大的文本进行搜索，使用全文索引而不要用使用 like ‘%…%’
- like语句不要以通配符开头

对于LIKE：在以通配符%和_开头作查询时，MySQL不会使用索引。like操作一般在全文索引中会用到（InnoDB数据表不支持全文索引）。

例如下句会使用索引：

SELECT * FROM mytable WHERE username like'admin%'

而下句就不会使用：

SELECT * FROM mytable WHEREt Name like'%admin'

- 不要在列上进行运算

索引列不能是表达式的一部分，也不是是函数的参数。

例如以下两个查询无法使用索引：

1）表达式： select actor_id from sakila.actor where actor_id+1=5;

2）函数参数：select ... where TO_DAYS(CURRENT_DATE) - TO_DAYS(date_col)<=10;

- 尽量不要使用NOT IN、<>、!= 操作

应尽量避免在 where 子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描。

对于not in，可以用not exists或者（外联结+判断为空）来代替；很多时候用 exists 代替 in 是一个好的选择： select num from a where num in(select num from b) 用下面的语句替换： select num from a where exists(select 1 from b where num=a.num)

对于<>，用其它相同功能的操作运算代替，如a<>0 改为 a>0 or a<0

- or条件

用 or 分割开的条件， 如果 or 前的条件中的列有索引， 而后面的列中没有索引， 那么涉及到的索引都不会被用到。

应尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，如： 假设num1有索引，num2没有索引，查询语句select id from t where num1=10 or num2=20会放弃使用索引，可以改为这样查询： select id from t where num1=10 union all select id from t where num2=20，这样虽然num2没有使用索引，但至少num1会使用索引，提高效率。

- 组合索引的使用要遵守“最左前缀”原则'

组合索引：当不需要考虑排序和分组时，将选择性最高的列放在前面通常是最好的。

例子：

```text
CREATE TABLE People (
  last_name varchar(50) not null,
  first_name varchar(50) not null,
  birthday date not null,
  gender enum('m', 'f') not null,
  key(last_name, first_name, birthday)
);
```

1. 查询必须从索引的最左边的列开始，否则无法使用索引。例如，你不能直接利用索引查找在某一天出生的人。
2. 不能跳过某一索引列。例如，你不能利用索引查找last name为Smith且出生于某一天的人。
3. 存储引擎不能使用索引中范围条件右边的列。例如，如果你的查询语句为WHERE last_name="Smith" AND first_name LIKE 'J%' AND dob='1976-12-23'，则该查询只会使用索引中的前两列，因为LIKE是范围查询。

- 使用索引排序时，ORDER BY也要遵守“最左前缀”原则

1. 当索引的顺序与ORDER BY中的列顺序相同，且所有的列是同一方向（全部升序或者全部降序）时，可以使用索引来排序。
2. ORDER BY子句和查询型子句的限制是一样的：需要满足索引的最左前缀的要求，有一种情况下ORDER BY子句可以不满足索引的最左前缀要求，那就是前导列为常量时：WHERE子句或者JOIN子句中对前导列指定了常量。
3. 如果查询是连接多个表，仅当ORDER BY中的所有列都是第一个表的列时才会使用索引。其它情况都会使用filesort文件排序。

![img](https://pic1.zhimg.com/80/v2-19d7202ee71287a606f644837d0c19cc_720w.jpg)

- 如果列类型是字符串，那么一定记得在 where 条件中把字符常量值用引号引起来，否则的话即便这个列上有索引，MySQL 也不会用到的，因为MySQL 默认把输入的常量值进行转换以后才进行检索。 例如：

![img](https://pic3.zhimg.com/80/v2-35599795a44296d2763ec240c81a0d02_720w.jpg)



![img](https://pic2.zhimg.com/80/v2-389506a680f6bfb999c1d7559e3245e9_720w.jpg)

- 任何地方都不要使用 select * from t ，用具体的字段列表代替“*”，不要返回用不到的任何字段
- 如果 MySQL 估计使用索引比全表扫描更慢，则不使用索引。当索引列有大量数据重复时,查询可能不会去利用索引，如一表中有字段sex，male、female几乎各一半，那么即使在sex上建了索引也对查询效率起不了作用。

##### 7）索引性能测试与索引优化

只有当数据库里已经有了足够多的测试数据时，它的性能测试结果才有实际参考价值。如果在测试数据库里只有几百条数据记录，它们往往在执行完第一条查询命令之后就被全部加载到内存里，这将使后续的查询命令都执行得非常快——不管有没有使用索引。只有当数据库里的记录超过了1000条、数据总量也超过了 MySQL服务器上的内存总量时，数据库的性能测试结果才有意义。

在不确定应该在哪些数据列上创建索引的时候，人们从EXPLAIN SELECT命令那里往往可以获得一些帮助。这其实只是简单地给一条普通的SELECT命令加一个EXPLAIN关键字作为前缀而已。有了这个关键字，MySQL将不是去执行那条SELECT命令，而是去对它进行分析。MySQL将以表格的形式把查询的执行过程和用到的索引(如果有的话)等信息列出来。

**查看索引使用情况：**

- 如果索引正在工作，Handler_read_key 的值将很高，这个值代表了一个行被索引值读的次数，很低的值表明增加索引得到的性能改善不高，因为索引并不经常使用。
- Handler_read_rnd_next 的值高则意味着查询运行低效，并且应该建立索引补救。这个值的含义是在数据文件中读下一行的请求数。如果正进行大量的表扫描， Handler_read_rnd_next 的值较高，则通常说明表索引不正确或写入的查询没有利用索引。

具体如下：

![img](https://pic3.zhimg.com/80/v2-92746b3546a6251de75b77e34cf54272_720w.jpg)

从上面的例子中可以看出，目前使用的 MySQL 数据库的索引情况并不理想。