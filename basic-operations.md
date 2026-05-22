# PostgreSQL Basic Operations Guide

## Overview

This guide covers the fundamental CRUD operations in PostgreSQL:
- **C**reate (CREATE TABLE, INSERT)
- **R**ead (SELECT with various filters and functions)
- **U**pdate (UPDATE)
- **D**elete (DELETE)

---

## 1. CREATE TABLE

### Syntax
```sql
CREATE TABLE table_name (
  column_name DATA_TYPE,
  column_name DATA_TYPE,
  ...
);
```

### Example: Cities Table
```sql
CREATE TABLE cities(
  name VARCHAR(50),
  country VARCHAR(50),
  population INT,
  area INT
);
```

**Explanation:**
- `name VARCHAR(50)` - Text column, max 50 characters
- `country VARCHAR(50)` - Text column, max 50 characters
- `population INT` - Integer column for population count
- `area INT` - Integer column for area in square units

---

## 2. INSERT - Adding Data

### Syntax
```sql
INSERT INTO table_name (column1, column2, column3)
VALUES 
  (value1, value2, value3),
  (value1, value2, value3),
  ...;
```

### Example: Insert Multiple Cities
```sql
INSERT INTO cities (name, country, population, area)
VALUES 
  ('Tokyo', 'Japan', 13960000, 2194000),
  ('New Delhi', 'India', 33000000, 1484000),
  ('Shanghai', 'China', 24870000, 6340000),
  ('Kathmandu', 'Nepal', 1500000, 49000);
```

**What happens:**
- Inserts 4 rows into the cities table
- Each row has name, country, population, and area values
- Multi-row insert is more efficient than individual INSERTs

### Single Row Insert
```sql
INSERT INTO cities (name, country, population, area)
VALUES ('Tokyo', 'Japan', 13960000, 2194000);
```

---

## 3. SELECT - Retrieving Data

### 3.1 Basic SELECT - All Columns

#### Syntax
```sql
SELECT * FROM table_name;
```

#### Example
```sql
SELECT 
  * 
FROM 
  cities;
```

**Output:**
```
       name       | country | population |  area
------------------+---------+------------+---------
 Tokyo            | Japan   |    13960000 | 2194000
 New Delhi        | India   |    33000000 | 1484000
 Shanghai         | China   |    24870000 | 6340000
 Kathmandu        | Nepal   |     1500000 |   49000
(4 rows)
```

---

### 3.2 SELECT Specific Columns

#### Syntax
```sql
SELECT column1, column2 FROM table_name;
```

#### Example
```sql
SELECT 
  name, 
  country 
FROM 
  cities;
```

**Output:**
```
       name       | country
------------------+---------
 Tokyo            | Japan
 New Delhi        | India
 Shanghai         | China
 Kathmandu        | Nepal
(4 rows)
```

---

### 3.3 SELECT with Calculations

#### Create New Columns Using Expressions
```sql
SELECT 
  name, 
  population / area AS population_density 
FROM 
  cities;
```

**Explanation:**
- `population / area` - Calculates population density
- `AS population_density` - Gives the calculated column a name (alias)
- This doesn't modify the table, just displays calculated values

**Output:**
```
       name       | population_density
------------------+--------------------
 Tokyo            |               6362
 New Delhi        |                 22
 Shanghai         |               3921
 Kathmandu        |                 30
(4 rows)
```

---

### 3.4 SELECT with String Functions

#### CONCAT - Combine Strings
```sql
SELECT 
  CONCAT(name, ' - ', country) AS formatted_name 
FROM 
  cities;
```

**Output:**
```
     formatted_name
------------------------
 Tokyo - Japan
 New Delhi - India
 Shanghai - China
 Kathmandu - Nepal
(4 rows)
```

#### CONCAT with UPPER - Convert to Uppercase
```sql
SELECT 
  UPPER(
    CONCAT(name, ' - ', country)
  ) AS formatted_name 
FROM 
  cities;
```

