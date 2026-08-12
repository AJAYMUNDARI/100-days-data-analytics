![Medium](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
# Last Transaction of Each Day

---

## Business Context

This analysis is commonly used in **banking operations, finance reporting, and transaction monitoring**. Financial institutions often need the final transaction of each day to support:

* end-of-day reconciliation,
* cash position reporting,
* audit trails,
* operational monitoring,
* daily balance calculations.

The last transaction represents the most recent activity before the day closes.

---

## Problem Statement

For each calendar day, return the **last transaction** based on transaction timestamp.

Return:

* `id`
* `created_at`
* `transaction_value`

Order the final result by transaction datetime.

---

## Input Tables

### bank_transactions

| Column            | Type     |
| ----------------- | -------- |
| id                | INTEGER  |
| created_at        | DATETIME |
| transaction_value | FLOAT    |

---

## Expected Output

| Column            | Type     |
| ----------------- | -------- |
| created_at        | DATETIME |
| transaction_value | FLOAT    |
| id                | INTEGER  |

---

## 1. Understand the Requirement

We need to:

1. Group transactions by calendar date.
2. Identify the transaction with the latest timestamp within each day.
3. Return that transaction’s id, timestamp, and value.
4. Sort the result chronologically.

---

## 2. Understand the Tables

| Table             | Purpose                         | Key Columns                       |
| ----------------- | ------------------------------- | --------------------------------- |
| bank_transactions | Stores bank transaction records | id, created_at, transaction_value |

---

## 3. Clarify the Grain

* `bank_transactions`: one row per transaction.
* Final output: one row per calendar day.

---

## 4. Identify Relationships

No joins are required because all required information exists in a single table.

---

## 5. Determine the Driving Table

**Driving table:** `bank_transactions`

Reason: transaction timestamps and values are stored directly in this table.

---

## 6. Think About Join Type

No join is required.

---

## 7. Find the “No Match” Condition

**Not required for this problem.**

---

## 8. Interpret Special Conditions / Notes

Important business rule:

* “Last transaction” means the transaction with the **maximum `created_at`** for that calendar date.

To identify this efficiently, we use a window function partitioned by `DATE(created_at)` and ordered by `created_at DESC`.

---

## 9. Final Calculation Logic

1. Extract the calendar date from `created_at`.
2. Rank transactions within each date by descending timestamp.
3. Keep the row with rank 1.
4. Order the final output by `created_at`.

---

## 10. SQL Solution

```sql id=
with cte as (
             select id,
                    created_at,
                    transaction_value,
                    row_number() over(partition by date(created_at) order by created_at desc) as rn
             from bank_transactions
            )

select id,
        created_at,
        transaction_value
from cte 
where rn = 1
```
