# SQL 和 nGQL

## 基本概念对比

|概念名称               | SQL | nGQL          |
| --- | --- | --- |
| vertex      | \  | vertex        |
| edge | \    | edge          |
| vertex type        | \   | tag           |
| edge type          | \  | edge type     |
| vertex identifier          | \ | vid           |
| edge identifier        | edge id generated by default   | src, dst, rank  |
| column | column | \ |
| row | row | \ |

## 语法对比

### 数据定义语言（DDL）

数据定义语言（DDL）用于定义数据库 schema。 DDL 语句可以创建或修改数据库的结构。

对比项                    | SQL                   | nGQL
-------------------------| ------------------------ | -----------
创建图空间（数据库）              | CREATE DATABASE `<database_name>`                    | CREATE SPACE `<space_name>`
列出图空间（数据库）          | SHOW DATABASES | SHOW SPACES
使用图空间（数据库）  | USE `<database_name>` | USE `<space_name>`
删除图空间（数据库） | DROP DATABASE `<database_name>` | DROP SPACE `<space_name>`
修改图空间（数据库） | ALTER DATABASE `<database_name>` alter_option | \
创建 tags/edges | \ | CREATE TAG \| EDGE `<tag_name>`
创建表 | CREATE TABLE `<tbl_name>` (create_definition,...) | \
列出表列名 | SHOW COLUMNS FROM `<tbl_name>` | \
列出 tags/edges | \ | SHOW TAGS \| EDGES
Describe tags/edge | \ | DESCRIBE TAG \| EDGE `<tag_name | edge_name>`
修改 tags/edge | \ | ALTER TAG \| EDGE `<tag_name | edge_name>`
修改表 | ALTER TABLE `<tbl_name>` | \

#### 索引

对比项                    | SQL                   | nGQL
-------------------------| ------------------------ | -----------
创建索引 | CREATE INDEX | CREATE {TAG \| EDGE} INDEX
删除索引 | DROP INDEX | DROP {TAG \| EDGE} INDEX
列出索引 | SHOW INDEX FROM | SHOW {TAG \| EDGE} INDEXES
重构索引 | ANALYZE TABLE | REBUILD {TAG \| EDGE} INDEX `<index_name>` [OFFLINE]

### 数据操作语言（DML）

数据操作语言（DML）用于操作数据库中的数据。

