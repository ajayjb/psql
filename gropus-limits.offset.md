# SQL: GROUP BY, LIMIT, and OFFSET

A reference for using aggregation and pagination clauses in PostgreSQL.

---

## GROUP BY

Groups rows that share the same values in one or more columns, typically used with aggregate functions.

### Syntax

```sql
SELECT column1, AGG_FUNC(column2)
FROM table_name
WHERE condition
GROUP BY column1;
```

### Aggregate Functions

| Function | Description |
|----------|-------------|
| `COUNT(*)` | Number of rows in the group |
| `SUM(col)` | Sum of values |
| `AVG(col)` | Average of values |
| `MIN(col)` | Minimum value |
| `MAX(col)` | Maximum value |
| `ARRAY_AGG(col)` | Aggregates values into an array |
| `STRING_AGG(col, sep)` | Concatenates strings with a separator |

### Examples

```sql
-- Count orders per customer
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id;

-- Total revenue per region
SELECT region, SUM(amount) AS total_revenue
FROM sales
GROUP BY region
ORDER BY total_revenue DESC;

-- Multiple grouping columns
SELECT region, product_category, AVG(amount) AS avg_sale
FROM sales
GROUP BY region, product_category;
```

### HAVING

Filter groups after aggregation (analogous to `WHERE` for rows).

```sql
-- Only customers with more than 5 orders
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 5;
```

> **Note:** `WHERE` filters rows before grouping; `HAVING` filters groups after aggregation.

### GROUP BY with expressions

```sql
-- Group by derived value
SELECT DATE_TRUNC('month', created_at) AS month, COUNT(*) AS signups
FROM users
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY month;
```

### ROLLUP and CUBE (PostgreSQL)

```sql
-- ROLLUP: subtotals + grand total
SELECT region, product, SUM(amount)
FROM sales
GROUP BY ROLLUP(region, product);

-- CUBE: all combinations of subtotals
SELECT region, product, SUM(amount)
FROM sales
GROUP BY CUBE(region, product);
```

---

## LIMIT

Restricts the number of rows returned by a query.

### Syntax

```sql
SELECT columns
FROM table_name
LIMIT n;
```

### Examples

```sql
-- Return only the top 10 rows
SELECT * FROM products
ORDER BY price DESC
LIMIT 10;

-- Always pair LIMIT with ORDER BY for deterministic results
SELECT id, name, created_at
FROM users
ORDER BY created_at DESC
LIMIT 5;
```

> **Warning:** Without `ORDER BY`, the rows returned by `LIMIT` are non-deterministic — the database may return different rows across executions.

---

## OFFSET

Skips a specified number of rows before returning results. Used with `LIMIT` for pagination.

### Syntax

```sql
SELECT columns
FROM table_name
LIMIT n OFFSET m;
```

### Examples

```sql
-- Page 1: rows 1–10
SELECT * FROM products ORDER BY id LIMIT 10 OFFSET 0;

-- Page 2: rows 11–20
SELECT * FROM products ORDER BY id LIMIT 10 OFFSET 10;

-- Page N formula: OFFSET = (page - 1) * page_size
```

### Pagination Pattern

```sql
-- For page_num (1-indexed) and page_size
SELECT *
FROM orders
ORDER BY created_at DESC
LIMIT :page_size OFFSET ((:page_num - 1) * :page_size);
```

---

## Combining GROUP BY, LIMIT, and OFFSET

```sql
-- Top 5 products by total revenue, with pagination
SELECT
    product_id,
    SUM(quantity * unit_price) AS revenue
FROM order_items
GROUP BY product_id
ORDER BY revenue DESC
LIMIT 5 OFFSET 0;
```

---

## Performance Considerations

### GROUP BY

- Ensure columns in `GROUP BY` are indexed if filtering with `WHERE` on those columns.
- Aggregations over large tables benefit from partial indexes or materialized views.
- `HAVING` cannot use indexes directly — pre-filter with `WHERE` where possible.

### LIMIT / OFFSET

- `OFFSET` does **not** skip work — the database still scans and discards the offset rows.
- For large offsets, use **keyset pagination** instead:

```sql
-- Keyset pagination (cursor-based) — much more efficient for large datasets
SELECT * FROM orders
WHERE created_at < :last_seen_cursor
ORDER BY created_at DESC
LIMIT 10;
```

| Method | Pros | Cons |
|--------|------|------|
| `LIMIT/OFFSET` | Simple, supports random page access | Slow for large offsets, inconsistent with concurrent writes |
| Keyset / cursor | Fast at any depth, stable with writes | No random page access, requires a sortable cursor column |

---

## Quick Reference

```sql
-- Aggregation with filter and pagination
SELECT
    department,
    COUNT(*)       AS headcount,
    AVG(salary)    AS avg_salary
FROM employees
WHERE active = true
GROUP BY department
HAVING COUNT(*) >= 3
ORDER BY avg_salary DESC
LIMIT 10 OFFSET 20;
```

**Clause execution order:**

```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT / OFFSET
```