**Output:**
```
     formatted_name
------------------------
 TOKYO - JAPAN
 NEW DELHI - INDIA
 SHANGHAI - CHINA
 KATHMANDU - NEPAL
(4 rows)
```

**Other String Functions:**
```sql
-- Convert to lowercase
SELECT LOWER(name) FROM cities;

-- Get length of string
SELECT name, LENGTH(name) FROM cities;

-- Extract substring (starting position, length)
SELECT SUBSTRING(name, 1, 3) FROM cities;  -- First 3 characters
```

---

## 4. WHERE - Filtering Data

### 4.1 Greater Than (>)

```sql
SELECT 
  * 
FROM 
  cities 
WHERE 
  area > 49000;
```

**Output:**
```
       name       | country | population |  area
------------------+---------+------------+---------
 Tokyo            | Japan   |    13960000 | 2194000
 New Delhi        | India   |    33000000 | 1484000
 Shanghai         | China   |    24870000 | 6340000
(3 rows)
```

---

### 4.2 Not Equal - Two Methods

#### Method 1: Using !=
```sql
SELECT 
  * 
FROM 
  cities 
WHERE 
  area != 49000;
```

#### Method 2: Using <>
```sql
SELECT 
  * 
FROM 
  cities 
WHERE 
  area <> 49000;
```

**Both produce the same result:**
```
       name       | country | population |  area
------------------+---------+------------+---------
 Tokyo            | Japan   |    13960000 | 2194000
 New Delhi        | India   |    33000000 | 1484000
 Shanghai         | China   |    24870000 | 6340000
(3 rows)
```

---

### 4.3 BETWEEN - Range Filtering

#### Syntax
```sql
SELECT * FROM table_name WHERE column BETWEEN value1 AND value2;
```

#### Example
```sql
SELECT 
  * 
FROM 
  cities 
WHERE 
  area BETWEEN 1000000 AND 6500000;
```

**Output:**
```
       name       | country | population |  area
------------------+---------+------------+---------
 Tokyo            | Japan   |    13960000 | 2194000
 New Delhi        | India   |    33000000 | 1484000
 Shanghai         | China   |    24870000 | 6340000
(3 rows)
```

**Explanation:**
- Inclusive on both ends (>= 1000000 AND <= 6500000)
- Kathmandu (49000) is excluded

---

### 4.4 IN - Multiple Values

#### Select rows matching ANY value in a list
```sql
SELECT
  * 
FROM 
  cities
WHERE
  name IN ('Tokyo', 'Shanghai');
```

**Output:**
```
       name       | country | population |  area
------------------+---------+------------+---------
 Tokyo            | Japan   |    13960000 | 2194000
 Shanghai         | China   |    24870000 | 6340000
(2 rows)
```

---

### 4.5 NOT IN - Exclude Values

#### Select rows NOT matching any value in a list
```sql
SELECT
  *
FROM
  cities
WHERE
  name NOT IN ('Tokyo', 'Shanghai');
```

**Output:**
```
       name       | country | population |  area
------------------+---------+------------+---------
 New Delhi        | India   |    33000000 | 1484000
 Kathmandu        | Nepal   |     1500000 |   49000
(2 rows)
```

---

### 4.6 Multiple Conditions - AND

#### Use AND to require ALL conditions
```sql
SELECT
  *
FROM
  cities
WHERE
  name NOT IN ('Tokyo', 'Shanghai') 
  AND area > 5000 
  AND name = 'New Delhi';
```

**Output:**
```
       name    | country | population |  area
---------------+---------+------------+---------
 New Delhi      | India   |    33000000 | 1484000
(1 row)
```

**Explanation:**
- Row must satisfy ALL three conditions:
  1. Name is NOT Tokyo or Shanghai ✓
  2. Area is > 5000 ✓
  3. Name equals 'New Delhi' ✓

