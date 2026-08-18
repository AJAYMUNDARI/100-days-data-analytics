![Medium](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
# Find the Third Purchase of Every User

---

## Business Context

This analysis is useful in **customer behavior, e-commerce, and lifecycle analytics**. The third purchase is an important milestone that can indicate whether a customer is becoming a repeat or loyal customer.

Businesses can use this analysis to:

* identify customers reaching repeat-purchase milestones,
* analyze time to third purchase,
* understand customer retention,
* build loyalty and lifecycle campaigns,
* identify products associated with repeat customers.

---

## Problem Statement

Find the **third purchase made by every user**.

Requirements:

* Rank purchases separately for each user.
* Sort purchases by `created_at`.
* If two purchases happen at the same time, use the lower `id` to determine which purchase came first.
* Return only the third purchase.
* Sort the final results by `user_id` ascending.

---

## Input Tables

### transactions

| Column     | Type     |
| ---------- | -------- |
| id         | INTEGER  |
| user_id    | INTEGER  |
| created_at | DATETIME |
| product_id | INTEGER  |
| quantity   | INTEGER  |

---

## Expected Output

| Column     | Type     |
| ---------- | -------- |
| user_id    | INTEGER  |
| created_at | DATETIME |
| product_id | INTEGER  |
| quantity   | INTEGER  |

---

## 1. Understand the Requirement

We need to:

1. Identify purchases for each user.
2. Order each user's purchases chronologically.
3. Use `id` as a tie-breaker when purchases have the same timestamp.
4. Assign a sequential purchase number.
5. Select purchase number 3.
6. Sort the final result by `user_id`.

---

## 2. Understand the Tables

| Table        | Purpose                      | Key Columns             |
| ------------ | ---------------------------- | ----------------------- |
| transactions | Stores purchase transactions | user_id, created_at, id |

---

## 3. Clarify the Grain

* `transactions`: one row per purchase transaction.
* Intermediate result: one row per transaction with a purchase sequence number.
* Final output: one row per user who has made at least three purchases.

---

## 4. Identify Relationships

No joins are required because all required information exists in the `transactions` table.

---

## 5. Determine the Driving Table

**Driving table:** `transactions`

Reason: all purchase information required to identify the third purchase is stored in this table.

---

## 6. Think About Join Type

No join is required.

---

## 7. Find the “No Match” Condition

Users who have made fewer than 3 purchases will not have a row with:

```sql
rn = 3
```

Therefore, they are automatically excluded from the final result.

---

## 8. Interpret Special Conditions / Notes

The purchase order is determined by:

```sql
ORDER BY created_at, id
```

This means:

1. Earlier `created_at` → earlier purchase.
2. If timestamps are identical, lower `id` → earlier purchase.

`ROW_NUMBER()` is appropriate because every transaction needs a unique purchase sequence.

---

## 9. Final Calculation Logic

1. Partition transactions by `user_id`.
2. Order each user's transactions by `created_at` and `id`.
3. Assign a sequential number using `ROW_NUMBER()`.
4. Filter for `rn = 3`.
5. Sort by `user_id`.

---

## 10. SQL Solution

```sql
WITH cte AS (
    SELECT
        user_id,
        created_at,
        product_id,
        quantity,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY created_at, id
        ) AS rn
    FROM transactions
)
SELECT
    user_id,
    created_at,
    product_id,
    quantity
FROM cte
WHERE rn = 3
ORDER BY user_id;
```

---

## 11. Why `ROW_NUMBER()`?

`ROW_NUMBER()` assigns a unique sequential number to every purchase within each user.

For example:

| user_id | created_at | id | row_number |
| ------- | ---------- | -: | ---------: |
| 1       | Jan 1      | 10 |          1 |
| 1       | Jan 5      | 15 |          2 |
| 1       | Jan 5      | 20 |          3 |
| 1       | Jan 10     | 25 |          4 |

The transaction with `id = 20` is the third purchase because the first two are determined by `created_at`, and the lower `id` wins when timestamps are identical.

---

## 12. Output Explanation

| Column     | Meaning                         |
| ---------- | ------------------------------- |
| user_id    | User who made the purchase      |
| created_at | Date/time of the third purchase |
| product_id | Product purchased               |
| quantity   | Quantity purchased              |

---

## 13. Key SQL Concepts Used

* `ROW_NUMBER()`
* Window functions
* `PARTITION BY`
* Multiple-column `ORDER BY`
* CTE
* Filtering window-function results

---

## 14. Edge Cases Considered

* Users with fewer than 3 purchases are excluded.
* Multiple purchases at the same timestamp are ordered using `id`.
* Every user gets an independent purchase sequence.
* Users are sorted by `user_id` in the final output.

---

## 15. Explanation

I used `ROW_NUMBER()` partitioned by `user_id` to create a sequential purchase number for each user. I ordered transactions by `created_at` and then `id` to correctly handle purchases occurring at the same time, and finally filtered for the third purchase using `rn = 3`.
