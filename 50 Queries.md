# 50+ Queries

Here is your SQL tutorial converted into a Notion-ready format with clean headings, toggle-friendly query sections, SQL code blocks, explanations, and practice tasks. You can copy and paste it directly into a Notion page.

# SQL Tutorial Notes: 50 Practical Queries

Database: `awesome_chocolates` · SQL dialect: MySQL · Level: Beginner to Intermediate

## 1. Overview and Database Setup

This tutorial covers 50 practical SQL queries using MySQL Workbench and the `awesome_chocolates` database.

Topics covered

- Table inspection and data retrieval
- Calculated columns and aliases
- Filtering and sorting
- Date functions and pattern matching
- `CASE` expressions
- Joins
- Grouping and aggregate functions
- `HAVING` and `LIMIT`

### Database tables

| Table | Purpose |
| --- | --- |
| `sales` | Sales dates, amounts, boxes, and related IDs |
| `people` | Salesperson names, teams, and IDs |
| `products` | Product names, categories, and IDs |
| `geo` | Geographic information |

Setup: Select the `awesome_chocolates` schema in MySQL Workbench before running the queries. Use semicolons to separate statements.

## 2. Inspecting Tables and Retrieving Data

### Query 1 — List all tables

```
SHOW TABLES;
```

Lists the tables in the selected database.

### Query 2 — Inspect the sales table

```
DESCRIBE sales;
```

Displays column names, data types, and schema details.

### Query 3 — Retrieve all columns

```
SELECT *
FROM sales;
```

The asterisk (`*`) selects every column.

### Query 4 — Retrieve selected columns

```
SELECT sale_date,
       amount,
       customers
FROM sales;
```

Returns only the specified columns in the requested order.

### Query 5 — Change column order

```
SELECT amount,
       customers,
       geo_id
FROM sales;
```

The output column order follows the `SELECT` list.

## 3. Calculations and Aliases

### Query 6 — Calculate amount per box

```
SELECT sale_date,
       amount,
       boxes,
       amount / boxes
FROM sales;
```

Divides `amount` by `boxes` for each row.

> Note: Handle rows where `boxes` is zero carefully.
> 

### Query 7 — Name a calculated column

```
SELECT sale_date,
       amount,
       boxes,
       amount / boxes AS amount_per_box
FROM sales;
```

`AS` assigns a readable alias to the calculated column.

### Query 8 — Alias without `AS`

```
SELECT amount / boxes amount_per_box
FROM sales;
```

MySQL allows column aliases without the `AS` keyword.

## 4. Filtering and Sorting

### Query 9 — Find amounts greater than 10,000

```
SELECT *
FROM sales
WHERE amount > 10000;
```

`WHERE` filters rows based on a condition.

### Query 10 — Sort amounts in ascending order

```
SELECT *
FROM sales
WHERE amount > 10000
ORDER BY amount ASC;
```

`ASC` sorts from lowest to highest.

### Query 11 — Sort amounts in descending order

```
SELECT *
FROM sales
WHERE amount > 10000
ORDER BY amount DESC;
```

`DESC` sorts from highest to lowest.

### Query 12 — Filter by geography and sort by two columns

```
SELECT *
FROM sales
WHERE geo_id = 'G1'
ORDER BY pid ASC, amount DESC;
```

Filters geography `G1`, sorts by product ID, and then sorts amounts in descending order within each product.

### Query 13 — Combine amount and date conditions

```
SELECT *
FROM sales
WHERE amount > 10000
  AND sale_date >= '2022-01-01';
```

`AND` requires both conditions to be true.

### Query 14 — Find sales from 2022

```
SELECT sale_date,
       amount
FROM sales
WHERE amount > 10000
  AND YEAR(sale_date) = 2022
ORDER BY amount DESC;
```

`YEAR()` extracts the year from a date.

### Query 15 — Filter a numeric range using comparisons

```
SELECT *
FROM sales
WHERE boxes > 0
  AND boxes <= 50;
```

Returns rows where the box count is greater than zero and no more than 50.

### Query 16 — Filter using `BETWEEN`

```
SELECT *
FROM sales
WHERE boxes BETWEEN 0 AND 50;
```

`BETWEEN` includes both endpoints.

### Query 17 — Find Friday sales

```
SELECT sale_date,
       amount,
       boxes,
       WEEKDAY(sale_date) AS day_of_week
FROM sales
WHERE WEEKDAY(sale_date) = 4;
```