---

### 4.7 WHERE with Calculated Columns

#### Filter by calculated values
```sql
SELECT
  name, population / area AS population_density
FROM
  cities
WHERE
  population / area > 10;
```

**Output:**
```
       name       | population_density
------------------+--------------------
 Tokyo            |               6362
 Shanghai         |               3921
 New Delhi        |                 22
 Kathmandu        |                 30
(4 rows)
```

**Explanation:**
- Calculates population_density for each row
- Only shows rows where density > 10
- This filters AFTER calculation

---

## 5. UPDATE - Modifying Data

### Syntax
```sql
UPDATE table_name
SET 
  column1 = value1,
  column2 = value2
WHERE
  condition;
```

### Example: Update Tokyo's Data
```sql
UPDATE 
  cities
SET 
  population = 10, 
  area = 1
WHERE
  name = 'Tokyo';

SELECT * FROM cities;
```

**Output after UPDATE:**
```
       name       | country | population |  area
------------------+---------+------------+---------
 Tokyo            | Japan   |         10 |     1
 New Delhi        | India   |    33000000 | 1484000
 Shanghai         | China   |    24870000 | 6340000
 Kathmandu        | Nepal   |     1500000 |   49000
(4 rows)
```

**Important:** Always use WHERE clause to specify which rows to update. Without WHERE, ALL rows get updated!

### Common UPDATE Patterns

#### Update Multiple Columns
```sql
UPDATE cities
SET population = 10000000, area = 500000
WHERE name = 'Shanghai';
```

#### Update with Calculation
```sql
UPDATE cities
SET population = population * 2
WHERE country = 'India';
```

#### Update Multiple Rows
```sql
UPDATE cities
SET area = 999
WHERE area > 1000000;
```

---

## 6. DELETE - Removing Data

### Syntax
```sql
DELETE FROM table_name
WHERE condition;
```

### Example: Delete Tokyo
```sql
DELETE
FROM
  cities
WHERE
  name = 'Tokyo';

SELECT * FROM cities;
```

**Output after DELETE:**
```
       name       | country | population |  area
------------------+---------+------------+---------
 New Delhi        | India   |    33000000 | 1484000
 Shanghai         | China   |    24870000 | 6340000
 Kathmandu        | Nepal   |     1500000 |   49000
(3 rows)
```

**Warning:** Always use WHERE clause. Without WHERE, ALL rows get deleted!

### Common DELETE Patterns

#### Delete Multiple Rows
```sql
DELETE FROM cities
WHERE population < 2000000;
```

#### Delete All Rows (use with caution!)
```sql
DELETE FROM cities;  -- No WHERE clause - deletes everything!
```

#### Delete with Multiple Conditions
```sql
DELETE FROM cities
WHERE country = 'India' AND area < 500000;
```

---

## 7. Re-inserting Deleted Data

### Example: Add Tokyo Back
```sql
INSERT INTO 
  cities (name, country, population, area)
VALUES
  ('Tokyo', 'Japan', 13960000, 2194000);

SELECT * FROM cities;
```

**Output:**
```
       name       | country | population |  area
------------------+---------+------------+---------
 New Delhi        | India   |    33000000 | 1484000
 Shanghai         | China   |    24870000 | 6340000
 Kathmandu        | Nepal   |     1500000 |   49000
 Tokyo            | Japan   |    13960000 | 2194000
(4 rows)
```

**Note:** Tokyo is now at the end. Database doesn't guarantee order unless you use ORDER BY.

---

