# Percentage of Transactions Sent to Home Address

# Business Context

This analysis is useful for **customer behavior analytics, logistics optimization, and fraud monitoring**. Businesses want to understand whether customers usually ship orders to their registered home address or to alternate addresses.

The metric can help teams:

* measure customer address loyalty,
* improve delivery planning,
* identify gifting or workplace delivery behavior,
* detect unusual shipping patterns,
* support fraud and risk monitoring.

A high percentage suggests that most customers prefer their primary address, while a lower percentage indicates significant use of alternate shipping locations.

---

# Problem Statement

Determine whether users tend to order more to their primary address versus other addresses.

Return the percentage of transactions shipped to the user’s home address as `home_address_percent`.

---

# Input Tables

## Table: transactions

| Column           | Type     |
| ---------------- | -------- |
| id               | INTEGER  |
| user_id          | INTEGER  |
| created_at       | DATETIME |
| shipping_address | VARCHAR  |

## Table: users

| Column  | Type    |
| ------- | ------- |
| id      | INTEGER |
| name    | VARCHAR |
| address | VARCHAR |

---

# Expected Output

| Column               | Type  |
| -------------------- | ----- |
| home_address_percent | FLOAT |

---

# 1. Understand the Requirement

We need to calculate:

**(Number of transactions shipped to the customer’s registered address) ÷ (Total transactions)**

Return the result rounded to **2 decimal places**.

---

# 2. Understand the Tables

| Table        | Purpose                                        | Important Columns         |
| ------------ | ---------------------------------------------- | ------------------------- |
| transactions | Stores transaction and shipping information    | user_id, shipping_address |
| users        | Stores customer profile and registered address | id, address               |

---

# 3. Clarify the Grain

* `transactions`: one row per transaction.
* `users`: one row per user.
* Final output: a single overall percentage value.

---

# 4. Identify Relationships

* `transactions.user_id = users.id`

One user can have many transactions.

---

# 5. Determine the Driving Table

**Driving table:** `transactions`

Reason: the analysis is transaction-based, and each transaction contributes to the numerator or denominator.

---

# 6. Think About Join Type

Use an **INNER JOIN** between `transactions` and `users`.

Reason: only transactions with a valid user record can be evaluated against the user’s home address.

---

# 7. Find the “No Match” Condition

**Not required for this problem.**

---

# 8. Interpret Special Conditions / Notes

Important business rules:

* Compare `shipping_address` with the user’s registered `address`.
* Count matching transactions as home-address orders.
* Divide by total transactions.
* Round the final percentage to 2 decimal places.

---

# 9. Data Quality Considerations

Potential considerations:

* Address formatting differences (e.g., abbreviations, capitalization, spacing) may affect equality comparisons.
* Null addresses would not count as home-address matches.
* Duplicate transaction rows would increase both numerator and denominator.

In production systems, address standardization is often required before comparison.

---

# 10. Final Calculation Logic

1. Join transactions with users.
2. Check whether `shipping_address = address`.
3. Count matching transactions.
4. Count total transactions.
5. Divide matches by total transactions.
6. Round the result to two decimal places.

---

# 11. SQL Solution

```sql id=
SELECT ROUND(
             SUM(
                 CASE WHEN u.address = t.shipping_address THEN 1 END)
                 / COUNT(t.id) ,2
                 ) as home_address_percent
FROM transactions as t
JOIN users as u ON t.user_id = u.id
```
