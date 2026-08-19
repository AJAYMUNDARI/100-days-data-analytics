![Medium](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
# Count Rows Produced by Different JOIN Types

---

## Business Context

This type of analysis is useful when working with **large-scale advertising and marketing datasets**, such as Allstate's online advertising platform.

Understanding how different join types affect the resulting row count is important for:

* validating SQL logic,
* estimating query output size,
* preventing accidental data duplication,
* understanding data relationships,
* optimizing analytical queries,
* debugging complex reporting pipelines.

This question is particularly useful for demonstrating a strong understanding of **SQL JOIN behavior**.

---

## Problem Statement

The `ads` table contains advertisements ranked by popularity using the `id` column, where:

```text
id = 1 → most popular
id = 2 → second most popular
id = 3 → third most popular
...
```

We need to:

1. Create a subquery/CTE named `top_ads`.
2. Store the top 3 most popular ads in `top_ads`.
3. Perform four different joins between `ads` and `top_ads`.
4. Count the number of rows produced by each join.
5. Return all four results in a single query.

Required join labels:

* `inner_join`
* `left_join`
* `right_join`
* `cross_join`

---

## Input Tables

### ads

| Column | Type    |
| ------ | ------- |
| id     | INTEGER |
| name   | VARCHAR |

---

## Expected Output

| Column         | Type    |
| -------------- | ------- |
| join_type      | VARCHAR |
| number_of_rows | INTEGER |

---

## 1. Understand the Requirement

We need to compare how different JOIN types affect the number of rows returned.

First, create:

```sql
top_ads
```

containing the three most popular ads.

Then calculate the row count for:

```text
ads INNER JOIN top_ads
ads LEFT JOIN top_ads
ads RIGHT JOIN top_ads
ads CROSS JOIN top_ads
```

---

## 2. Understand the Tables

| Table   | Purpose                           | Key Columns |
| ------- | --------------------------------- | ----------- |
| ads     | Contains all advertisements       | id, name    |
| top_ads | Contains the top 3 advertisements | id          |

`top_ads` is derived from `ads`.

---

## 3. Clarify the Grain

* `ads`: one row per advertisement.
* `top_ads`: one row per top advertisement.
* Each JOIN produces a different number of rows.
* Final output: one row per JOIN type.

---

## 4. Identify Relationships

For `INNER`, `LEFT`, and `RIGHT JOIN`:

```sql
top_ads.id = ads.id
```

The `CROSS JOIN` has **no join condition** because it produces the Cartesian product.

---

## 5. Determine the Driving Table

For the first three joins:

```text
ads → top_ads
```

For the `CROSS JOIN`, both tables participate equally in the Cartesian product.

---

## 6. Think About Join Type

We need to explicitly demonstrate four JOIN types:

### INNER JOIN

Returns only matching rows.

### LEFT JOIN

Returns every row from `ads` and matching rows from `top_ads`.

### RIGHT JOIN

Returns every row from `top_ads` and matching rows from `ads`.

### CROSS JOIN

Returns every possible combination between `ads` and `top_ads`.

---

## 7. Find the "No Match" Condition

For the `LEFT JOIN`, advertisements outside the top 3 have no matching row in `top_ads`, but they are still retained.

For the `RIGHT JOIN`, every row from `top_ads` is retained.

The `INNER JOIN` removes non-matching advertisements.

The `CROSS JOIN` does not have a match/no-match condition because it creates every possible pair.

---

## 8. Interpret Special Conditions / Notes

The top 3 ads are determined using:

```sql
ORDER BY id
LIMIT 3
```

Because lower IDs represent higher popularity.

For example:

```text
ads
 ↓
ORDER BY id
 ↓
LIMIT 3
 ↓
top_ads
```

If `ads` contains 10 rows:

```text
top_ads = IDs 1, 2, 3
```

---

## 9. Final Calculation Logic

### Step 1 — Create `top_ads`

```sql
WITH top_ads AS (
    SELECT id
    FROM ads
    ORDER BY id
    LIMIT 3
)
```

### Step 2 — Count each JOIN

Calculate:

```text
INNER JOIN → matching rows
LEFT JOIN  → all ads
RIGHT JOIN → top 3 ads
CROSS JOIN → ads × 3
```

### Step 3 — Combine results

Use `UNION ALL` to return all four counts as separate rows.

---

## 10. SQL Solution

```sql
WITH top_ads AS (SELECT id
                 FROM ads
                 ORDER BY id
                 LIMIT 3
                )

SELECT
    'inner_join' AS join_type,
     COUNT(*) AS number_of_rows
FROM ads AS a
INNER JOIN top_ads AS ta
    ON ta.id = a.id

UNION ALL

SELECT
    'left_join' AS join_type,
    COUNT(*) AS number_of_rows
FROM ads AS a
LEFT JOIN top_ads AS ta
    ON ta.id = a.id

UNION ALL

SELECT
    'right_join' AS join_type,
    COUNT(*) AS number_of_rows
FROM ads AS a
RIGHT JOIN top_ads AS ta
    ON ta.id = a.id

UNION ALL

SELECT
    'cross_join' AS join_type,
    COUNT(*) AS number_of_rows
FROM ads AS a
CROSS JOIN top_ads AS ta;
```

---

## 11. Understand the JOIN Results

Assume there are **10 ads**:

```text
ads = 10 rows
top_ads = 3 rows
```

### INNER JOIN

Only the top 3 ads match:

```text
10 ads
   ↓
3 matching ads
```

Result:

```text
3 rows
```

### LEFT JOIN

Every row from `ads` is retained:

```text
10 ads
   ↓
10 rows
```

Result:

```text
10 rows
```

### RIGHT JOIN

Every row from `top_ads` is retained.

Since every `top_ads` row exists in `ads`:

```text
3 top ads
   ↓
3 matching rows
```

Result:

```text
3 rows
```

### CROSS JOIN

Every `ads` row is combined with every `top_ads` row:

```text
10 × 3 = 30
```

Result:

```text
30 rows
```

---

## 12. JOIN Behavior Summary

| JOIN Type  | Logic                   | If `ads` has N rows |
| ---------- | ----------------------- | ------------------: |
| INNER JOIN | Only top 3 matching ads |                   3 |
| LEFT JOIN  | All ads retained        |                   N |
| RIGHT JOIN | All top ads retained    |                   3 |
| CROSS JOIN | Every combination       |               N × 3 |

---

## 13. Why `UNION ALL`?

We want to return **four separate rows**, one for each JOIN type.

`UNION ALL` combines the four result sets without removing duplicates.

For example:

```text
inner_join → 3
left_join  → 10
right_join → 3
cross_join → 30
```

---

## 14. Key SQL Concepts Used

* CTE
* `WITH`
* `INNER JOIN`
* `LEFT JOIN`
* `RIGHT JOIN`
* `CROSS JOIN`
* `UNION ALL`
* `COUNT(*)`
* `ORDER BY`
* `LIMIT`
* Cartesian product

---

## 15. Edge Cases Considered

* If `ads` contains fewer than 3 rows, `top_ads` will contain all available ads.
* If `ads` contains exactly 3 rows, all three standard joins return 3 rows.
* `CROSS JOIN` multiplies the row counts of both tables.
* `UNION ALL` ensures all four JOIN results are preserved.
* The query assumes `id` uniquely identifies an advertisement.

---

## 16. Explanation

I first created a `top_ads` CTE containing the three most popular advertisements. I then calculated the row count produced by each JOIN type and combined the four counts using `UNION ALL`. The key difference is that `INNER`, `LEFT`, and `RIGHT JOIN` depend on matching rows, while `CROSS JOIN` creates every possible combination.
