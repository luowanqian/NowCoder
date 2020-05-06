解答 Part 1

- [1. 分页查询employees表，每5行一页，返回第2页的数据](#1-分页查询employees表每5行一页返回第2页的数据)
- [2. 获取select * from employees对应的执行计划](#2-获取select--from-employees对应的执行计划)
- [3. 如何获取emp_v和employees有相同的数据no](#3-如何获取emp_v和employees有相同的数据no)
- [4. 获取employees中的行数据，且这些行也存在于emp_v中](#4-获取employees中的行数据且这些行也存在于emp_v中)
- [5. 查找字符串'10,A,B'中逗号','出现的次数cnt](#5-查找字符串10ab中逗号出现的次数cnt)
- [6. 获取Employees中的first_name，查询按照first_name最后两个字母，按照升序进行排列](#6-获取employees中的first_name查询按照first_name最后两个字母按照升序进行排列)
- [7. 将id=5以及emp_no=10001的行数据替换成id=5以及emp_no=10005,其他数据保持不变，使用replace实现](#7-将id5以及emp_no10001的行数据替换成id5以及emp_no10005其他数据保持不变使用replace实现)
- [8. 将titles_test表名修改为titles_2017](#8-将titles_test表名修改为titles_2017)
- [9. 删除emp_no重复的记录，只保留最小的id对应的记录](#9-删除emp_no重复的记录只保留最小的id对应的记录)
- [10. 按照dept_no进行汇总，属于同一个部门的emp_no按照逗号进行连接，结果给出dept_no以及连接出的结果employees](#10-按照dept_no进行汇总属于同一个部门的emp_no按照逗号进行连接结果给出dept_no以及连接出的结果employees)
- [11. 从titles表获取按照title进行分组](#11-从titles表获取按照title进行分组)
- [12. 将所有to_date为9999-01-01的全部更新为NULL,且from_date更新为2001-01-01](#12-将所有to_date为9999-01-01的全部更新为null且from_date更新为2001-01-01)
- [13. 将employees表中的所有员工的last_name和first_name通过(')连接起来](#13-将employees表中的所有员工的last_name和first_name通过连接起来)
- [14. 批量插入数据，不使用replace操作](#14-批量插入数据不使用replace操作)
  - [14.1. Solution 1](#141-solution-1)
  - [14.2. Solution 2](#142-solution-2)
- [15. 针对actor表创建视图actor_name_view](#15-针对actor表创建视图actor_name_view)
- [16. 找出所有员工当前薪水salary情况](#16-找出所有员工当前薪水salary情况)
  - [16.1. Solution 1](#161-solution-1)
  - [16.2. Solution 2](#162-solution-2)

## 1. 分页查询employees表，每5行一页，返回第2页的数据

```sql
SELECT * FROM employees LIMIT 5 OFFSET 5;
```

## 2. 获取select * from employees对应的执行计划

```sql
EXPLAIN SELECT * FROM employees
```

## 3. 如何获取emp_v和employees有相同的数据no

```sql
SELECT e1.* FROM emp_v e1, employees e2 WHERE e1.emp_no = e2.emp_no;
```

## 4. 获取employees中的行数据，且这些行也存在于emp_v中

```sql
SELECT e2.* FROM emp_v e1, employees e2 WHERE e1.emp_no = e2.emp_no;
```

## 5. 查找字符串'10,A,B'中逗号','出现的次数cnt

```sql
SELECT LENGTH('10,A,B') - LENGTH(REPLACE('10,A,B', ',', '')) as cnt
```

## 6. 获取Employees中的first_name，查询按照first_name最后两个字母，按照升序进行排列

```sql
SELECT first_name FROM employees ORDER BY SUBSTR(first_name, LENGTH(first_name)-1);
```

使用 `substr()` 函数提取字符串最后两位字符

## 7. 将id=5以及emp_no=10001的行数据替换成id=5以及emp_no=10005,其他数据保持不变，使用replace实现

```sql
UPDATE titles_test SET emp_no = REPLACE(emp_no, 10001, 10005) WHERE id = 5;
```

## 8. 将titles_test表名修改为titles_2017

```sql
ALTER TABLE titles_test RENAME TO titles_2017;
```

## 9. 删除emp_no重复的记录，只保留最小的id对应的记录

```sql
DELETE FROM titles_test
WHERE id NOT IN (SELECT MIN(id) FROM titles_test GROUP BY emp_no);
```

解决思路：首先是找出 `emp_no` 重复记录中 `id` 最小的那个，此时可以根据 `emp_no` 进行分组，选出每个分组中最小的 `id`

```sql
SELECT MIN(id) FROM titles_test GROUP BY emp_no
```

然后是删除非前面得到的 `id` 的记录，使用 `id NOT IN` 语法即可

## 10. 按照dept_no进行汇总，属于同一个部门的emp_no按照逗号进行连接，结果给出dept_no以及连接出的结果employees

```sql
SELECT dept_no, GROUP_CONCAT(emp_no) as employees FROM dept_emp GROUP BY dept_no;
```

使用SQLite的 `GROUP_CONCAT()` 函数

## 11. 从titles表获取按照title进行分组

```sql
SELECT title, COUNT(*) as t FROM titles GROUP BY title HAVING COUNT(*) >= 2;
```

## 12. 将所有to_date为9999-01-01的全部更新为NULL,且from_date更新为2001-01-01

```sql
UPDATE titles_test SET to_date = NULL, from_date = '2001-01-01' WHERE to_date = '9999-01-01';
```

## 13. 将employees表中的所有员工的last_name和first_name通过(')连接起来

```sql
SELECT last_name || '''' || first_name AS name FROM employees;
```

SQLite字段的拼接是 `||`

## 14. 批量插入数据，不使用replace操作

### 14.1. Solution 1

```sql
INSERT INTO actor
SELECT '3', 'ED', 'CHASE', '2006-02-15 12:34:33'
WHERE NOT EXISTS (SELECT * FROM actor WHERE actor_id = '3');
```

Reference: [StackOverflow: “Insert if not exists” statement in SQLite](https://stackoverflow.com/questions/19337029/insert-if-not-exists-statement-in-sqlite)

### 14.2. Solution 2

```sql
INSERT OR IGNORE INTO actor VALUES('3', 'ED', 'CHASE', '2006-02-15 12:34:33');
```

* `INSERT OR IGNORE INTO tablename VALUES(...);`

    如果不存在则插入，如果存在则忽略

* `INSERT OR REPLACE INTO tablename VALUES(...);`

    如果不存在则插入，如果存在则替换

这里`存在`表示的是unique属性的列值存在的情况下，unique表示键值唯一

## 15. 针对actor表创建视图actor_name_view

```sql
CREATE VIEW actor_name_view AS
SELECT first_name AS first_name_v, last_name AS last_name_v
FROM actor;
```

## 16. 找出所有员工当前薪水salary情况

### 16.1. Solution 1

```sql
SELECT DISTINCT salary FROM salaries WHERE to_date = '9999-01-01' ORDER BY salary DESC;
```

### 16.2. Solution 2

```sql
SELECT salary FROM salaries WHERE to_date = '9999-01-01' GROUP BY salary ORDER BY salary DESC;
```