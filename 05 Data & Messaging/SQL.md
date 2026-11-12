# SQL

---
tags: [sql, database]
---

> [!note] Confidence check
> Only "SQL query writing" is confirmed-likely (from another candidate's account). Clustered/non-clustered indexing is a *reasonable guess*, not confirmed — know the basics, don't over-invest.

## SQL — the short version

Execution order (not writing order!): `FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY`

```sql
-- JOIN: combine two tables on a matching key
SELECT e.name, d.name
FROM employees e
JOIN departments d ON e.dept_id = d.id;

-- GROUP BY + HAVING: aggregate, then filter the aggregate
SELECT dept_id, COUNT(*) AS headcount
FROM employees
GROUP BY dept_id
HAVING COUNT(*) > 3;         -- HAVING filters groups, WHERE can't (no aggregate yet at WHERE time)

-- Top-per-group: GROUP BY loses the row (can't SELECT name). Use a window function:
SELECT name, dept_id, salary
FROM (
  SELECT name, dept_id, salary,
         RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
  FROM employees
) x
WHERE rnk = 1;
```

> [!warning] WHERE vs HAVING
> WHERE = filters rows, before grouping, no aggregates. HAVING = filters groups, after grouping, aggregates OK.

> Mnemonic: **GROUP BY collapses rows. Window functions (PARTITION BY) rank rows without losing them.**
