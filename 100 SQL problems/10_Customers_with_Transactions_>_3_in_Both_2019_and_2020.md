![Medium](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
# Customers with More Than 3 Transactions in Both 2019 and 2020
# Business Context

This analysis is commonly used in **customer retention and loyalty analytics**. Businesses want to identify customers who remained consistently active across multiple years rather than making purchases in only one period.

The result can help teams:

* identify loyal customers,
* build VIP or rewards programs,
* target high-engagement customers with personalized campaigns,
* measure year-over-year customer retention,
* support customer lifetime value analysis.

Customers who transact frequently in consecutive years are often more valuable and less likely to churn.

---

# Problem Statement

Write a query to identify customers who placed **more than three transactions in 2019 and more than three transactions in 2020**.

Return the customer name.

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

## Table: users

| Column | Type    |
| ------ | ------- |
| id     | INTEGER |
| name   | VARCHAR |

---

# Expected Output

| Column        | Type    |
| ------------- | ------- |
| customer_name | VARCHAR |

---

# 1. Understand the Requirement

We need customers who satisfy **both conditions simultaneously**:

* transaction count in **2019 > 3**
* transaction count in **2020 > 3**

The output should contain only the customer name.

---

# 2. Understand the Tables

| Table        | Purpose                               | Important Columns   |
| ------------ | ------------------------------------- | ------------------- |
| transactions | Stores customer purchase transactions | user_id, created_at |
| users        | Stores customer information           | id, name            |

---

# 3. Clarify the Grain

* `transactions`: one row per transaction.
* `users`: one row per user.
* Intermediate result: one row per user with yearly transaction counts.
* Final output: one row per qualifying customer.

---

# 4. Identify Relationships

* `transactions.user_id = users.id`

One user can have many transactions.

---

# 5. Determine the Driving Table

**Driving table:** `transactions`

Reason: transaction activity is the basis of the analysis.

---

# 6. Think About Join Type

Use an **INNER JOIN** between `transactions` and `users`.

Reason: we only need customers who have transactions.

---

# 7. Find the “No Match” Condition

**Not required for this problem.**

---

# 8. Interpret Special Conditions / Notes

Important business rules:

* Count transactions separately for 2019 and 2020.
* The customer must satisfy **both** thresholds.
* Customers with transactions in only one year should be excluded.

---

# 9. Data Quality Considerations

Potential considerations:

* Duplicate transaction rows would increase counts.
* Null transaction dates would not be counted in either year.
* Multiple transactions on the same day are counted separately because the requirement is transaction count, not active days.

---

# 10. Final Calculation Logic

1. Join transactions with users.
2. Group by customer.
3. Count 2019 transactions using conditional aggregation.
4. Count 2020 transactions using conditional aggregation.
5. Keep only customers with counts greater than 3 in both years.
6. Return customer names.

---

# 11. SQL Solution

```sql id=
WITH transaction_counts AS (
SELECT u.id, 
name,
SUM(CASE WHEN YEAR(t.created_at)= '2019' THEN 1 ELSE 0 END) AS t_2019,
SUM(CASE WHEN YEAR(t.created_at)= '2020' THEN 1 ELSE 0 END) AS t_2020
FROM transactions t
JOIN users u
ON u.id = user_id
GROUP BY 1
HAVING t_2019 > 3 AND t_2020 > 3)

SELECT tc.name AS customer_name
FROM transaction_counts tc
```
