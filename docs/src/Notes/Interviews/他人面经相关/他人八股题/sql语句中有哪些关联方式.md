### 来自网络面经

#### sql语句中有哪些关联方式

1. 内连接
仅返回两个表中满足连接条件（一般是键相等）的行，也称“交集”

```sql
SELECT A.*, B.*
FROM A
INNER JOIN B ON A.id = B.a_id;
```

2. 外连接
包括**左、右、全**三种形式，处理匹配不上时的情况。

- 左外连接
返回左表所有行，若右表无匹配则对应列为 NULL
```sql
SELECT A.*, B.*
FROM A
LEFT JOIN B ON A.id = B.a_id;
```

- 右外连接
返回右表所有行，左表若无匹配则为 NULL。
```sql
SELECT A.*, B.*
FROM A
RIGHT JOIN B ON A.id = B.a_id;
```

- 全外连接
返回两表中所有行，匹配不上的一边用 NULL 填充。
```sql
SELECT A.*, B.*
FROM A
FULL OUTER JOIN B ON A.id = B.a_id;
```

3. 交叉连接
返回笛卡尔积：即左表每行与右表每行两两配对，总数 = 行数相乘。
```sql
SELECT *
FROM A
CROSS JOIN B;
```

4. 自然连接
自动找出两个表中同名的列作为连接条件，无需显式指定 ON。
一般不推荐使用

5. 自连接
用于表内部关联自身，一般通过别名实现，便于处理如层级结构（例如员工与其管理者）等需求。
```sql
SELECT e1.name AS employee, e2.name AS manager
FROM employees e1
JOIN employees e2 ON e1.manager_id = e2.id;
```

6. 联合查询
虽然不属于 JOIN，但常用于组合来自多表或实现全外连接便利。

关联方式一览表
| 关联方式              | 返回内容说明                    | 示例场景               |
| ----------------- | ------------------------- | ------------------ |
| INNER JOIN        | 两表都匹配上的行                  | 查询同时存在用户和订单的记录     |
| LEFT OUTER JOIN   | 左表所有 + 匹配右表的（右表不匹配则 NULL） | 查询所有用户及其订单（可能无订单）  |
| RIGHT OUTER JOIN  | 右表所有 + 匹配左表的（左表不匹配则 NULL） | 查询所有订单及其用户（可能用户缺失） |
| FULL OUTER JOIN   | 两表所有行（匹配则列显示，否则 NULL）     | 全部用户与全部订单情况（兼顾）    |
| CROSS JOIN        | 笛卡尔积（全组合）                 | 生成所有组合（如产品 × 配色）   |
| NATURAL JOIN      | 根据同名列自动匹配                 | 简写方便但易错（不推荐）       |
| SELF JOIN         | 表与自身关联（技巧）                | 查询员工的上级信息          |
| UNION / UNION ALL | 多表横向合并（非 JOIN）            | 合并不同来源数据           |