In MySQL, `WEEKDAY()` returns `0` for Monday through `6` for Sunday. Therefore, `4` represents Friday.

### Query 18 — Match either of two teams

```
SELECT *
FROM people
WHERE team = 'Delish'
   OR team = 'Jucies';
```

`OR` returns rows matching either condition.

### Query 19 — Use `IN` for multiple values

```
SELECT *
FROM people
WHERE team IN ('Delish', 'Jucies');
```

`IN` is a concise alternative to multiple equality conditions joined by `OR`.

### Query 20 — Find names beginning with B

```
SELECT *
FROM people
WHERE sales_person LIKE 'B%';
```

`%` matches zero or more characters.

### Query 21 — Find names containing B

```
SELECT *
FROM people
WHERE sales_person LIKE '%B%';
```

Finds names containing `B` anywhere in the text, subject to the database's collation rules.

### Query 22 — Exclude a team

```
SELECT *
FROM people
WHERE team <> 'Delish';
```

`<>` means not equal to. `NULL` values are not included by this comparison.

### Query 23 — Find `NULL` team values

```
SELECT *
FROM people
WHERE team IS NULL;
```

`IS NULL` finds missing SQL values.

### Query 24 — Find empty-string team values

```
SELECT *
FROM people
WHERE team = '';
```

An empty string differs from `NULL`. To check for values containing only spaces, use:

```
SELECT *
FROM people
WHERE TRIM(team) = '';
```

## 5. Categorizing Data with CASE

### Query 25 — Create amount categories

```
SELECT sale_date,
       amount,
       CASE
           WHEN amount < 1000 THEN 'Under 1K'
           WHEN amount < 5000 THEN 'Under 5K'
           WHEN amount < 10000 THEN 'Under 10K'
           ELSE '10K or more'
       END AS amount_category
FROM sales;
```

`CASE` evaluates conditions in order and returns the label for the first matching condition.

### Query 26 — Label shipment sizes

```
SELECT boxes,
       CASE
           WHEN boxes <= 10 THEN 'Small shipment'
           WHEN boxes <= 50 THEN 'Medium shipment'
           ELSE 'Large shipment'
       END AS shipment_size
FROM sales;
```

This classifies shipment sizes using numeric conditions.

## 6. Joining Tables

The examples use these relationships:

- `sales.spid` → `people.spid`
- `sales.pid` → `products.pid`
- `sales.geo_id` → `geo.geo_id`

### Query 27 — Join sales to people

```
SELECT s.sale_date,
       s.amount,
       s.spid,
       p.sales_person
FROM sales AS s
JOIN people AS p
  ON p.spid = s.spid;
```

`JOIN`, or `INNER JOIN`, returns rows with matching salesperson IDs in both tables.

### Query 28 — Preserve all sales with `LEFT JOIN`

```
SELECT s.sale_date,
       s.amount,
       p.sales_person
FROM sales AS s
LEFT JOIN people AS p
  ON p.spid = s.spid;
```

Keeps every row from `sales`. Unmatched people columns return `NULL`.

### Query 29 — Join sales to products

```
SELECT s.sale_date,
       s.amount,
       s.pid,
       pr.product
FROM sales AS s
LEFT JOIN products AS pr
  ON pr.pid = s.pid;
```

Adds product names while retaining unmatched sales rows.

### Query 30 — Join sales, people, and products

```
SELECT s.sale_date,
       s.amount,
       p.sales_person,
       p.team,
       pr.product
FROM sales AS s
LEFT JOIN people AS p
  ON p.spid = s.spid
LEFT JOIN products AS pr
  ON pr.pid = s.pid;
```

Combines information from three related tables.

### Query 31 — Filter joined data by amount

```
SELECT s.sale_date,
       s.amount,
       p.sales_person,
       p.team,
       pr.product
FROM sales AS s
LEFT JOIN people AS p
  ON p.spid = s.spid
LEFT JOIN products AS pr
  ON pr.pid = s.pid
WHERE s.amount < 500;
```

Returns sales records with amounts below 500.

### Query 32 — Filter by amount and team

```
SELECT s.sale_date,
       s.amount,
       p.sales_person,
       p.team,
       pr.product
FROM sales AS s
LEFT JOIN people AS p
  ON p.spid = s.spid
LEFT JOIN products AS pr
  ON pr.pid = s.pid
WHERE s.amount < 500
  AND p.team = 'Delish';
```

Both conditions must be true.

