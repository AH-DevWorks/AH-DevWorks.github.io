---
Title: SQL Test - Review
Date: 2025-05-04
Tags: ["learning", "sql"]
Description: "重點觀念複習"
draft: false
---

### ❌ 所有的 DML 都可以在 View 上操作
+ **Simple View** ==> 滿足下列條件，才可以用DML
  + 只有一個Base Table，沒有JOIN
  + 沒有使用Function(SUM, AVG等)或分組(GROUP BY, DISTINCT 等)
  + Updateable 可更新
+ **Complex View**
  + <b><u>NO DML</u></b>

---

### ✅ Foreign Key 可由多個欄位組成
+ 可以由多個欄位組成複合外鍵（Composite Foreign Key），只要這些欄位對應參照表的一組 PK 或 UK 即可
+ p.s. FK可以是NULL空值

---

### DDL (Data Definition Language)
+ CREATE, DROP, ALTER, RENAME, TRUNCATE

### DML (Data Manipulation Language)
+ INSERT, UPDATE, DELETE, *SELECT
+ *註: 亦有分類將 SELECT 單獨歸類為 DQL (Data Query Language)

### DCL (Data Control Language)
+ GRANT, REVOKE

### TCL (Transaction Control Language)
+ COMMIT, ROLLBACK, SAVEPOINT

---

### SQL 語法順序 ≠ 實際執行順序

#### 語法順序ex.
```sql
SELECT name, COUNT(*) as order_count
FROM orders
WHERE status = 'completed'
GROUP BY name
HAVING COUNT(*) > 5
ORDER BY order_count DESC
LIMIT 10;
```

#### 實際執行順序 Logical Execution Order
```sql
FROM        -- 1.
WHERE       -- 2.
GROUP BY    -- 3.
HAVING      -- 4.
SELECT      -- 5.※
ORDER BY    -- 6.所以ORDER BY 裡可用 SELECT 定義的 column別名
LIMIT       -- 7.
```

#### ⚠ MySQL 實作上會提前解析 SELECT 子句中的別名：可看成SELECT在第3順位
```sql
SELECT employee_name Name, department_id D_ID, MAX(salary) M_sal 
FROM employees
WHERE hiredate > '2011-02-01'
GROUP BY Name, D_ID         -- 可以用別名Name, D_ID
HAVING M_sal > 1600         -- 可以用別名 m_sal
ORDER BY Name
LIMIT 3;

''' ↓ SELECT 提前解析 '''

FROM
WHERE
SELECT   -- <-- ※
GROUP BY -- 以下幾個都可用column別名
HAVING
ORDER BY
LIMIT
```