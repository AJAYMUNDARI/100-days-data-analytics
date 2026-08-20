![Medium](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
# Month-over-Month Revenue Change — 2019

---

## Business Context

Month-over-month (MoM) revenue analysis is a common **business and product analytics** metric used to understand how revenue is changing over time.

For a business, this analysis can help:

* monitor monthly revenue growth or decline,
* identify strong and weak months,
* evaluate the impact of promotions or product launches,
* detect seasonal patterns,
* support financial and business forecasting.

The key metric here is the **percentage change in revenue compared with the previous month**.

---

## Problem Statement

Given `transactions` and `products` tables, calculate the **month-over-month percentage change in revenue for each month of 2019**.

Revenue is calculated as:

```text
Revenue = quantity × product price
```

The MoM change is:

```text
(Current Month Revenue - Previous Month Revenue)
------------------------------------------------ × 100
        Previous Month Revenue
```

The result should be rounded to **2 decimal places**.

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

### products

| Column | Type    |
| ------ | ------- |
| id     | INTEGER |
| name   | VARCHAR |
| price  | FLOAT   |

---

## Expected Output

| Column           | Type    |
| ---------------- | ------- |
| month            | INTEGER |
| month_over_month | FLOAT   |

---

## 1. Understand the Requirement

We need to:

1. Calculate revenue for each month in 2019.
2. Revenue = `quantity × price`.
3. Compare each month's revenue with the previous month's revenue.
4. Calculate the percentage change.
5. Round the result to 2 decimal places.
6. Return the month and MoM change.

---

## 2. Understand the Tables

| Table        | Purpose                      | Key Columns                      |
| ------------ | ---------------------------- | -------------------------------- |
| transactions | Stores purchase transactions | product_id, quantity, created_at |
| products     | Stores product prices        | id, price                        |

---

## 3. Clarify the Grain

* `transactions`: one row per transaction.
* After joining: one row per transaction with its product price.
* First aggregation: one row per month.
* Final output: one row per month.

---

## 4. Identify Relationships

The tables are related through:

```sql id="7ojt8v"
transactions.product_id = products.id
```

This allows us to retrieve the price of each purchased product.

---

## 5. Determine the Driving Table

**Driving table:** `transactions`

Reason: each transaction contains the quantity and date needed to calculate revenue.

The `products` table supplies the corresponding product price.

---

## 6. Think About Join Type

Use an **INNER JOIN** because a transaction needs a matching product price to calculate revenue.

```sql id="s3y8yd"
JOIN products p
    ON p.id = t.product_id
```

---

## 7. Find the "No Match" Condition

No special no-match handling is required because the calculation depends on having a matching product price.

---

## 8. Interpret Special Conditions / Notes

The analysis is restricted to:

```sql id="k3td9u"
YEAR(created_at) = 2019
```

Revenue is:

```sql id="w2m7s8"
SUM(quantity * price)
```

Then `LAG()` retrieves the previous month's revenue.

---

## 9. Final Calculation Logic

### Step 1 — Calculate monthly revenue

```text id="3ubv7r"
January   → revenue
February  → revenue
March     → revenue
...
December  → revenue
```

### Step 2 — Get previous month's revenue

Use:

```sql id="f3xw8c"
LAG(revenue) OVER (ORDER BY month)
```

### Step 3 — Calculate MoM change

```text id="jz8g6x"
(Current Revenue - Previous Revenue)
-------------------------------------
        Previous Revenue
```

### Step 4 — Round to 2 decimals.

---

## 10. SQL Solution

```sql id="h1d2v8"
WITH monthly_revenue AS (
    SELECT
        MONTH(t.created_at) AS month,
        SUM(t.quantity * p.price) AS revenue
    FROM transactions AS t
    INNER JOIN products AS p
        ON p.id = t.product_id
    WHERE YEAR(t.created_at) = 2019
    GROUP BY MONTH(t.created_at)
),

revenue_with_previous AS (
    SELECT
        month,
        revenue,
        LAG(revenue) OVER (
            ORDER BY month
        ) AS previous_revenue
    FROM monthly_revenue
)

SELECT
    month,
    ROUND(
        (revenue - previous_revenue) / previous_revenue,
        2
    ) AS month_over_month
FROM revenue_with_previous
ORDER BY month;
```

---

## 11. Why `LAG()`?

`LAG()` allows us to access the **previous month's revenue** without joining the monthly revenue table to itself.

For example:

| Month | Revenue | Previous Revenue |
| ----- | ------: | ---------------: |
| 1     |  10,000 |             NULL |
| 2     |  12,000 |           10,000 |
| 3     |   9,000 |           12,000 |

For February:

```text id="8qzv9a"
(12,000 - 10,000) / 10,000
= 0.20
```

So:

```text id="3o0a6y"
month_over_month = 0.20
```

which represents **20% growth** if interpreted as a ratio.

---

## 12. Important Note About the Output

The provided solution calculates:

```sql id="qv8j0k"
ROUND((budget - prev_budget) / prev_budget, 2)
```

This returns the **percentage change as a decimal ratio**.

For example:

```text id="r4xqz6"
0.20 = 20% increase
-0.10 = 10% decrease
```

If the expected output instead requires `20.00` for 20%, multiply by 100:

```sql id="3c5e2k"
ROUND(
    ((revenue - previous_revenue) / previous_revenue) * 100,
    2
)
```

For reproducing the provided solution, keep the original ratio calculation.

---

## 13. What Happens to January?

January has no previous month **within the 2019 dataset**.

Therefore:

```text id="p2lqkr"
January → previous_revenue = NULL
```

and the MoM calculation returns `NULL`.

This is expected because there is no December 2018 revenue included in the calculation.

---

## 14. Key SQL Concepts Used

* `INNER JOIN`
* `SUM()`
* `LAG()`
* Window functions
* `MONTH()`
* `YEAR()`
* `GROUP BY`
* Revenue calculation
* Percentage change
* `ROUND()`

---

## 15. Edge Cases Considered

* January has no previous month within 2019 → MoM is `NULL`.
* Months with no transactions are not included by the current query.
* If previous month's revenue is 0, division by zero needs to be handled separately.
* Revenue is calculated from `quantity × price`.
* Results are rounded to 2 decimal places.

---

## 16. Explanation

I first aggregated transaction revenue by month for 2019 by joining transactions with products and calculating `quantity × price`. I then used `LAG()` to retrieve the previous month's revenue and calculated the month-over-month change, rounding the result to two decimal places.
