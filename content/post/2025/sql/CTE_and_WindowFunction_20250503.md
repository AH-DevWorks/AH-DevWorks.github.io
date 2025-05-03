---
Title: SQL - CTE(Common Table Expression)＆Window Function
Date: 2025-05-03
Tags: ["learning", "SQL", "CTE", "Window Function"]
Description: "SQL 自學筆記——CTE＆Window Function"
---
> 上課老師說時間不夠，這兩項目講不到，就當作延伸。
> 
> 自學後做了些紀錄。

---

#### 目錄
+ **[CTE(Common Table Expression)](#ctecommon-table-expression-一般資料表運算式)**
+ **[Window Function](#window-function-窗口函數視窗函數)**

---

## CTE(Common Table Expression) 一般資料表運算式

### 說明
+ CTE是 SQL 裡的一種臨時結果集
+ 「暫存」且「具名」的結果集合，透過`AS`關鍵字將查詢結果暫時儲存
+ 創建後可在SELECT、INSERT、UPDATE、DELETE等處使用，讓查詢邏輯更清晰、可讀性更高
+ `【基本語法】`
  ```sql
  WITH CTE名稱 AS (
      SELECT 查詢語句
  )
  SELECT *
  FROM CTE名稱;
  ```
+ 【基本範例1：查各部門平均薪資，按部門分類】
    ```sql
    -- CTE
    WITH cte_name AS (
        SELECT Department, AVG(Salary) AS AvgSalary
        FROM Employees
        GROUP BY Department
    )

    SELECT *
    FROM cte_name;  -- 使用上面創的CTE
    ```
+ 【基本範例2：查出薪資高於全體平均薪資的員工】
    ```sql
    WITH avg_salary AS (
        SELECT AVG(sal) AS avg_sal
        FROM emp
    )

    SELECT e.*
    FROM employees e CROSS JOIN avg_salary a
    WHERE e.salary > a.avg_sal;
    ```

### CTE 用於 Tree Recursion 樹狀結構遞迴
+ CTE適用在處理遞迴資料結構，或整理複雜查詢邏輯
+ *【範例：查出公司內部從老闆開始，往下每層的隸屬關係】*
    ```sql
    WITH RECURSIVE employee_hierarchy AS (
        -- 單只寫 NULL 會被推斷成字元長度太短，所以明確指定型別: CONVERT(NULL, CHAR(25))
        -- 這裡CONVERT()預設型別不能使用 VARCHAR --> MySQL本身的限制，能用固定size的CHAR，但可變長度的VARCHAR不能處理
        -- 加level只是方便把各員工所屬層級看得更清楚
        SELECT id, name, manager_id, CAST(NULL AS CHAR(25)) AS manager_name, 1 AS level
        FROM employees
        WHERE manager_id IS NULL    -- 初始層級：BOSS (沒有上級mgr(manager))

        UNION ALL

        -- 遞迴層級：找出每一層的下屬
        SELECT e.id, e.name, e.manager_id, h.name AS manager_name, h.level + 1  
        FROM employees e
        JOIN employee_hierarchy h ON e.manager_id = h.id
    )
    SELECT * FROM employee_hierarchy
    ORDER BY level;
    ```

### CTE vs. Subquery
| 項目   | CTE（Common Table Expression） | Subquery（子查詢）                   |
| ---- | ---------------------------- | ------------------------------- |
| 語法位置 | 使用 `WITH` 開頭，通常在主查詢之前        | 可以寫在 `FROM`、`WHERE`、`SELECT` 等處 |
| 可重用性 | 定義好的CTE可在同一個查詢中重複使用      | 無法重複使用；每次都要寫完整的查詢內容             |
| 可讀性  | 較佳，易於拆解複雜邏輯                  | 當巢狀太深時，可讀性較差                    |
| 支援遞迴 | ✅ 可使用 `WITH RECURSIVE`   | ❌ 不支援遞迴                       |
| 效能差異 | 理論上差不多，實際依資料庫優化器與查詢內容而異      | 一般查詢引擎也會優化 subquery 為臨時表格       |


---

+ *參考資料：[SQL語法 - CTE 一般資料表運算式（COMMON TABLE EXPRESSION）](https://vocus.cc/wo/66af87e8fd89780001717db8)*
 
---
---

## Window Function 窗口函數／視窗函數

### 說明
+ SQL 中的一種分析工具，可以針對每一列資料，計算其在某個「資料視窗（window）」中的統計結果，而不會將資料壓縮成單一列
+ 「在資料列上滑動『一個局部範圍』進行計算」（e.g.每個部門的薪資排名、累積總和等）
+ `【基本語法】`
    ```sql
    <Window_Function_name>(欄位) OVER (
        [PARTITION BY 分組欄位]
        [ORDER BY 排序欄位]
        [ROWS BETWEEN ...]
    )
    ```
+ `PARTITION BY`: 把資料按照某欄位分組形成多個「window」，然後在每一組中各自執行視窗函數邏輯
  + 跟GROUP BY不同：GROUP BY後資料筆數通常會減少；PARTITION BY後資料筆數不變、結合原本欄位。
    + GROUP BY不能搭配Window Function
  + PARTITION BY 可以省略 --> 表示「整張table為一個視窗」，window function 會對所有資料列共同計算（如下聚合型的範例2）

### 常見的 Window Function
+ **Aggregate-style 聚合型**：SUM(), COUNT(), AVG(), MAX(), MIN()
+ **Ranking 排序型**：ROW_NUMBER(), RANK(), DENSE_RANK()
+ **Value Offset 位移型**：LAG(), LEAD(), FIRST_VALUE(), LAST_VALUE()

#### 聚合型 Window Function
> 在一般查詢 / GROUP BY 語境下也可用，但在 Window Function語境下，不會壓縮rows
+ 【範例1：加總每個部門的銷售總額（每列都顯示）】
    ```sql
    SELECT employee_id, department, amount, SUM(amount) OVER (PARTITION BY department) AS total_in_dept
    FROM sales;
    -- 顯示每筆銷售及所在部門的總銷售金額
    ```
+ 【範例2：排序各員工薪資佔人事總成本的百分比】
    ```sql
    -- 省略PARTITION BY 的用法
    SELECT id, name, job, salary, CONCAT(ROUND(salary/SUM(salary) OVER() * 100, 2), '%') AS personnel_costs_percent
    FROM employee
    ORDER BY salary DESC; -- 高到低排序
    ```

#### 排序型 Window Function
+ **ROW_NUMBER()**: 給每一行一個不重複的序號，且`就算指定比對的數值一樣（如score都=1000），仍會排出高低次序`。
    ```sql
    SELECT *, ROW_NUMBER() OVER (ORDER BY salary DESC) AS sal_row_rank
    FROM employee;
    ```
  + 所以各 row 再多加明確排序欄位（如 `ROW_NUMBER() OVER (ORDER BY SALARY DESC, ID ASC)`）比較好 ==> 避免同值之間「非確定性」（nondeterministic）的先後順序。

+ **RANK()**: 相較於ROW_NUMBER()，RANK()會把同數值的項目給予同編號，且`若序號重複，往後序號會跳號` (ex. 1,2,2,4,5,...)
    ```sql
    SELECT *, RANK() OVER (ORDER BY salary DESC) AS sal_row_rank
    FROM employee;
    ```
+ **DENSE_RANK()**: 類似RANK()，但`重複不跳號`  (ex. 1,2,2,3,4,...)
```sql
    SELECT *, DENSE_RANK() OVER (ORDER BY salary DESC) AS sal_row_rank
    FROM employee;
```

+ **NTILE(n)**: （或歸類在分析型）分位排序，把資料依照參數(正整數n)拆成n組(tiles)，每組儘量平均放入數量儘量相同的資料、且依序給予序號
  + 常用在需要分等第/區間的情況
  + 若data總筆數不能被n整除，前幾組會多一筆資料
    ```sql
    -- 將所有員工依照薪水高到低排序，分成4組，並且標示每筆屬於哪一組
    SELECT employee_id, salary, NTILE(4) OVER (ORDER BY salary DESC) AS salary_quartile
    FROM employee;
    ```

#### 位移型 Window Function
> 這類函數會取排序之後的上一筆、下一筆或特定位置的值，方便做時間序列比較、成長率分析等
> 
> 透過 SELF JOIN 也能達成類似結果（但外觀看起來不太一樣）
+ `LAG() OVER (PARTITION BY ... ORDER BY ...)`：把資料往後移動，用來與上一列資料比較
	+ `LAG( [ 要位移的欄位 ], [ 位移列數 ], [ 沒資料時的預設值 ] )`
+ `LEAD() OVER (PARTITION BY ... ORDER BY ...)`: 把資料往前移動，用來與下一列資料比較
+ `FIRST_VALUE(column_name) OVER (PARTITION BY ... ORDER BY ...)`：查詢該分區第一列資料
+ `LAST_VALUE(column_name) OVER (PARTITION BY ... ORDER BY ...)`：查詢該分區最後一列資料
```sql
-- 找出各部門裡「最早錄用員工的錄用日期」，分別顯示在各部門員工欄位右側
SELECT name, DATE(hiredate), department_id, DATE(FIRST_VALUE(hiredate) OVER (PARTITION BY department_id)) AS dept_earliest_hiredate
FROM employee;
```

---

+ *參考資料： [SQL 窗口函數 Window Function：三大應用快速教學](https://haosquare.com/sql-window-function-intro/)*