## 8. Common Comparison Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` | Equals | `WHERE name = 'Tokyo'` |
| `!=` or `<>` | Not equal | `WHERE population != 1000` |
| `>` | Greater than | `WHERE area > 1000000` |
| `<` | Less than | `WHERE population < 5000000` |
| `>=` | Greater or equal | `WHERE area >= 1000000` |
| `<=` | Less or equal | `WHERE population <= 10000000` |
| `BETWEEN` | Range (inclusive) | `WHERE area BETWEEN 100 AND 1000` |
| `IN` | Multiple values | `WHERE name IN ('Tokyo', 'Delhi')` |
| `NOT IN` | Exclude values | `WHERE name NOT IN ('Tokyo')` |
| `LIKE` | Pattern matching | `WHERE name LIKE 'To%'` |

---

## 9. Logical Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `AND` | ALL conditions must be true | `WHERE area > 1000 AND population > 1000000` |
| `OR` | ANY condition can be true | `WHERE country = 'Japan' OR country = 'India'` |
| `NOT` | Reverse a condition | `WHERE NOT (area > 1000000)` |

---

## 10. LIKE - Pattern Matching

### Syntax
```sql
SELECT * FROM table_name WHERE column LIKE 'pattern';
```

### Wildcards
- `%` - Matches any number of characters
- `_` - Matches exactly one character

### Examples

#### Match names starting with 'T'
```sql
SELECT * FROM cities WHERE name LIKE 'T%';
```

**Output:**
```
  name  | country | population |  area
--------+---------+------------+---------
 Tokyo  | Japan   |    13960000 | 2194000
(1 row)
```

#### Match names containing 'Del'
```sql
SELECT * FROM cities WHERE name LIKE '%Del%';
```

**Output:**
```
    name    | country | population |  area
------------+---------+------------+---------
 New Delhi  | India   |    33000000 | 1484000
(1 row)
```

#### Match names with 5 characters
```sql
SELECT * FROM cities WHERE name LIKE '_____';
```

**Output:**
```
 name  | country | population |  area
-------+---------+------------+---------
 Tokyo | Japan   |    13960000 | 2194000
(1 row)
```

---

## 11. DISTINCT - Remove Duplicates

### Syntax
```sql
SELECT DISTINCT column FROM table_name;
```

### Example: Get unique countries
```sql
SELECT DISTINCT country FROM cities;
```

**Output:**
```
 country
---------
 Japan
 India
 China
 Nepal
(4 rows)
```

**Without DISTINCT (shows all rows):**
```sql
SELECT country FROM cities;
```

Output would show each country even if duplicates exist.

### Multiple Columns
```sql
SELECT DISTINCT country, population FROM cities;
```

---

## 12. ORDER BY - Sort Results

### Syntax
```sql
SELECT * FROM table_name ORDER BY column ASC|DESC;
```

### Sort Ascending (default, A-Z or smallest to largest)
```sql
SELECT * FROM cities ORDER BY population;
```

**Output:**
```
       name       | country | population |  area
------------------+---------+------------+---------
 Kathmandu        | Nepal   |     1500000 |   49000
 Tokyo            | Japan   |    13960000 | 2194000
 Shanghai         | China   |    24870000 | 6340000
 New Delhi        | India   |    33000000 | 1484000
(4 rows)
```

### Sort Descending (Z-A or largest to smallest)
```sql
SELECT * FROM cities ORDER BY population DESC;
```

**Output:**
```
       name       | country | population |  area
------------------+---------+------------+---------
 New Delhi        | India   |    33000000 | 1484000
 Shanghai         | China   |    24870000 | 6340000
 Tokyo            | Japan   |    13960000 | 2194000
 Kathmandu        | Nepal   |     1500000 |   49000
(4 rows)
```

### Sort by Multiple Columns
```sql
SELECT * FROM cities ORDER BY country ASC, population DESC;
```

**Explanation:**
- First sort by country (A-Z)
- Within same country, sort by population (largest to smallest)

### Sort by Calculated Column
```sql
SELECT name, population / area AS density 
FROM cities 
ORDER BY population / area DESC;
```

---

## 13. LIMIT - Restrict Number of Rows

### Syntax
```sql
SELECT * FROM table_name LIMIT number;
```

