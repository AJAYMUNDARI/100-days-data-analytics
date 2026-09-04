![Medium](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
# Return Only Duplicate Rows from a Users Table

---

## Business Context

This is a common **data quality and data cleaning** problem.

Duplicate records can occur when data is inserted multiple times due to system errors, repeated uploads, ETL issues, or incorrect data-entry processes.

Identifying duplicate rows is important for:

* maintaining data quality,
* preventing double counting,
* cleaning customer databases,
* identifying repeated records,
* preparing reliable datasets for analysis.

The same SQL pattern is widely used when we need to identify **duplicate records while retaining the original rows**.

---

## Problem Statement

Given the `users` table, identify and return only the **duplicate rows**.

For each `id`, the earliest record based on `created_at` should be considered the original record.

Any subsequent record with the same `id` should be treated as a duplicate.

The output should contain:

```text
id
name
created_at
```

---

## Input Table

### users

| Column       | Type     |
| ------------ | -------- |
| `id`         | INTEGER  |
| `name`       | VARCHAR  |
| `created_at` | DATETIME |

---

## Expected Output

| Column       | Type     |
| ------------ | -------- |
| `id`         | INTEGER  |
| `name`       | VARCHAR  |
| `created_at` | DATETIME |

The output should contain **only duplicate records**, not the first/original record.

---

# 1. Understand the Requirement

We need to identify users where the same `id` appears multiple times.

For each `id`:

```text
Earliest record → Original
Later records   → Duplicates
```

For example:

```text
id | created_at
---|-----------
101 | 2026-01-01
101 | 2026-01-05
101 | 2026-01-10
```

should become:

```text
101 | 2026-01-05
101 | 2026-01-10
```

The main challenge is:

> How do we identify the first occurrence and then return only the subsequent occurrences?

The key SQL concept is:

```text
ROW_NUMBER()
```

---

# 2. Understand the Table

| Table   | Purpose             | Important Columns          |
| ------- | ------------------- | -------------------------- |
| `users` | Stores user records | `id`, `name`, `created_at` |

There is only one table, so no JOIN is required.

---

# 3. Clarify the Grain

### Input grain

One row represents:

> One user record created at a particular point in time.

However, the same `id` may appear more than once.

For example:

```text
101 + 2026-01-01
101 + 2026-01-05
```

### Output grain

One row represents:

> One duplicate user record.

The first occurrence of each `id` should not be returned.

---

# 4. Identify Relationships

There are no table relationships because only one table is involved.

The important relationship is within the table:

```text
Same id
  ↓
Multiple records
  ↓
Order by created_at
```

We need to compare records that belong to the same `id`.

---

# 5. Determine the Driving Table

The driving table is:

```sql
users
```

because all required information exists in this table.

We can directly analyze the records using a window function.

Conceptually:

```text
users
  ↓
Partition by id
  ↓
Order by created_at
  ↓
Assign row numbers
  ↓
Keep ranking > 1
```

---

# 6. Think About Join Type

No JOIN is required.

Instead, we use a **window function**:

```sql
ROW_NUMBER()
```

The window function allows us to assign a sequence number to each record within the same `id`.

---

# 7. Find the "No Match" / Required Condition

We need to identify the first occurrence of each `id`.

The logic is:

```sql
ROW_NUMBER() OVER (
    PARTITION BY id
    ORDER BY created_at ASC
)
```

This produces:

| id  | created_at | ranking |
| --- | ---------- | ------: |
| 101 | Jan 1      |       1 |
| 101 | Jan 5      |       2 |
| 101 | Jan 10     |       3 |
| 102 | Jan 2      |       1 |
| 102 | Jan 8      |       2 |

The first record gets:

```text
ranking = 1
```

All subsequent records get:

```text
ranking > 1
```

Therefore, the duplicate condition is:

```sql
ranking > 1
```

---

# 8. Interpret the Conditional Logic

The main logic is:

```sql
ROW_NUMBER() OVER (
    PARTITION BY id
    ORDER BY created_at ASC
)
```

### `PARTITION BY id`

Groups records with the same `id` together.

```text
id = 101
    ↓
101
101
101
```

### `ORDER BY created_at ASC`

Sorts those records from oldest to newest.

```text
Oldest
   ↓
Newest
```

### `ROW_NUMBER()`

Assigns a sequence:

```text
1 → Original
2 → Duplicate
3 → Duplicate
```

Then:

```sql
WHERE ranking > 1
```

returns only the duplicates.

---

# 9. Final Calculation Logic

### Step 1

Read all records from `users`.

```sql
FROM users
```

### Step 2

Partition records by `id`.

```sql
PARTITION BY id
```

### Step 3

Order records by creation time.

```sql
ORDER BY created_at ASC
```

### Step 4

Assign a row number.

```sql
ROW_NUMBER()
```

### Step 5

Create the ranking in a subquery.

```text
users
   ↓
ROW_NUMBER()
   ↓
ranking
```

### Step 6

Keep only:

```sql
ranking > 1
```

Final flow:

```text
Users
  ↓
Group by ID
  ↓
Sort by created_at
  ↓
ROW_NUMBER()
  ↓
1 = Original
2+ = Duplicate
  ↓
WHERE ranking > 1
  ↓
Duplicate rows
```

---

# 10. SQL Solution

```sql
SELECT
    id,
    name,
    created_at
FROM (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY id
            ORDER BY created_at ASC
        ) AS ranking
    FROM users
) AS u
WHERE ranking > 1;
```

---

### Key takeaway

This is a classic **duplicate-record identification using window functions** problem.

When you need to:

> Keep the first record and identify subsequent records as duplicates

think:

```sql
ROW_NUMBER() OVER (
    PARTITION BY [duplicate_key]
    ORDER BY [date/time] ASC
)
```

and then:

```sql
WHERE ranking > 1
```

This approach is particularly useful when you need to **return the actual duplicate rows**, rather than simply finding which IDs are duplicated.
