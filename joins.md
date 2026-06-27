# PostgreSQL JOINs Reference

A practical guide to all JOIN types using the `users`, `photos`, and `comments` schema.

---

## Schema Overview

```sql
users    (id, username)
photos   (id, url, user_id → users.id)   -- user_id can be NULL
comments (id, contents, user_id → users.id, photo_id → photos.id)
```

> Note: `photos` has one row with `user_id = NULL`, which makes it ideal for demonstrating outer join behaviour.

---

## 1. INNER JOIN

Returns **only rows that have a match in both tables**. Non-matching rows are dropped entirely.

```
Table A   Table B
  1    ──►  1   ✓ included
  2    ──►  2   ✓ included
  3    ✗         ✗ dropped (no match)
       4   ✗     ✗ dropped (no match)
```

### Syntax

```sql
SELECT <columns>
FROM <left_table>
[INNER] JOIN <right_table> ON <left_table.col> = <right_table.col>;
```

The `INNER` keyword is optional — `JOIN` alone means `INNER JOIN`.

### Example — comments with their photo URLs

```sql
SELECT comments.contents, photos.url
FROM comments
JOIN photos ON photos.id = comments.photo_id;
```

Every comment must have a matching photo. If a comment had a `photo_id` pointing to a non-existent photo it would be excluded.

### Example — comments where commenter also owns the photo

```sql
SELECT comments.contents
FROM comments
JOIN photos  ON comments.photo_id = photos.id
JOIN users   ON users.id = comments.user_id
           AND users.id = photos.user_id;
```

The extra `AND` condition on the second join means only rows where the same user both wrote the comment and owns the photo survive.

---

## 2. LEFT JOIN  *(LEFT OUTER JOIN)*

Returns **all rows from the left table** plus matched rows from the right table. Where there is no match, right-table columns come back as `NULL`.

```
Table A   Table B
  1    ──►  1   ✓ included (match)
  2    ──►  2   ✓ included (match)
  3    ✗         ✓ included (NULL for B columns)
       4   ✗     ✗ dropped
```

### Syntax

```sql
SELECT <columns>
FROM <left_table>
LEFT [OUTER] JOIN <right_table> ON <left_table.col> = <right_table.col>;
```

### Example — all users and any photos they've posted

```sql
SELECT users.username, photos.url
FROM users
LEFT JOIN photos ON photos.user_id = users.id;
```

Users with no photos still appear; their `url` column will be `NULL`.

### Example — users who have never posted a photo

```sql
SELECT users.username
FROM users
LEFT JOIN photos ON photos.user_id = users.id
WHERE photos.id IS NULL;
```

Filtering on a right-table column being `NULL` after a LEFT JOIN is the standard pattern for "find rows with no match".

---

## 3. RIGHT JOIN  *(RIGHT OUTER JOIN)*

The mirror image of LEFT JOIN. Returns **all rows from the right table** plus matched rows from the left table. Where there is no match, left-table columns are `NULL`.

```
Table A   Table B
  1    ──►  1   ✓ included (match)
  2    ──►  2   ✓ included (match)
             3  ✓ included (NULL for A columns)
  4    ✗        ✗ dropped
```

### Syntax

```sql
SELECT <columns>
FROM <left_table>
RIGHT [OUTER] JOIN <right_table> ON <left_table.col> = <right_table.col>;
```

### Example — all photos and the user who posted each (including the photo with no owner)

```sql
SELECT photos.url, users.username
FROM users
RIGHT JOIN photos ON photos.user_id = users.id;
```

The photo with `user_id = NULL` (the last inserted row, `http://sasha.com`) will appear with `username = NULL`.

> **Tip:** A RIGHT JOIN can always be rewritten as a LEFT JOIN by swapping the table order. Most developers prefer LEFT JOIN for readability and consistency.

```sql
-- Equivalent
SELECT photos.url, users.username
FROM photos
LEFT JOIN users ON photos.user_id = users.id;
```

---

## 4. FULL JOIN  *(FULL OUTER JOIN)*

Returns **all rows from both tables**. Where there is no match on either side, the missing columns are `NULL`.

```
Table A   Table B
  1    ──►  1   ✓ included (match)
  2    ──►  2   ✓ included (match)
  3    ✗         ✓ included (NULL for B columns)
       4   ✗     ✓ included (NULL for A columns)
```

### Syntax

```sql
SELECT <columns>
FROM <left_table>
FULL [OUTER] JOIN <right_table> ON <left_table.col> = <right_table.col>;
```

### Example — every user and every photo, matched where possible

```sql
SELECT users.username, photos.url
FROM users
FULL JOIN photos ON photos.user_id = users.id;
```

- Users with no photos → `url` is `NULL`  
- The photo with no owner → `username` is `NULL`  
- Matched pairs → both columns populated

### Example — finding unmatched rows on either side

```sql
SELECT users.username, photos.url
FROM users
FULL JOIN photos ON photos.user_id = users.id
WHERE users.id IS NULL OR photos.id IS NULL;
```

This surfaces users who've never posted and photos that have no owner in one query.

---

## Quick Comparison

| JOIN type   | Keeps unmatched left rows | Keeps unmatched right rows |
|-------------|:-------------------------:|:--------------------------:|
| INNER JOIN  | ✗                         | ✗                          |
| LEFT JOIN   | ✓ (NULLs on right)        | ✗                          |
| RIGHT JOIN  | ✗                         | ✓ (NULLs on left)          |
| FULL JOIN   | ✓ (NULLs on right)        | ✓ (NULLs on left)          |

---

## Multi-table JOINs

JOINs chain left-to-right. Each successive JOIN operates on the result set produced so far.

```sql
-- All three tables joined: comment text, who wrote it, which photo it's on
SELECT
  comments.id   AS comment_id,
  users.id      AS user_id,
  photos.id     AS photo_id
FROM comments
JOIN users  ON comments.user_id  = users.id
JOIN photos ON comments.photo_id = photos.id;
```

You can mix JOIN types in one query — for example, INNER JOIN to users (every comment must have an author) and LEFT JOIN to photos (keep comments even if the photo was deleted):

```sql
SELECT comments.contents, users.username, photos.url
FROM comments
JOIN      users  ON comments.user_id  = users.id
LEFT JOIN photos ON comments.photo_id = photos.id;
```

---

## Common Pitfalls

**Cartesian product / missing ON clause**  
Omitting the `ON` condition (or using a `CROSS JOIN`) multiplies every row from one table against every row from the other. With 5 users and 22 photos that's 110 rows — almost never what you want.

```sql
-- Dangerous: returns 5 × 22 = 110 rows
SELECT * FROM users, photos;
```

**Filtering a LEFT JOIN in WHERE instead of ON**  
Putting a right-table condition in `WHERE` silently converts a LEFT JOIN into an INNER JOIN because `NULL` rows fail the filter.

```sql
-- BUG: loses users with no photos
SELECT users.username, photos.url
FROM users
LEFT JOIN photos ON photos.user_id = users.id
WHERE photos.url LIKE 'https%';       -- NULL rows filtered out

-- FIX: move the condition into ON
SELECT users.username, photos.url
FROM users
LEFT JOIN photos ON  photos.user_id = users.id
                 AND photos.url LIKE 'https%';
```

**Ambiguous column names**  
When two joined tables share a column name (e.g. both have `id`), always qualify with the table name or an alias.

```sql
-- Ambiguous:
SELECT id FROM comments JOIN photos ON photos.id = comments.photo_id;

-- Clear:
SELECT comments.id AS comment_id, photos.id AS photo_id
FROM comments
JOIN photos ON photos.id = comments.photo_id;
```