### Get Top 2 Cities by Population
```sql
SELECT * FROM cities ORDER BY population DESC LIMIT 2;
```

**Output:**
```
       name       | country | population |  area
------------------+---------+------------+---------
 New Delhi        | India   |    33000000 | 1484000
 Shanghai         | China   |    24870000 | 6340000
(2 rows)
```

### OFFSET - Skip Rows
```sql
SELECT * FROM cities ORDER BY population DESC LIMIT 2 OFFSET 1;
```

**Explanation:**
- `LIMIT 2` - Get 2 rows
- `OFFSET 1` - Skip the first 1 row
- Returns rows 2 and 3

**Output:**
```
       name       | country | population |  area
------------------+---------+------------+---------
 Shanghai         | China   |    24870000 | 6340000
 Tokyo            | Japan   |    13960000 | 2194000
(2 rows)
```

### Pagination Example
```sql
-- Page 1 (rows 1-2)
SELECT * FROM cities ORDER BY name LIMIT 2 OFFSET 0;

-- Page 2 (rows 3-4)
SELECT * FROM cities ORDER BY name LIMIT 2 OFFSET 2;

-- Page 3 (rows 5-6)
SELECT * FROM cities ORDER BY name LIMIT 2 OFFSET 4;
```

---

## 14. Aggregate Functions - Data Summary

Aggregate functions work on multiple rows and return a single result.

### 14.1 COUNT - Count Rows

#### Syntax
```sql
SELECT COUNT(*) FROM table_name;
```

#### Count all cities
```sql
SELECT COUNT(*) FROM cities;
```

**Output:**
```
 count
-------
     4
(1 row)
```

#### Count cities with population > 10 million
```sql
SELECT COUNT(*) FROM cities WHERE population > 10000000;
```

**Output:**
```
 count
-------
     3
(1 row)
```

#### Count with alias
```sql
SELECT COUNT(*) AS total_cities FROM cities;
```

---

### 14.2 SUM - Add Values

#### Total population of all cities
```sql
SELECT SUM(population) AS total_population FROM cities;
```

**Output:**
```
 total_population
------------------
         72330000
(1 row)
```

#### Total area of cities with > 1M population
```sql
SELECT SUM(area) AS total_area 
FROM cities 
WHERE population > 1000000;
```

---

### 14.3 AVG - Average Value

#### Average population across cities
```sql
SELECT AVG(population) AS avg_population FROM cities;
```

**Output:**
```
 avg_population
----------------
   18082500.000
(1 row)
```

#### Average area
```sql
SELECT AVG(area) AS avg_area FROM cities;
```

---

### 14.4 MIN - Minimum Value

#### Smallest population
```sql
SELECT MIN(population) AS smallest_population FROM cities;
```

**Output:**
```
 smallest_population
---------------------
             1500000
(1 row)
```

#### City with smallest area
```sql
SELECT name, MIN(area) FROM cities;
```

---

### 14.5 MAX - Maximum Value

#### Largest population
```sql
SELECT MAX(population) AS largest_population FROM cities;
```

**Output:**
```
 largest_population
--------------------
           33000000
(1 row)
```

#### City with largest area
```sql
SELECT name, MAX(area) FROM cities;
```

---

### 14.6 Multiple Aggregate Functions

```sql
SELECT 
  COUNT(*) AS total_cities,
  SUM(population) AS total_population,
  AVG(population) AS avg_population,
  MIN(population) AS min_population,
  MAX(population) AS max_population
FROM cities;
```

**Output:**
```
 total_cities | total_population | avg_population  | min_population | max_population
--------------+------------------+-----------------+----------------+----------------
            4 |         72330000 |   18082500.0000 |         1500000 |       33000000
(1 row)
```

---

## 15. GROUP BY - Aggregate by Category

### Syntax
```sql
SELECT column, aggregate_function(column) 
FROM table_name 
GROUP BY column;
```

### Example: Count cities per country