### Query 33 — Find low-value sales with a `NULL` team

```
SELECT s.sale_date,
       s.amount,
       p.sales_person,
       p.team,
       pr.product
FROM sales AS s
LEFT JOIN people AS p
  ON p.spid = s.spid
LEFT JOIN products AS pr
  ON pr.pid = s.pid
WHERE s.amount < 500
  AND p.team IS NULL;
```

Finds sales below 500 where the joined team value is `NULL`. This can also include sales with no matching salesperson record.

### Query 34 — Filter sales by geography

```
SELECT s.sale_date,
       s.amount,
       p.sales_person,
       pr.product,
       g.geo
FROM sales AS s
LEFT JOIN people AS p
  ON p.spid = s.spid
LEFT JOIN products AS pr
  ON pr.pid = s.pid
JOIN geo AS g
  ON g.geo_id = s.geo_id
WHERE s.amount < 500
  AND g.geo IN ('New Zealand', 'India')
ORDER BY s.sale_date;
```

Returns sales below 500 for New Zealand or India, sorted by date.

## 7. Grouping and Aggregate Functions

Aggregate functions summarize data across rows.

| Function | Purpose |
| --- | --- |
| `SUM()` | Calculates a total |
| `AVG()` | Calculates an average |
| `COUNT()` | Counts rows or non-NULL values |
| `MIN()` | Finds the minimum value |
| `MAX()` | Finds the maximum value |

### Query 35 — Total amount by geography ID

```
SELECT geo_id,
       SUM(amount) AS total_amount
FROM sales
GROUP BY geo_id;
```

Calculates total sales amount for each geography.

### Query 36 — Average amount by geography ID

```
SELECT geo_id,
       AVG(amount) AS average_amount
FROM sales
GROUP BY geo_id;
```

Calculates average sales amount for each geography.

### Query 37 — Total boxes by geography ID

```
SELECT geo_id,
       SUM(boxes) AS total_boxes
FROM sales
GROUP BY geo_id;
```

Calculates total boxes for each geography.

### Query 38 — Summarize sales by geography name

```
SELECT g.geo,
       SUM(s.amount) AS total_amount,
       AVG(s.amount) AS average_amount,
       SUM(s.boxes) AS total_boxes
FROM sales AS s
JOIN geo AS g
  ON g.geo_id = s.geo_id
GROUP BY g.geo;
```

Displays geography names alongside summary metrics.

### Query 39 — Summarize by product category and team

```
SELECT pr.category,
       p.team,
       SUM(s.boxes) AS total_boxes,
       SUM(s.amount) AS total_amount
FROM sales AS s
JOIN people AS p
  ON p.spid = s.spid
JOIN products AS pr
  ON pr.pid = s.pid
GROUP BY pr.category, p.team;
```

Creates a summary for each category-and-team combination.

### Query 40 — Sort a grouped report

```
SELECT pr.category,
       p.team,
       SUM(s.boxes) AS total_boxes,
       SUM(s.amount) AS total_amount
FROM sales AS s
JOIN people AS p
  ON p.spid = s.spid
JOIN products AS pr
  ON pr.pid = s.pid
GROUP BY pr.category, p.team
ORDER BY pr.category, p.team;
```

Sorts grouped results by category and then team.

### Query 41 — Exclude empty team values

```
SELECT pr.category,
       p.team,
       SUM(s.boxes) AS total_boxes,
       SUM(s.amount) AS total_amount
FROM sales AS s
JOIN people AS p
  ON p.spid = s.spid
JOIN products AS pr
  ON pr.pid = s.pid
WHERE p.team <> ''
GROUP BY pr.category, p.team
ORDER BY pr.category, p.team;
```

Filters empty strings before grouping.

Note: This condition does not exclude `NULL`. Add `AND p.team IS NOT NULL` if needed.

### Query 42 — Count sales rows by geography

```
SELECT geo_id,
       COUNT(*) AS sales_count
FROM sales
GROUP BY geo_id;
```

Counts sales records in each geography.

### Query 43 — Find minimum and maximum amount by geography

```
SELECT geo_id,
       MIN(amount) AS minimum_amount,
       MAX(amount) AS maximum_amount
FROM sales
GROUP BY geo_id;
```

Returns the smallest and largest sales amounts in each geography.

### Query 44 — Filter groups with `HAVING`

```
SELECT geo_id,
       SUM(amount) AS total_amount
FROM sales
GROUP BY geo_id
HAVING SUM(amount) > 100000;
```

