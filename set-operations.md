# PostgreSQL Set Operations

Set operations combine the results of two or more `SELECT` queries into a single result set. Both queries must return the **same number of columns** with **compatible data types**.

---

## Overview

| Operator | Duplicates | NULLs Compared |
|---|---|---|
| `UNION` | Removed | Yes (NULL = NULL) |
| `UNION ALL` | Kept | Yes |
| `INTERSECT` | Removed | Yes |
| `INTERSECT ALL` | Kept (min count) | Yes |
| `EXCEPT` | Removed | Yes |
| `EXCEPT ALL` | Kept (subtracted count) | Yes |

---

## Setup (Sample Tables)

```sql
CREATE TABLE engineers (name TEXT, dept TEXT);
CREATE TABLE managers  (name TEXT, dept TEXT);

INSERT INTO engineers VALUES
  ('Alice', 'Backend'),
  ('Bob',   'Backend'),
  ('Carol', 'Frontend'),
  ('Alice', 'Backend');   -- duplicate

INSERT INTO managers VALUES
  ('Alice', 'Backend'),
  ('Dave',  'Ops'),
  ('Carol', 'Frontend');
```

---

## 1. UNION

Returns all rows from both queries, **removing duplicates**.

```sql
SELECT name, dept FROM engineers
UNION
SELECT name, dept FROM managers;
```

**Result:**

| name  | dept     |
|-------|----------|
| Alice | Backend  |
| Bob   | Backend  |
| Carol | Frontend |
| Dave  | Ops      |

> The duplicate `('Alice', 'Backend')` row in `engineers` is collapsed into one.

---

## 2. UNION ALL

Returns all rows from both queries, **keeping duplicates**.

```sql
SELECT name, dept FROM engineers
UNION ALL
SELECT name, dept FROM managers;
```

**Result:**

| name  | dept     |
|-------|----------|
| Alice | Backend  |
| Bob   | Backend  |
| Carol | Frontend |
| Alice | Backend  | ← duplicate from engineers
| Alice | Backend  | ← from managers
| Dave  | Ops      |
| Carol | Frontend |

> Faster than `UNION` — no deduplication pass. Prefer `UNION ALL` when you know rows are distinct or duplicates are acceptable.

---

## 3. INTERSECT

Returns only rows that appear in **both** result sets, **removing duplicates**.

```sql
SELECT name, dept FROM engineers
INTERSECT
SELECT name, dept FROM managers;
```

**Result:**

| name  | dept     |
|-------|----------|
| Alice | Backend  |
| Carol | Frontend |

> Only rows present in both tables are returned.

---

## 4. INTERSECT ALL

Returns rows present in **both** result sets, **preserving duplicates** up to the **minimum occurrence count** across both sides.

```sql
SELECT name, dept FROM engineers
INTERSECT ALL
SELECT name, dept FROM managers;
```

**Result:**

| name  | dept     |
|-------|----------|
| Alice | Backend  |
| Carol | Frontend |

> `('Alice', 'Backend')` appears twice in `engineers` and once in `managers` → min(2,1) = 1 row returned.

**When counts differ:**

```sql
-- engineers has 2x ('Alice','Backend'), managers has 1x → returns 1 row
-- engineers has 3x some row, managers has 2x → returns 2 rows
```

---

## 5. EXCEPT

Returns rows from the **first** query that do **not** appear in the second query, **removing duplicates**.

```sql
SELECT name, dept FROM engineers
EXCEPT
SELECT name, dept FROM managers;
```

**Result:**

| name | dept    |
|------|---------|
| Bob  | Backend |

> `Alice` and `Carol` exist in `managers`, so they are excluded. Only `Bob` remains.

> **Order matters:** `A EXCEPT B` ≠ `B EXCEPT A`.

---

## 6. EXCEPT ALL

Returns rows from the **first** query not in the second, **preserving duplicates** by subtracting counts.

```sql
SELECT name, dept FROM engineers
EXCEPT ALL
SELECT name, dept FROM managers;
```

**Result:**

| name  | dept    |
|-------|---------|
| Bob   | Backend |
| Alice | Backend |

> `('Alice', 'Backend')` appears twice in `engineers` and once in `managers` → 2 - 1 = 1 row kept.
> `('Carol', 'Frontend')` appears once in each → 1 - 1 = 0 rows kept.

---

## Ordering and Limiting Results

`ORDER BY` and `LIMIT` apply to the **combined** result. Use column position or alias — column names come from the **first** SELECT:

```sql
SELECT name, dept FROM engineers
UNION ALL
SELECT name, dept FROM managers
ORDER BY name
LIMIT 5;
```

To sort by column name, reference the **first query's** column name:

```sql
SELECT name AS person, dept FROM engineers
UNION
SELECT name, dept FROM managers
ORDER BY person;
```

---

## Precedence

When mixing operators, `INTERSECT` binds more tightly than `UNION` / `EXCEPT`. Use parentheses to be explicit:

```sql
-- INTERSECT evaluated first:
SELECT ... UNION SELECT ... INTERSECT SELECT ...

-- Force UNION first:
(SELECT ... UNION SELECT ...) INTERSECT SELECT ...
```

---

## Performance Notes

| Concern | Recommendation |
|---|---|
| Avoid dedup overhead | Use `UNION ALL` / `INTERSECT ALL` / `EXCEPT ALL` where duplicates are acceptable |
| Large result sets | `UNION ALL` is O(n); `UNION` requires a hash/sort pass |
| Index usage | Each sub-query can use indexes independently |
| CTEs with set ops | Wrap complex queries in a `WITH` clause for readability |

---

## Quick Cheat Sheet

```
A UNION B         → A + B, no duplicates
A UNION ALL B     → A + B, with duplicates

A INTERSECT B     → rows in both A and B, no duplicates
A INTERSECT ALL B → rows in both A and B, duplicate count = min(count_A, count_B)

A EXCEPT B        → rows in A not in B, no duplicates
A EXCEPT ALL B    → rows in A not in B, duplicate count = count_A - count_B (floor 0)
```