First, let's add more cities for better example:
```sql
INSERT INTO cities (name, country, population, area)
VALUES 
  ('Osaka', 'Japan', 19000000, 223000),
  ('Mumbai', 'India', 20411000, 604000);
```

Now group by country:
```sql
SELECT 
  country, 
  COUNT(*) AS city_count,
  AVG(population) AS avg_population
FROM cities
GROUP BY country;
```

**Output:**
```
 country | city_count |  avg_population
---------+------------+------------------
 Japan   |          2 |    16480000.0000
 India   |          2 |    26705500.0000
 China   |          1 |    24870000.0000
 Nepal   |          1 |     1500000.0000
(4 rows)
```

**Explanation:**
- Groups rows by country
- Calculates COUNT and AVG for each group
- Returns one row per country

### GROUP BY Multiple Columns

```sql
SELECT 
  country,
  COUNT(*) AS total_cities,
  SUM(population) AS total_population,
  AVG(area) AS avg_area
FROM cities
GROUP BY country
ORDER BY total_population DESC;
```

---

## 16. HAVING - Filter After GROUP BY

### Syntax
```sql
SELECT column, aggregate_function(column)
FROM table_name
GROUP BY column
HAVING aggregate_function(column) condition;
```

**Difference from WHERE:**
- `WHERE` - Filters BEFORE grouping
- `HAVING` - Filters AFTER grouping

### Example: Countries with > 1 city

```sql
SELECT 
  country, 
  COUNT(*) AS city_count
FROM cities
GROUP BY country
HAVING COUNT(*) > 1;
```

**Output:**
```
 country | city_count
---------+------------
 Japan   |          2
 India   |          2
(2 rows)
```

### Countries with average population > 5 million

```sql
SELECT 
  country, 
  AVG(population) AS avg_population,
  COUNT(*) AS city_count
FROM cities
GROUP BY country
HAVING AVG(population) > 5000000
ORDER BY avg_population DESC;
```

**Output:**
```
 country |  avg_population  | city_count
---------+------------------+------------
 India   |  26705500.000000 |          2
 Japan   |  16480000.000000 |          2
 China   |  24870000.000000 |          1
(3 rows)
```

---

## 17. WHERE vs HAVING - Complete Example

### WHERE - Filter individual rows first
```sql
SELECT 
  country, 
  COUNT(*) AS city_count,
  AVG(population) AS avg_pop
FROM cities
WHERE population > 10000000  -- Filter rows BEFORE grouping
GROUP BY country;
```

### HAVING - Filter groups after aggregation
```sql
SELECT 
  country, 
  COUNT(*) AS city_count,
  AVG(population) AS avg_pop
FROM cities
GROUP BY country
HAVING COUNT(*) > 1;  -- Filter groups AFTER aggregation
```

### Both - Filter rows, then groups
```sql
SELECT 
  country, 
  COUNT(*) AS city_count,
  AVG(population) AS avg_pop
FROM cities
WHERE population > 10000000  -- Filter rows first
GROUP BY country
HAVING COUNT(*) > 1;  -- Filter groups second
```

---

## 18. NULL Values - Handling Missing Data

### NULL in WHERE Clause

#### Check for NULL
```sql
SELECT * FROM cities WHERE population IS NULL;
```

#### Check for NOT NULL
```sql
SELECT * FROM cities WHERE population IS NOT NULL;
```

#### COALESCE - Replace NULL with default
```sql
SELECT 
  name, 
  COALESCE(population, 0) AS population
FROM cities;
```

**Explanation:**
- If population is NULL, use 0
- Otherwise use actual population value

---

## 19. AS - Column Aliases

### Rename columns in output
```sql
SELECT 
  name AS city_name,
  country AS nation,
  population AS total_residents
FROM cities;
```

### Alias with expressions
```sql
SELECT 
  name AS city_name,
  population / area AS people_per_sq_unit
FROM cities;
```

