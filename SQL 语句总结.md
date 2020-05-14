## SQL 语句总结

| 语句                                                    | 描述                              | 语法                                                         |
| :------------------------------------------------------ | --------------------------------- | :----------------------------------------------------------- |
| AND / OR                                                | 基于一个以上的条件对记录进行过滤  | SELECT column_name(s)<br/>FROM table_name<br/>WHERE condition AND\|OR condition |
| **ALTER TABLE (add column)**                            | 变更（改变）数据库表              | ALTER TABLE table_name<br/>ADD column_name datatype          |
| ALTER TABLE (drop column)                               | 修改数据库表                      | ALTER TABLE table_name<br/>DROP COLUMN column_name           |
| AS (alias for column)                                   |                                   | SELECT column_name<br/>AS column_alias<br/>FROM table_name   |
| AS (alias for table)                                    |                                   | SELECT column_name<br/>FROM table_name<br/>AS table_alias    |
| BETWEEN                                                 | 选取介于两个值之间的数据范围      | SELECT column_name(s)<br/>FROM table_name<br/>WHERE column_name BETWEEN value1 AND value2 |
| **CREATE DATABASE**                                     | 创建新数据库                      | CREATE DATABASE database_name                                |
| **CREATE INDEX**                                        | 创建索引（搜索键）                | CREATE INDEX index_name<br/>ON table_name (column_name)      |
| **CREATE TABLE**                                        | 创建新表                          | CREATE TABLE table_name ( column_name1 data_type, column_name2 data_type, ....... ) |
| CREATE UNIQUE INDEX                                     |                                   | CREATE UNIQUE INDEX index_name<br/>ON table_name (column_name) |
| CREATE VIEW                                             | 创建视图                          | CREATE VIEW view_name AS<br/>SELECT column_name(s)<br/>FROM table_name<br/>WHERE condition |
| DELETE FROM                                             |                                   | DELETE FROM table_name<br/>(**Note:** Deletes the entire table!!)<br/>**or**<br/>DELETE FROM table_name<br/>WHERE condition |
| DROP DATABASE                                           |                                   | DROP DATABASE database_name                                  |
| **DROP INDEX**                                          | 删除索引                          | DROP INDEX table_name.index_name                             |
| **DROP TABLE**                                          | 删除表                            | DROP TABLE table_name                                        |
| GROUP BY                                                |                                   | SELECT column_name1,SUM(column_name2)<br/>FROM table_name<br/>GROUP BY column_name1 |
| HAVING                                                  |                                   | SELECT column_name1,SUM(column_name2)<br/>FROM table_name<br/>GROUP BY column_name1<br/>HAVING SUM(column_name2) condition value |
| IN                                                      | 在 WHERE 子句中规定多个值         | SELECT column_name(s)<br/>FROM table_name<br/>WHERE column_name IN (value1,value2,..) |
| INSERT INTO                                             | 向表格中插入新的行                | INSERT INTO table_name<br/>VALUES (value1, value2,....)<br/>**or**<br/>INSERT INTO table_name<br/>(column_name1, column_name2,...)<br/>VALUES (value1, value2,....) |
| LIKE                                                    | 在 WHERE 子句中搜索列中的指定模式 | SELECT column_name(s)<br/>FROM table_name<br/>WHERE column_name LIKE pattern |
| ORDER BY                                                | 对结果集进行排序                  | SELECT column_name(s)<br/>FROM table_name<br/>ORDER BY column_name [ASC\|DESC] |
| SELECT                                                  | 从表中选取数据                    | SELECT column_name(s)<br/>FROM table_name                    |
| SELECT *                                                | 从表中选取所有列                  | SELECT *<br/>FROM table_name                                 |
| SELECT DISTINCT                                         | 从表中选取各不相同的列            | SELECT DISTINCT column_name(s)<br/>FROM table_name           |
| SELECT INTO (used to create backup copies of tables)    |                                   | SELECT *<br/>INTO new_table_name<br/>FROM original_table_name<br/>***or**<br/>SELECT column_name(s)<br/>INTO new_table_name<br/>FROM original_table_name |
| TRUNCATE TABLE (deletes only the data inside the table) |                                   | TRUNCATE TABLE table_name                                    |
| UPDATE                                                  | 修改表中的数据                    | UPDATE table_name<br/>SET column_name=new_value<br/>[, column_name=new_value]<br/>WHERE column_name=some_value |
| WHERE                                                   | 规定选择的标准                    | SELECT column_name(s)<br/>FROM table_name<br/>WHERE condition |