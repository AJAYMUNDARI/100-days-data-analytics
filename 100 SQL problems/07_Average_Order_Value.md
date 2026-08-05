# Average Order Value

---

# Business Context

Explain how this problem relates to a real business scenario.

Include points such as:

* Why a business would ask this question.
* Which team might use this analysis (Product, Finance, HR, Marketing, Operations, etc.).
* What decision could be made from the result.
* What business insight the metric provides.

Example:

This analysis helps the Product team compare engagement between free and paid users to understand customer value and support pricing decisions.

---

# 1. Understand the Requirement

Describe in simple business language:

* What needs to be calculated.
* What the final output should contain.
* Any important conditions mentioned in the question.

---

# 2. Understand the Tables

| Table      | Purpose               | Key Columns     |
| ---------- | --------------------- | --------------- |
| table_name | What the table stores | id, foreign_key |

Add one row for each table.

---

# 3. Clarify the Grain

State the level of detail for each table.

Example:

* users: one row per user.
* orders: one row per order.
* order_items: one row per product within an order.

Then specify the grain of the final output.

Example:

* Final output: one row per user.

---

# 4. Identify Relationships

List the join relationships.

Example:

* orders.user_id = users.id
* order_items.order_id = orders.id

Mention whether the relationship is one-to-many, many-to-one, etc., if relevant.

---

# 5. Determine the Driving Table

Identify the table from which the query should start.

**Driving table:** table_name

Explain why this table is the correct starting point.

---

# 6. Think About Join Type

For each join, explain the reason.

Example:

* INNER JOIN → keep only users with orders.
* LEFT JOIN → keep all users even if they have no orders.
* RIGHT JOIN → rarely used; keep all records from the right table.
* FULL JOIN → keep records from both sides.

---

# 7. Find the “No Match” Condition

If the problem asks for missing records, explain how they are identified.

Example:

```sql
WHERE o.id IS NULL
```

If not required, write:

**Not required for this problem.**

---

# 8. Interpret Special Conditions / Notes

Explain all business rules and filters.

Examples:

* Consider only year 2020.
* Ignore cancelled orders.
* Use distinct salaries.
* Include only active customers.
* Round the result to 2 decimal places.

Mention any assumptions if the question is ambiguous.

---

# 9. Final Calculation Logic

Explain the calculation step by step.

Example:

1. Filter orders to completed status.
2. Join with customers.
3. Calculate order amount = quantity × price.
4. Aggregate by customer.
5. Compute the average.

Keep it concise and logical.

---

# 10. SQL Solution

```sql
SELECT
    u.sex
    , ROUND(AVG(quantity  *price), 2) AS aov
FROM users AS u
INNER JOIN transactions AS t
   ON u.id = t.user_id
INNER JOIN products AS p
    ON t.product_id = p.id
GROUP BY 1
```

Use clean formatting and aliases.

---

# 11. Output Explanation

| Column      | Meaning                        |
| ----------- | ------------------------------ |
| column_name | Business meaning of the column |

Explain every output column.

---

# 12. Key SQL Concepts Used

List the SQL concepts demonstrated.

Example:

* INNER JOIN
* LEFT JOIN
* GROUP BY
* HAVING
* CASE WHEN
* DISTINCT
* CTE
* Subquery
* Window Function
* DATEDIFF
* ROUND
* ORDER BY
* LIMIT / OFFSET

---

# 13. Edge Cases Considered

Mention important edge cases.

Examples:

* Multiple employees sharing the highest salary.
* Users with only one session.
* Accounts with no downloads.
* Null values.
* Duplicate transactions.
* Zero quantity or zero price.

If none are relevant, write:

**No special edge cases beyond the stated requirements.**

---

# 14. Query Complexity / Optimization Notes

Briefly discuss performance.

Examples:

* Filtering before aggregation reduces processed rows.
* Indexes on join keys improve performance.
* Avoid unnecessary DISTINCT operations.
* Aggregation occurs after filtering.

Keep this section short.

---

# 15. Alternative Approach (Optional)

Provide another valid SQL approach if useful.

Example:

* Using a window function instead of LIMIT/OFFSET.
* Using a CTE for readability.

---

# 16. Explanation

I first identified the driving table and clarified the output grain. Then I applied the appropriate joins and business filters, performed the required aggregation, and returned the final metric in the requested format.