---

## 20. Quick Reference - All Operations Together

```sql
-- 1. CREATE TABLE
CREATE TABLE cities(
  name VARCHAR(50),
  country VARCHAR(50),
  population INT,
  area INT
);

-- 2. INSERT data
INSERT INTO cities (name, country, population, area)
VALUES ('Tokyo', 'Japan', 13960000, 2194000);

-- 3. SELECT all
SELECT * FROM cities;

-- 4. SELECT with filter
SELECT name, country FROM cities WHERE population > 10000000;

-- 5. SELECT with calculation
SELECT name, population / area AS density FROM cities;

-- 6. SELECT with LIKE pattern
SELECT * FROM cities WHERE name LIKE 'T%';

-- 7. SELECT DISTINCT
SELECT DISTINCT country FROM cities;

-- 8. SELECT with ORDER BY
SELECT * FROM cities ORDER BY population DESC;

-- 9. SELECT with LIMIT
SELECT * FROM cities LIMIT 5;

-- 10. SELECT with LIMIT and OFFSET
SELECT * FROM cities LIMIT 5 OFFSET 10;

-- 11. Aggregate functions
SELECT 
  COUNT(*) AS total,
  AVG(population) AS avg_pop,
  MAX(population) AS max_pop,
  MIN(population) AS min_pop,
  SUM(area) AS total_area
FROM cities;

-- 12. GROUP BY
SELECT country, COUNT(*) AS city_count
FROM cities
GROUP BY country;

-- 13. GROUP BY with HAVING
SELECT country, COUNT(*) AS city_count
FROM cities
GROUP BY country
HAVING COUNT(*) > 1;

-- 14. UPDATE data
UPDATE cities SET population = 14000000 WHERE name = 'Tokyo';

-- 15. DELETE data
DELETE FROM cities WHERE name = 'Tokyo';

-- 16. Verify deletion
SELECT * FROM cities;
```

---

## Practice Exercises

Try these on your own cities table:

1. **Find all cities with population > 20 million**
   ```sql
   SELECT * FROM cities WHERE population > 20000000;
   ```

2. **Update Kathmandu's population to 2000000**
   ```sql
   UPDATE cities SET population = 2000000 WHERE name = 'Kathmandu';
   ```

3. **Find cities with area between 1 million and 5 million**
   ```sql
   SELECT * FROM cities WHERE area BETWEEN 1000000 AND 5000000;
   ```

4. **Delete all cities with area < 100000**
   ```sql
   DELETE FROM cities WHERE area < 100000;
   ```

5. **Show all cities sorted by population (descending)**
   ```sql
   SELECT * FROM cities ORDER BY population DESC;
   ```

---

## Key Takeaways

✅ **CREATE TABLE** - Define table structure  
✅ **INSERT** - Add new data  
✅ **SELECT** - Retrieve data with filters and calculations  
✅ **WHERE** - Filter rows based on conditions  
✅ **LIKE** - Pattern matching with % and _ wildcards  
✅ **DISTINCT** - Remove duplicate values  
✅ **ORDER BY** - Sort results ascending (ASC) or descending (DESC)  
✅ **LIMIT** - Restrict number of rows returned  
✅ **OFFSET** - Skip rows (pagination)  
✅ **COUNT, SUM, AVG, MIN, MAX** - Aggregate functions for summarization  
✅ **GROUP BY** - Group rows and aggregate by category  
✅ **HAVING** - Filter groups after aggregation (vs WHERE before)  
✅ **NULL handling** - IS NULL, IS NOT NULL, COALESCE  
✅ **AS** - Create aliases for columns  
✅ **UPDATE** - Modify existing data (always use WHERE!)  
✅ **DELETE** - Remove data (always use WHERE!)  
✅ **Multiple conditions** - Use AND, OR for complex filtering  

These basic operations form the foundation of all database work. Master them, and you're ready for more complex queries!
