可以把 SQL 分为两个部分：数据操作语言 (DML) 和 数据定义语言 (DDL)。

SQL (结构化查询语言)是用于执行查询的语法。但是 SQL 语言也包含用于更新、插入和删除记录的语法。

查询和更新指令构成了 SQL 的 DML 部分：

- *SELECT* - 从数据库表中获取数据
- *UPDATE* - 更新数据库表中的数据
- *DELETE* - 从数据库表中删除数据
- *INSERT INTO* - 向数据库表中插入数据

SQL 的数据定义语言 (DDL) 部分使我们有能力创建或删除表格。我们也可以定义索引（键），规定表之间的链接，以及施加表间的约束。

SQL 中最重要的 DDL 语句:

- *CREATE DATABASE* - 创建新数据库
- *ALTER DATABASE* - 修改数据库
- *CREATE TABLE* - 创建新表
- *ALTER TABLE* - 变更（改变）数据库表
- *DROP TABLE* - 删除表
- *CREATE INDEX* - 创建索引（搜索键）
- *DROP INDEX* - 删除索引

## SQL 语句总结

| 语句                                                    | 描述                                   | 语法                                                         |
| :------------------------------------------------------ | -------------------------------------- | :----------------------------------------------------------- |
| AND / OR                                                | 基于一个以上的条件对记录进行过滤       | SELECT column_name(s)<br/>FROM table_name<br/>WHERE condition **AND\|OR** condition |
| **ALTER TABLE (add column)**                            | 变更（改变）数据库表                   | ALTER TABLE table_name<br/>ADD column_name datatype          |
| ALTER TABLE (drop column)                               | 修改数据库表                           | ALTER TABLE table_name<br/>DROP COLUMN column_name           |
| AS (alias for column)                                   | 为列名称指定别名                       | SELECT column_name<br/>AS column_alias<br/>FROM table_name   |
| AS (alias for table)                                    | 为表名称指定别名                       | SELECT column_name<br/>FROM table_name<br/>AS table_alias    |
| BETWEEN                                                 | 选取介于两个值之间的数据范围           | SELECT column_name(s)<br/>FROM table_name<br/>WHERE column_name BETWEEN value1 AND value2 |
| **CREATE DATABASE**                                     | 创建新数据库                           | CREATE DATABASE database_name                                |
| **CREATE INDEX**                                        | 创建索引（搜索键）                     | CREATE INDEX index_name<br/>ON table_name (column_name)      |
| **CREATE TABLE**                                        | 创建新表                               | CREATE TABLE table_name ( column_name1 data_type, column_name2 data_type, ....... ) |
| CREATE UNIQUE INDEX                                     |                                        | CREATE UNIQUE INDEX index_name<br/>ON table_name (column_name) |
| CREATE VIEW                                             | 创建视图                               | CREATE VIEW view_name AS<br/>SELECT column_name(s)<br/>FROM table_name<br/>WHERE condition |
| DELETE FROM                                             | 删除表中的行                           | DELETE FROM table_name<br/>(**Note:** Deletes the entire table!!)<br/>**or**<br/>DELETE FROM table_name<br/>WHERE condition |
| DROP DATABASE                                           | 删除数据库                             | DROP DATABASE database_name                                  |
| **DROP INDEX**                                          | 删除索引                               | DROP INDEX table_name.index_name                             |
| **DROP TABLE**                                          | 删除表                                 | DROP TABLE table_name                                        |
| GROUP BY                                                | 根据一个或多个列对结果集进行分组       | SELECT column_name1,SUM(column_name2)<br/>FROM table_name<br/>GROUP BY column_name1 |
| HAVING                                                  |                                        | SELECT column_name1,SUM(column_name2)<br/>FROM table_name<br/>GROUP BY column_name1<br/>HAVING SUM(column_name2) condition value |
| IN                                                      | 在 WHERE 子句中规定多个值              | SELECT column_name(s)<br/>FROM table_name<br/>WHERE column_name IN (value1,value2,..) |
| INSERT INTO                                             | 向表格中插入新的行                     | INSERT INTO table_name<br/>VALUES (value1, value2,....)<br/>**or**<br/>INSERT INTO table_name<br/>(column_name1, column_name2,...)<br/>VALUES (value1, value2,....) |
| LIKE                                                    | 在 WHERE 子句中搜索列中的指定模式      | SELECT column_name(s)<br/>FROM table_name<br/>WHERE column_name LIKE pattern |
| ORDER BY                                                | 对结果集进行排序                       | SELECT column_name(s)<br/>FROM table_name<br/>ORDER BY column_name [ASC\|DESC] |
| SELECT                                                  | 从表中选取数据                         | SELECT column_name(s)<br/>FROM table_name                    |
| SELECT *                                                | 从表中选取所有列                       | SELECT *<br/>FROM table_name                                 |
| SELECT DISTINCT                                         | 从表中选取各不相同的列                 | SELECT DISTINCT column_name(s)<br/>FROM table_name           |
| SELECT INTO (used to create backup copies of tables)    |                                        | SELECT *<br/>INTO new_table_name<br/>FROM original_table_name<br/>***or**<br/>SELECT column_name(s)<br/>INTO new_table_name<br/>FROM original_table_name |
| TRUNCATE TABLE (deletes only the data inside the table) | 仅仅删除表格中的数据，但并不删除表本身 | TRUNCATE TABLE table_name                                    |
| UPDATE                                                  | 修改表中的数据                         | UPDATE table_name<br/>SET column_name=new_value<br/>[, column_name=new_value]<br/>WHERE column_name=some_value |
| WHERE                                                   | 规定选择的标准                         | SELECT column_name(s)<br/>FROM table_name<br/>WHERE condition |

```sql
(7)     SELECT 
(8)     DISTINCT <select_list>
(1)     FROM <left_table>
(3)     <join_type> JOIN <right_table>
(2)     ON <join_condition>
(4)     WHERE <where_condition>
(5)     GROUP BY <group_by_list>
(6)     HAVING <having_condition>
(9)     ORDER BY <order_by_condition>
(10)    LIMIT <limit_number>
```

