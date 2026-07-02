## SQL 常见命令动词总览

| 命令动词       | 含义   | 常见用途          |
| ---------- | ---- | ------------- |
| `CREATE`   | 创建   | 创建数据库、表、视图、索引 |
| `DROP`     | 删除   | 删除数据库对象       |
| `ALTER`    | 修改结构 | 修改表结构、字段、约束   |
| `SELECT`   | 查询   | 从表中查询数据       |
| `INSERT`   | 插入   | 向表中加入新元组      |
| `UPDATE`   | 更新   | 修改已有数据        |
| `DELETE`   | 删除数据 | 删除表中的元组       |
| `GRANT`    | 授权   | 给用户权限         |
| `REVOKE`   | 收回权限 | 撤销用户权限        |
| `COMMIT`   | 提交事务 | 永久保存事务结果      |
| `ROLLBACK` | 回滚事务 | 撤销事务中的操作      |

-----

## 常见命令动词含义

`CREATE`：创建数据库对象。

```sql
create table Student (
  Sno char(10),
  Sname varchar(20)
);
```

`DROP`：删除数据库对象，通常会删除结构和其中的数据。

```sql
drop table Student;
```

`ALTER`：修改已有表结构。

```sql
alter table Student add Age int;
```

`SELECT`：查询数据。

```sql
select Sno, Sname
from Student;
```

`INSERT`：插入新数据。

```sql
insert into Student(Sno, Sname)
values ('001', '张三');
```

`UPDATE`：修改已有数据。

```sql
update Student
set Sname = '李四'
where Sno = '001';
```

`DELETE`：删除表中满足条件的数据。

```sql
delete from Student
where Sno = '001';
```

`GRANT`：授予权限。

```sql
grant select on Student to user1;
```

`REVOKE`：收回权限。

```sql
revoke select on Student from user1;
```

`COMMIT`：提交事务，使修改正式生效。

```sql
commit;
```

`ROLLBACK`：回滚事务，撤销尚未提交的修改。

```sql
rollback;
```

---

## 游标操作命令总览

| 命令           | 含义        |
| ------------ | --------- |
| `DECLARE`    | 定义游标      |
| `OPEN`       | 打开游标，执行查询 |
| `FETCH`      | 取出当前一行数据  |
| `CLOSE`      | 关闭游标      |
| `DEALLOCATE` | 释放游标资源    |

---

## `DECLARE`

定义游标，把一个查询结果集绑定到游标上。

```sql
declare cursor_name cursor for
select Sno, Sname
from Student;
```

> 游标本质上是“指向查询结果集的一根指针”。

---

## `OPEN`

打开游标，执行游标对应的查询，生成结果集。

```sql
open cursor_name;
```

> 打开后，游标通常位于第一行之前。

---

## `FETCH`

从游标中读取一行数据，并把游标移动到下一行。

```sql
fetch cursor_name into variable1, variable2;
```

例如：

```sql
fetch cursor_name into v_sno, v_sname;
```

> 每执行一次 `FETCH`，通常取一行。

---

## `CLOSE`

关闭游标，释放当前结果集，但游标定义还在。

```sql
close cursor_name;
```

> 关闭后不能继续 `FETCH`，除非重新 `OPEN`。

---

## `DEALLOCATE`

释放游标定义及相关资源。

```sql
deallocate cursor_name;
```

> `CLOSE` 是关掉结果集，`DEALLOCATE` 是彻底释放游标。

---

## 基本使用顺序

```sql
declare cursor_name cursor for 查询语句;
open cursor_name;
fetch cursor_name into 变量;
close cursor_name;
deallocate cursor_name;
```
---
## 查询谓词总览

| 谓词                    | 含义            | 例子                         |
| --------------------- | ------------- | -------------------------- |
| `BETWEEN ... AND ...` | 判断是否在某个范围内    | `grade between 60 and 100` |
| `IN`                  | 判断是否属于某个集合    | `dept in ('CS', 'MA')`     |
| `LIKE`                | 字符串模式匹配       | `name like '张%'`           |
| `IS NULL`             | 判断是否为空值       | `phone is null`            |
| `EXISTS`              | 判断子查询结果是否非空   | `exists (select ...)`      |
| `ANY` / `SOME`        | 与子查询结果中某一个值比较 | `grade > any (...)`        |
| `ALL`                 | 与子查询结果中所有值比较  | `grade >= all (...)`       |

