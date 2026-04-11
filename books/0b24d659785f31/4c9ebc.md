---
title: "パフォーマンスの考慮"
free: false
---

# 問題X-X
- 実行計画を取得

### 期待する結果
| PLAN_TABLE_OUTPUT |
| - |
| Plan hash value: 2096651594 |
|  |
| -------------------------------------------------------------------------------------------------- |
|  | Id | Operation | Name | Rows | Bytes | Cost (%CPU) | Time |  |
| -------------------------------------------------------------------------------------------------- |
|  | 0 | SELECT STATEMENT |  | 2 | 70 | 2   (0) | 00:00:01 |  |
|  | 1 | TABLE ACCESS BY INDEX ROWID BATCHED | EMPLOYEES | 2 | 70 | 2   (0) | 00:00:01 |  |
|  | *  2 | INDEX RANGE SCAN | EMP_JOB_IX | 2 |  | 1   (0) | 00:00:01 |  |
| -------------------------------------------------------------------------------------------------- |
|  |
| Predicate Information (identified by operation id): |
| --------------------------------------------------- |
|  |
| 2 - access("JOB_ID"='AD_VP') |


### 解答例
```sql:sysdateで取得
explain PLAN for(
SELECT
    employee_id,
    first_name || ' ' || last_name as name,
    TO_CHAR(hire_date, 'YYYY-MM-DD')                   AS "HIRE_DATE",
    TO_CHAR(hire_date + 2, 'YYYY-MM-DD')               AS "HIRE_DATE + 2DAY",
    TO_CHAR(ADD_MONTHS(hire_date, 2), 'YYYY-MM-DD')    AS "HIRE_DATE + 2MONTH",
    TO_CHAR(ADD_MONTHS(hire_date, 2*12), 'YYYY-MM-DD') AS "HIRE_DATE + 2YEAR"
FROM
    hr.employees
WHERE
    job_id = 'AD_VP');

select * from table(dbms_xplan.display());
```

### 解説





### 参考リンク
https://www.shift-the-oracle.com/sql/functions/sysdate.html

----