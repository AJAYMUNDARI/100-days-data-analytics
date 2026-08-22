![Easy](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
# Find Neighborhoods with Zero Users

## 1. Understand the Requirement

We need to identify all neighborhoods that do **not have any users associated with them**.

The output should contain the neighborhood names where the number of users is zero.

---

## 2. Understand the Tables

| Table           | Purpose                                                 | Key Columns             |
| --------------- | ------------------------------------------------------- | ----------------------- |
| `users`         | Stores user details and the neighborhood they belong to | `id`, `neighborhood_id` |
| `neighborhoods` | Stores neighborhood information                         | `id`, `name`            |

---

## 3. Clarify the Grain

* `users`: one row per user.
* `neighborhoods`: one row per neighborhood.
* Final output: one row per neighborhood with zero users.

---

## 4. Identify Relationships

* `users.neighborhood_id = neighborhoods.id`

This is a many-to-one relationship:

* many users can belong to one neighborhood.

---

## 5. Determine the Driving Table

**Driving table:** `neighborhoods`

Reason: We need all neighborhoods, including those that may not have any matching users.

---

## 6. Think About Join Type

Use a **LEFT JOIN** from `neighborhoods` to `users`.

* `LEFT JOIN` keeps every neighborhood.
* Matching users are attached when they exist.
* Non-matching neighborhoods produce `NULL` values for user columns.

---

## 7. Find the “No Match” Condition

After the `LEFT JOIN`, neighborhoods without users will have:

```sql
u.id IS NULL
```

This condition filters only the neighborhoods with zero users.

---

## 8. Interpret Special Conditions / Notes

No additional filters, dates, or business rules are given.

The key requirement is simply to return neighborhoods with **no associated users**.

---

## 9. Final Calculation Logic

1. Start from all neighborhoods.
2. Attach users using `LEFT JOIN`.
3. Keep only rows where no user matched (`u.id IS NULL`).
4. Return the neighborhood name.

---

## 10. SQL Solution

```sql
SELECT n.name
FROM neighborhoods n
LEFT JOIN users u
    ON u.neighborhood_id = n.id
WHERE u.id IS NULL;
```

---

## 11. Output Explanation

| Column | Meaning                                      |
| ------ | -------------------------------------------- |
| `name` | Name of the neighborhood that has zero users |

---

## 12. Key SQL Concepts Used

* `LEFT JOIN`
* Join condition
* `NULL` filtering
* Anti-join pattern

---

## 13. Explanation

I started with the `neighborhoods` table because we need to keep all neighborhoods. I used a `LEFT JOIN` to attach users and then filtered rows where `u.id IS NULL`, which identifies neighborhoods that have no matching users. This is a standard SQL anti-join pattern for finding missing relationships.
