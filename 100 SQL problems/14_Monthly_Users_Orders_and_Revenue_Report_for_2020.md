# Monthly Users, Orders, and Revenue Report for 2020
# Business Context

This analysis is a standard **monthly business performance report** used by Finance, Product, and Growth teams. Companies track customer activity, order volume, and revenue over time to understand seasonal trends and overall business health.

The report helps stakeholders:

* monitor monthly active purchasing customers,
* track order growth,
* measure monthly revenue performance,
* compare seasonal demand patterns,
* evaluate the impact of marketing campaigns and promotions.

This is one of the most common e-commerce KPI dashboards.

---

# Problem Statement

Generate a monthly report for the year **2020** showing:

* number of users who placed orders,
* number of transactions,
* total order amount.

Return one row for each month from January through December.

---

# Input Tables

## Table: transactions

| Column     | Type     |
| ---------- | -------- |
| id         | INTEGER  |
| user_id    | INTEGER  |
| created_at | DATETIME |
| product_id | INTEGER  |
| quantity   | INTEGER  |

## Table: products

| Column | Type    |
| ------ | ------- |
| id     | INTEGER |
| name   | VARCHAR |
| price  | FLOAT   |

## Table: users

| Column | Type    |
| ------ | ------- |
| id     | INTEGER |
| name   | VARCHAR |
| sex    | VARCHAR |

---

# Expected Output

| Column        | Type    |
| ------------- | ------- |
| month         | INTEGER |
| num_customers | INTEGER |
| num_orders    | INTEGER |
| order_amt     | FLOAT   |

---

# 1. Understand the Requirement

We need a **monthly summary for 2020** containing:

1. Distinct customers who placed at least one transaction.
2. Total transactions.
3. Total order amount (`quantity × price`).

The report should be grouped by month.

---

# 2. Understand the Tables

| Table        | Purpose                     | Important Columns                         |
| ------------ | --------------------------- | ----------------------------------------- |
| transactions | Stores transaction activity | user_id, created_at, product_id, quantity |
| products     | Stores product prices       | id, price                                 |
| users        | Stores customer information | id                                        |

Note: The `users` table is not required for this calculation because customer counts can be derived from `transactions.user_id`.

---

# 3. Clarify the Grain

* `transactions`: one row per transaction line item.
* `products`: one row per product.
* Final output: one row per month.

---

# 4. Identify Relationships

* `transactions.product_id = products.id`

One product can appear in many transactions.

---

# 5. Determine the Driving Table

**Driving table:** `transactions`

Reason: all three metrics are transaction-based.

---

# 6. Think About Join Type

Use an **INNER JOIN** between `transactions` and `products`.

Reason: every transaction must have a valid product price to calculate revenue.

---

# 7. Find the “No Match” Condition

**Not required for this problem.**

---

# 8. Interpret Special Conditions / Notes

Important business rules:

* Include only transactions from **2020**.
* Group results by calendar month.
* Count distinct users for `num_customers`.
* Revenue is calculated as `quantity × price`.

---

# 9. Data Quality Considerations

Potential considerations:

* Duplicate transaction rows would inflate order counts and revenue.
* Null prices would produce null revenue unless handled explicitly.
* Negative quantities (returns/refunds) would reduce revenue if present.
* Transactions without valid products are excluded by the inner join.

---

# 10. Final Calculation Logic

1. Filter transactions to year 2020.
2. Join products to obtain prices.
3. Extract the month from `created_at`.
4. Count distinct users.
5. Count transaction rows.
6. Sum `quantity × price`.
7. Group by month.

---

# 11. SQL Solution

```sql id=
SELECT MONTH(t.created_at) AS month, 
       COUNT(DISTINCT t.user_id) AS num_customers,
       COUNT(t.id) AS num_orders, 
       SUM(t.quantity * p.price) AS order_amt
FROM transactions t 
JOIN products p ON t.product_id = p.id
WHERE YEAR(created_at) ='2020'
GROUP BY 1
```
