![Medium](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
# Identify Customers Who Were Upsold

---

## Business Context

This analysis is useful in **e-commerce, SaaS, and product analytics** to measure whether customers return and purchase additional products after their initial purchase.

An upsold customer can indicate stronger **customer engagement, retention, and product expansion**.

Businesses can use this metric to:

* measure customer retention and repeat purchasing,
* evaluate cross-sell and upsell strategies,
* identify customers with higher lifetime value,
* understand purchasing behavior,
* measure the effectiveness of customer engagement campaigns.

The important business rule is that **multiple purchases on the same day do not count as an upsell** because they are considered part of the same purchasing session/timeframe.

---

## Problem Statement

Given a `transactions` table where each row represents a product purchase, determine the **number of customers who made purchases on more than one distinct day**.

A customer is considered **upsold** if:

```text
First purchase day → Additional purchase on a later day
```

Multiple purchases on the same day should count as **one purchasing day**.

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

| Column                  | Type    |
| ----------------------- | ------- |
| num_of_upsold_customers | INTEGER |

---

## 1. Understand the Requirement

We need to:

1. Identify each customer's purchase dates.
2. Treat multiple purchases on the same day as one purchase occasion.
3. Count the number of distinct purchase days for each customer.
4. Customers with more than one purchase day are considered upsold.
5. Count those customers.

---

## 2. Understand the Tables

| Table        | Purpose                             | Key Columns                     |
| ------------ | ----------------------------------- | ------------------------------- |
| transactions | Stores individual product purchases | user_id, created_at, product_id |

---

## 3. Clarify the Grain

Initially:

* `transactions`: one row per product purchase.

For this analysis:

* Convert transactions to **one row per user per purchase date**.

Then:

* Aggregate to one row per user.
* Count users with more than one distinct purchase day.

---

## 4. Identify Relationships

No joins are required because all required information exists in the `transactions` table.

---

## 5. Determine the Driving Table

**Driving table:** `transactions`

Reason: the transaction date and customer information needed to identify upsells are stored here.

---

## 6. Think About Join Type

No join is required.

---

## 7. Find the "No Match" Condition

There is no explicit no-match condition.

Instead, the key condition is:

```sql id="t7c3sd"
COUNT(*) > 1
```

after reducing the data to one row per user per purchase day.

This means the customer purchased on at least **two different days**.

---

## 8. Interpret Special Conditions / Notes

The critical business rule is:

> Two or more purchases on the same day do not count as an upsell.

Therefore, we first extract:

```sql id="z0q8dc"
DATE(created_at)
```

and group by:

```sql id="4fyv1p"
user_id, DATE(created_at)
```

For example:

| user_id | created_at  |
| ------: | ----------- |
|       1 | Jan 1 10:00 |
|       1 | Jan 1 15:00 |
|       1 | Jan 5 12:00 |

After converting to purchase days:

| user_id | purchase_date |
| ------: | ------------- |
|       1 | Jan 1         |
|       1 | Jan 5         |

The customer has **2 purchase days**, so they count as an upsold customer.

---

## 9. Final Calculation Logic

### Step 1 — Convert timestamps to dates

```sql id="8t8qhl"
DATE(created_at)
```

### Step 2 — Deduplicate purchases on the same day

```sql id="ny5g75"
GROUP BY user_id, DATE(created_at)
```

### Step 3 — Count purchase days per customer

```sql id="1j5w88"
COUNT(*) AS purchase_days
```

### Step 4 — Keep customers with more than one purchase day

```sql id="j1q7zq"
purchase_days > 1
```

### Step 5 — Count those customers.

---

## 10. SQL Solution

```sql id="4otm2w"
SELECT
    COUNT(*) AS num_of_upsold_customers
FROM (
    SELECT
        user_id,
        COUNT(*) AS purchase_days
    FROM (
        SELECT
            user_id,
            DATE(created_at) AS purchase_date
        FROM transactions
        GROUP BY
            user_id,
            DATE(created_at)
    ) AS daily_purchases
    GROUP BY user_id
    HAVING COUNT(*) > 1
) AS upsold_customers;
```

---

## 11. Why Do We Group by Date First?

Suppose a customer has:

| user_id | created_at       | product |
| ------: | ---------------- | ------- |
|       1 | 2020-01-01 10:00 | A       |
|       1 | 2020-01-01 11:00 | B       |
|       1 | 2020-01-01 14:00 | C       |
|       1 | 2020-01-05 09:00 | D       |

Simply counting transactions would give:

```text
4 purchases
```

But from the business perspective, there were only:

```text
January 1 → First purchasing occasion
January 5 → Additional purchasing occasion
```

Therefore, this customer **was upsold**.

The first grouping removes the same-day duplicates.

---

## 12. Alternative and Simpler Solution

The same logic can be written more compactly using `COUNT(DISTINCT DATE())`:

```sql id="j5w1xk"
SELECT
    COUNT(*) AS num_of_upsold_customers
FROM (
    SELECT
        user_id
    FROM transactions
    GROUP BY user_id
    HAVING COUNT(DISTINCT DATE(created_at)) > 1
) AS upsold_customers;
```

This is arguably the cleaner solution because the requirement is specifically about **distinct purchase days**.

---

## 13. Why `COUNT(DISTINCT DATE(created_at))` Works

For each user:

```sql id="3s4rnf"
COUNT(DISTINCT DATE(created_at))
```

directly counts the number of different days on which they purchased.

For example:

```text
User 1:
Jan 1
Jan 1
Jan 5
Jan 10
```

becomes:

```text
3 distinct purchase days
```

Since:

```text
3 > 1
```

the customer is considered upsold.

---

## 14. Output Explanation

| Column                    | Meaning                                                          |
| ------------------------- | ---------------------------------------------------------------- |
| `num_of_upsold_customers` | Number of users who made purchases on more than one distinct day |

---

## 15. Key SQL Concepts Used

* `DATE()`
* `GROUP BY`
* `HAVING`
* `COUNT()`
* `COUNT(DISTINCT)`
* Nested subqueries
* Customer-level aggregation
* Behavioral segmentation

---

## 16. Edge Cases Considered

### Multiple purchases on the same day

Does **not** count as an upsell.

### One purchase ever

Does not count as an upsell.

### Purchases on two different days

Counts as an upsell.

### Purchases across many different days

Still counts as one upsold customer.

---

## 17. Explanation

I considered an upsell to be a customer making purchases on more than one distinct day. I therefore counted distinct purchase dates for each user and filtered users with more than one purchasing day. Multiple transactions on the same day are treated as a single purchasing occasion.
