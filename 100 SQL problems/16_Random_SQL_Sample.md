![Medium](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
# Randomly Sample One Row from a Very Large Table
---
# Business Context

This problem is common in **data engineering, analytics platforms, experimentation systems, and monitoring pipelines**. When a table contains hundreds of millions of rows, using a naive random query can cause full table scans, heavy sorting, and database throttling.

Typical business use cases include:

* quickly previewing a random customer record,
* validating data quality on large datasets,
* selecting a random record for testing,
* generating unbiased samples for analytics,
* reducing load on production databases.

The goal is to obtain a reasonably random row while minimizing database work.

---

# Problem Statement

The table `big_table` contains more than **100 million rows**.

Return a **random row** without performing an expensive full-table random sort.

---

# Input Tables

## Table: big_table

| Column | Type    |
| ------ | ------- |
| id     | INTEGER |
| name   | VARCHAR |

---

# Expected Output

| Column | Type    |
| ------ | ------- |
| id     | INTEGER |
| name   | VARCHAR |

---

# 1. Understand the Requirement

We need to:

1. Return one random row.
2. Avoid expensive operations such as `ORDER BY RAND()`.
3. Keep the query efficient for a very large table.

---

# 2. Understand the Tables

| Table     | Purpose            | Important Columns |
| --------- | ------------------ | ----------------- |
| big_table | Large source table | id, name          |

---

# 3. Clarify the Grain

* `big_table`: one row per entity.
* Final output: exactly one row.

---

# 4. Identify Relationships

No joins between different business tables are required. The query uses a derived table only to generate a random target id.

---

# 5. Determine the Driving Table

**Driving table:** `big_table`

Reason: the random row must come from this table.

---

# 6. Think About Join Type

Use an **INNER JOIN** between `big_table` and the derived table containing the random id.

Reason: we want rows whose id is greater than or equal to the generated random id.

---

# 7. Find the “No Match” Condition

**Not required for this problem.**

---

# 8. Interpret Special Conditions / Notes

Important performance requirement:

> Do not throttle the database.

The query avoids `ORDER BY RAND()` on the full table and instead:

1. finds `MAX(id)`,
2. generates a random integer between `1` and `MAX(id)`,
3. seeks the first row whose id is greater than or equal to that value.

This is much cheaper when `id` is indexed.

---

# 9. Data Quality Considerations

Potential considerations:

* Gaps in `id` values are allowed.
* Deleted rows create missing ids.
* The returned distribution is approximate rather than perfectly uniform when gaps exist.
* `id` should be indexed, ideally as the primary key.

---

# 10. Final Calculation Logic

1. Compute the maximum id in the table.
2. Generate a random integer in that range.
3. Find the smallest id greater than or equal to the random value.
4. Return that row.

---

# 11. SQL Solution

```sql id=
SELECT r1.id, r1.name
FROM big_table AS r1 
JOIN (SELECT CEIL(RAND() * (SELECT MAX(id)
                            FROM big_table)
                         ) AS id
                ) AS r2 ON r1.id >= r2.id
ORDER BY r1.id ASC
LIMIT 1
```
