![Easy](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
# Identify Users with Low Order Count or Low Spending

---

## Business Context

This is a common **customer analytics and segmentation** problem.

Businesses often analyze customer purchasing behavior to identify users with relatively low order activity or low spending.

This type of analysis can be used to:

* identify low-engagement customers,
* create targeted marketing campaigns,
* design customer retention strategies,
* understand purchasing behavior,
* segment customers based on order frequency and spending.

The same SQL pattern is widely used when combining **transaction data with product information** and applying conditions to aggregated customer-level metrics.

---

## Problem Statement

Given the `users`, `transactions`, and `products` tables, identify users who satisfy **either** of the following conditions:

```text
Number of orders < 3
OR
Total product value < $500
```

The total product value should be calculated as:

```text
quantity × product price
```

Return the names of users who meet at least one of these conditions.

---

## Input Tables

### users

| Column | Type    |
| ------ | ------- |
| `id`   | INTEGER |
| `name` | VARCHAR |
| `sex`  | VARCHAR |

### transactions

| Column       | Type     |
| ------------ | -------- |
| `id`         | INTEGER  |
| `user_id`    | INTEGER  |
| `created_at` | DATETIME |
| `product_id` | INTEGER  |
| `quantity`   | INTEGER  |

### products

| Column  | Type    |
| ------- | ------- |
| `id`    | INTEGER |
| `name`  | VARCHAR |
| `price` | FLOAT   |

---

## Expected Output

| Column            | Type    |
| ----------------- | ------- |
| `users_less_than` | VARCHAR |

---

# 1. Understand the Requirement

We need to identify users who have:

```text
Less than 3 transactions
```

**OR**

```text
Less than $500 total product value
```

The total product value is:

```text
quantity × price
```

The key point is that the conditions are evaluated at the **user level**, not at the individual transaction level.

Therefore, we need to:

1. Connect users with their transactions.
2. Connect transactions with product prices.
3. Calculate total spending for each user.
4. Count the number of transactions for each user.
5. Keep users where either condition is satisfied.

---

# 2. Understand the Tables

| Table          | Purpose                               | Important Columns                         |
| -------------- | ------------------------------------- | ----------------------------------------- |
| `users`        | Stores customer information           | `id`, `name`                              |
| `transactions` | Stores customer purchases             | `id`, `user_id`, `product_id`, `quantity` |
| `products`     | Stores product information and prices | `id`, `price`                             |

The important relationships are:

```text
users
  ↓
user_id
  ↓
transactions
  ↓
product_id
  ↓
products
```

---

# 3. Clarify the Grain

### `users` table grain

One row represents:

> One user.

### `transactions` table grain

One row represents:

> One transaction made by one user for a product.

### `products` table grain

One row represents:

> One product and its price.

### Output grain

One row represents:

> One user who satisfies at least one of the required conditions.

---

# 4. Identify Relationships

The tables are connected as follows:

```text
users.id
   ↓
transactions.user_id
```

and:

```text
transactions.product_id
   ↓
products.id
```

Therefore:

```text
users
   │
   │ user_id
   ↓
transactions
   │
   │ product_id
   ↓
products
```

We need both relationships because the transaction table contains the **quantity**, while the products table contains the **price**.

---

# 5. Determine the Driving Table

The driving table is:

```sql
users
```

because the question asks:

> Which **users** satisfy the conditions?

Starting with `users` also allows us to retain users who have no transactions.

Conceptually:

```text
users
   ↓
transactions
   ↓
products
   ↓
calculate user-level metrics
```

---

# 6. Think About Join Type

We use `LEFT JOIN` from `users`:

```sql
FROM users u
LEFT JOIN transactions t
    ON t.user_id = u.id
LEFT JOIN products p
    ON p.id = t.product_id
```

### Why `LEFT JOIN`?

We want to preserve users even if they have **no transactions**.

A user with no transactions has:

```text
COUNT(transactions) = 0
```

which satisfies:

```text
0 < 3
```

Therefore, that user should be included.

Using `INNER JOIN` could remove users who have no transactions.

---

# 7. Find the "No Match" / Required Condition

We need two user-level metrics.

### Metric 1 — Number of orders

```sql
COUNT(t.id)
```

We need:

```text
COUNT(t.id) < 3
```

### Metric 2 — Total spending

Each transaction contributes:

```text
quantity × price
```

Therefore:

```sql
SUM(t.quantity * p.price)
```

We need:

```text
SUM(quantity × price) < 500
```

The two conditions are connected using:

```sql
OR
```

So the final condition is:

```sql
COUNT(t.id) < 3
OR
SUM(t.quantity * p.price) < 500
```

---

# 8. Interpret the Conditional Logic

The important logic is:

```sql
HAVING
    COUNT(t.id) < 3
    OR
    SUM(t.quantity * p.price) < 500
```

We use `HAVING` rather than `WHERE` because:

```text
COUNT()
SUM()
```

are aggregate functions.

The conditions are evaluated **after grouping users**.

Think of it as:

```text
Individual transactions
        ↓
GROUP BY user
        ↓
COUNT + SUM
        ↓
HAVING
        ↓
Keep qualifying users
```

---

# 9. Final Calculation Logic

### Step 1 — Start with all users

```sql
FROM users u
```

### Step 2 — Connect transactions

```sql
LEFT JOIN transactions t
    ON t.user_id = u.id
```

### Step 3 — Connect products

```sql
LEFT JOIN products p
    ON p.id = t.product_id
```

### Step 4 — Group transactions by user

```sql
GROUP BY u.id, u.name
```

### Step 5 — Count orders

```sql
COUNT(t.id)
```

### Step 6 — Calculate total spending

```sql
SUM(t.quantity * p.price)
```

### Step 7 — Apply the conditions

```sql
HAVING
    COUNT(t.id) < 3
    OR
    SUM(t.quantity * p.price) < 500
```

Final flow:

```text
Users
  ↓
LEFT JOIN Transactions
  ↓
LEFT JOIN Products
  ↓
GROUP BY User
  ↓
COUNT Orders
+
SUM Spending
  ↓
HAVING
  ↓
Less than 3 orders
OR
Less than $500 spending
```

---

# 10. SQL Solution

```sql
SELECT
    u.name AS users_less_than
FROM users u
LEFT JOIN transactions t
    ON t.user_id = u.id
LEFT JOIN products p
    ON p.id = t.product_id
GROUP BY
    u.id,
    u.name
HAVING
    COUNT(t.id) < 3
    OR
    SUM(t.quantity * p.price) < 500;
```

### Note

The original solution groups only by:

```sql
GROUP BY u.name
```

I recommend:

```sql
GROUP BY u.id, u.name
```

because `id` uniquely identifies the user and prevents two different users with the same name from being incorrectly combined.

### Key takeaway

This is a classic **customer-level aggregation + HAVING** problem.

When a SQL question asks you to identify customers based on metrics such as:

```text
Number of orders
Total spending
Average order value
Total quantity
```

think:

> **GROUP BY customer → calculate aggregate metrics → HAVING to filter the aggregated results.**