`HAVING` filters groups after aggregation. In contrast, `WHERE` filters individual rows before grouping.

## 8. Top Results with LIMIT

### Query 45 — Calculate total sales by product

```
SELECT pr.product,
       SUM(s.amount) AS total_amount
FROM sales AS s
JOIN products AS pr
  ON pr.pid = s.pid
GROUP BY pr.product;
```

Calculates the total sales amount for each product.

### Query 46 — Sort products by total sales

```
SELECT pr.product,
       SUM(s.amount) AS total_amount
FROM sales AS s
JOIN products AS pr
  ON pr.pid = s.pid
GROUP BY pr.product
ORDER BY total_amount DESC;
```

Ranks products from highest to lowest total sales.

### Query 47 — Return the top 10 products by sales

```
SELECT pr.product,
       SUM(s.amount) AS total_amount
FROM sales AS s
JOIN products AS pr
  ON pr.pid = s.pid
GROUP BY pr.product
ORDER BY total_amount DESC
LIMIT 10;
```

Returns the first ten products after sorting.

### Query 48 — Find five products with the lowest sales

```
SELECT pr.product,
       SUM(s.amount) AS total_amount
FROM sales AS s
JOIN products AS pr
  ON pr.pid = s.pid
GROUP BY pr.product
ORDER BY total_amount ASC
LIMIT 5;
```

Returns the five products with the lowest total sales amounts.

### Query 49 — Top 10 products by boxes sold

```
SELECT pr.product,
       SUM(s.boxes) AS total_boxes
FROM sales AS s
JOIN products AS pr
  ON pr.pid = s.pid
GROUP BY pr.product
ORDER BY total_boxes DESC
LIMIT 10;
```

Ranks products by total boxes sold rather than sales amount.

### Query 50 — Build a team-level sales report

```
SELECT p.team,
       SUM(s.amount) AS total_amount,
       SUM(s.boxes) AS total_boxes,
       AVG(s.amount) AS average_sale
FROM sales AS s
JOIN people AS p
  ON p.spid = s.spid
GROUP BY p.team
ORDER BY total_amount DESC;
```

Combines joins, aggregate functions, grouping, aliases, and sorting into a team-level report.

## 9. Important SQL Concepts — Quick Revision

| Concept | Remember |
| --- | --- |
| `SELECT` | Chooses columns |
| `FROM` | Specifies the source table |
| `WHERE` | Filters individual rows |
| `AND` / `OR` | Combines conditions |
| `ORDER BY` | Sorts query results |
| `BETWEEN` | Includes both range endpoints |
| `IN` | Matches a list of values |
| `LIKE` | Matches text patterns |
| `CASE` | Creates conditional categories |
| `JOIN` | Combines matching rows |
| `LEFT JOIN` | Preserves every row from the left table |
| `GROUP BY` | Creates groups for summaries |
| `HAVING` | Filters grouped results |
| `LIMIT` | Restricts the number of returned rows |
| `IS NULL` | Checks for missing SQL values |

### Simplified SQL execution order

1. `FROM` / `JOIN`
2. `WHERE`
3. `GROUP BY`
4. `HAVING`
5. `SELECT`
6. `ORDER BY`
7. `LIMIT`

## 10. Practice Tasks

Try solving these independently before checking your notes.

- [ ]  Display `sale_date`, `amount`, and `boxes` for sales above 5,000.
- [ ]  Find sales in 2022 with more than 25 boxes.
- [ ]  List salespeople whose names begin with A.
- [ ]  Calculate total sales by salesperson.
- [ ]  Calculate average amount by product category.
- [ ]  Use `HAVING` to find categories whose total sales exceed a chosen threshold.
- [ ]  Show the top five geographies by total sales amount.
- [ ]  Compare total boxes by team and category.
- [ ]  Use `LEFT JOIN` to find sales with no matching salesperson or product.
- [ ]  Create a `CASE` expression that labels sales as low, medium, or high.

## Final Notes

- These examples target MySQL.
- Verify actual table and column names before executing queries.
- Remember that `NULL`, empty strings, and whitespace-only values are different.
- Use `ORDER BY` with `LIMIT` when you need meaningful top or bottom results.
- Practice by changing filters, sorting directions, grouping columns, and aggregate functions.

Notion tip: To make each query expandable, paste a query into a Notion Toggle Heading 3 block and place its SQL code and explanation inside. This keeps all 50 queries organized while making the page easier to navigate.