对比项                    | SQL                   | nGQL
-------------------------| ------------------------ | -----------
插入数据 | INSERT IGNORE INTO `<tbl_name>` [(col_name [, col_name] ...)] {VALUES \| VALUE} [(value_list) [, (value_list)] | INSERT VERTEX `<tag_name>` (prop_name_list[, prop_name_list]) {VALUES \| VALUE} vid: (prop_value_list[, prop_value_list]) <br/> INSERT EDGE `<edge_name>` ( `<prop_name_list>` ) VALUES \| VALUE `<src_vid>` -> `<dst_vid>`[`@<rank>`] : ( `<prop_value_list>` )
查询数据 | SELECT | GO, FETCH
更新数据 | UPDATE `<tbl_name>` SET field1=new-value1, field2=new-value2 [WHERE Clause] | UPDATE VERTEX `<vid>` SET `<update_columns>` [WHEN `<condition>`] <br/> UPDATE EDGE `<edge>` SET `<update_columns>` [WHEN `<condition>`]
删除数据 | DELETE FROM `<tbl_name>` [WHERE Clause] | DELETE EDGE `<edge_type>` `<vid>` -> `<vid>`[`@<rank>`] [, `<vid>` -> `<vid>` ...] <br/> DELETE VERTEX `<vid_list>`
拼接数据| JOIN | `|` |

### 数据查询语言（DQL）

数据查询语言（DQL）语句用于执行数据查询。本节说明如何使用 SQL 语句和 nGQL 语句查询数据。

```sql
SELECT
 [DISTINCT]
 select_expr [, select_expr] ...
 [FROM table_references]
 [WHERE where_condition]
 [GROUP BY {col_name | expr | position}]
 [HAVING  where_condition]
 [ORDER BY {col_name | expr | position} [ASC | DESC]]
```

```SQL
GO [[<M> TO] <N> STEPS ] FROM <node_list>
 OVER <edge_type_list> [REVERSELY] [BIDIRECT]
 [WHERE where_condition]
 [YIELD [DISTINCT] <return_list>]
 [| ORDER BY <expression> [ASC | DESC]]
 [| LIMIT [<offset_value>,] <number_rows>]
 [| GROUP BY {col_name | expr | position} YIELD <col_name>]

<node_list>
   | <vid> [, <vid> ...]
   | $-.id

<edge_type_list>
   edge_type [, edge_type ...]

<return_list>
    <col_name> [AS <col_alias>] [, <col_name> [AS <col_alias>] ...]
```

### 数据控制语言（DCL）

数据控制语言（DCL）包含诸如 `GRANT` 和 `REVOKE` 之类的命令，这些命令主要用来处理数据库系统的权限，其他控件。

对比项                    | SQL                   | nGQL
-------------------------| ------------------------ | -----------
创建用户 | CREATE USER | CREATE USER
删除用户 | DROP USER | DROP USER
更改密码 | SET PASSWORD | CHANGE PASSWORD
授予权限 | GRANT `<priv_type>` ON [object_type] TO `<user>`| GRANT ROLE `<role_type>` ON `<space>` TO `<user>`
删除权限 | REVOKE `<priv_type>` ON [object_type] TO `<user>` | REVOKE ROLE `<role_type>` ON `<space>` FROM `<user>`

## 数据模型

查询语句基于以下数据模型：

### MySQL

![image](../../figs/ng-ug-018.png)

### Nebula Graph

![image](../../figs/ng-ug-019.png)

## 增删改查（CRUD）

本节介绍如何使用 SQL 和 nGQL 语句创建（C）、读取（R）、更新（U）和删除（D）数据。

### 插入数据

```sql
mysql> INSERT INTO player VALUES (100, 'Tim Duncan', 42);

nebula> INSERT VERTEX player(name, age) VALUES 100: ('Tim Duncan', 42);
```

### 查询数据

Find the player whose id is 100 and output the `name` property:

```sql
mysql> SELECT player.name FROM player WHERE player.id = 100;

nebula> FETCH PROP ON player 100 YIELD player.name;
```

### 更新数据

```sql
mysql> UPDATE player SET name = 'Tim';

nebula> UPDATE VERTEX 100 SET player.name = "Tim";
```

### 删除数据

```sql
mysql> DELETE FROM player WHERE name = 'Tim';

nebula> DELETE VERTEX 121;
nebula> DELETE EDGE follow 100 -> 200;
```

## 示例查询

### 示例 1

返回年龄超过 36 岁的球员。

```sql
mysql> SELECT player.name
FROM player
WHERE player.age < 36;
```

使用 nGQL 查询有些不同，因为您必须在过滤属性之前创建索引。更多信息请参见 [索引文档](../2.query-language/4.statement-syntax/1.data-definition-statements/index.md)。

```ngql
nebula> CREATE TAG INDEX player_age ON player(age);
nebula> REBUILD TAG INDEX player_age OFFLINE;
nebula> LOOKUP ON player WHERE player.age < 36;
```

### 示例 2

查找球员 Tim Duncan 并返回他效力的所有球队。

```sql
mysql> SELECT a.id, a.name, c.name
FROM player a
JOIN serve b ON a.id=b.player_id
JOIN team c ON c.id=b.team_id
WHERE a.name = 'Tim Duncan';
```

```ngql
nebula> CREATE TAG INDEX player_name ON player(name);
nebula> REBUILD TAG INDEX player_name OFFLINE;
nebula> LOOKUP ON player WHERE player.name == 'Tim Duncan' YIELD player.name AS name | GO FROM $-.VertexID OVER serve YIELD $-.name, $$.team.name;
```

### 示例 3

查找球员 Tim Duncan 的队友。

```sql
mysql> SELECT a.id, a.name, c.name
FROM player a
JOIN serve b ON a.id=b.player_id
JOIN team c ON c.id=b.team_id
WHERE c.name IN (SELECT c.name
FROM player a
JOIN serve b ON a.id=b.player_id
JOIN team c ON c.id=b.team_id
WHERE a.name = 'Tim Duncan');
```

在 nGQL 中，我们使用管道将上一条语句的输出作为下一条语句的输入。

```ngql
nebula> GO FROM 100 OVER serve YIELD serve._dst AS Team | GO FROM $-.Team OVER serve REVERSELY YIELD $$.player.name;
```
