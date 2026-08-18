![Easy](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
# Distance Traveled by Each User

---

## Business Context

This analysis is useful in **ride-hailing and mobility analytics** to understand how much distance each customer travels.

It can help businesses:

* identify high-usage customers,
* understand customer travel patterns,
* segment users by ride activity,
* support loyalty programs,
* analyze customer value and engagement.

Including users with **no rides** is important so the report represents the entire user base.

---

## Problem Statement

For each user, calculate the **total distance traveled** across all their rides and display users in descending order of distance traveled.

Users who have not taken any rides should have a distance of **0**.

---

## Input Tables

### users

| Column | Type    |
| ------ | ------- |
| id     | INTEGER |
| name   | VARCHAR |

### rides

| Column            | Type    |
| ----------------- | ------- |
| id                | INTEGER |
| passenger_user_id | INTEGER |
| distance          | FLOAT   |

---

## Expected Output

| Column            | Type    |
| ----------------- | ------- |
| name              | VARCHAR |
| distance_traveled | FLOAT   |

---

## 1. Understand the Requirement

We need to:

1. Match each user with their rides.
2. Calculate the total distance traveled by each user.
3. Include users who have no rides.
4. Sort users by total distance in descending order.
5. Use the user's name as a secondary sorting criterion.

---

## 2. Understand the Tables

| Table | Purpose                 | Key Columns                 |
| ----- | ----------------------- | --------------------------- |
| users | Stores user information | id, name                    |
| rides | Stores ride information | passenger_user_id, distance |

---

## 3. Clarify the Grain

* `users`: one row per user.
* `rides`: one row per ride.
* Intermediate aggregation: one row per user.
* Final output: one row per user.

---

## 4. Identify Relationships

`users.id = rides.passenger_user_id`

One user can have **many rides**.

---

## 5. Determine the Driving Table

**Driving table:** `users`

Reason: we need to include **all users**, including users who have never taken a ride.

---

## 6. Think About Join Type

Use a **LEFT JOIN**.

Reason:

* Keeps every user.
* Users without rides will have `NULL` distance values.
* These `NULL` values can be converted to 0 using `IFNULL()` or `COALESCE()`.

---

## 7. Find the “No Match” Condition

When a user has no matching ride:

```sql
rides.passenger_user_id IS NULL
```

Their total distance should be treated as **0**.

---

## 8. Interpret Special Conditions / Notes

Important business rules:

* Sum all ride distances for each user.
* Users with no rides should return 0.
* Sort by distance traveled descending.
* If two users have the same distance, sort their names alphabetically.

---

## 9. Final Calculation Logic

1. Start with all users.
2. Left join their rides.
3. Sum the ride distances per user.
4. Replace `NULL` totals with 0.
5. Sort by total distance descending.
6. Use name ascending as the tie-breaker.

---

## 10. SQL Solution

```sql
SELECT
    u.name,
    IFNULL(SUM(r.distance), 0) AS distance_traveled
FROM users AS u
LEFT JOIN rides AS r
    ON u.id = r.passenger_user_id
GROUP BY u.id, u.name
ORDER BY distance_traveled DESC, u.name ASC;
```

---

## 11. Why the Original Solution Was Slightly Improved

The original query used:

```sql
GROUP BY name
```

Grouping by the user's unique `id` as well is safer:

```sql
GROUP BY u.id, u.name
```

This avoids incorrectly combining different users who happen to have the same name.

Also, ordering by the calculated alias:

```sql
ORDER BY distance_traveled DESC, u.name ASC
```

makes the intent clearer.

---

## 12. Output Explanation

| Column            | Meaning                                           |
| ----------------- | ------------------------------------------------- |
| name              | User's name                                       |
| distance_traveled | Total distance across all rides taken by the user |

---

## 13. Key SQL Concepts Used

* `LEFT JOIN`
* `SUM()`
* `IFNULL()` / `COALESCE()`
* `GROUP BY`
* `ORDER BY`
* Aggregation at user level

---

## 14. Edge Cases Considered

* Users with no rides → distance = 0.
* Users with multiple rides → all distances are summed.
* Multiple users with the same name → handled separately using `user_id`.
* Null ride distances → ignored by `SUM()` and ultimately converted to 0 if no valid distances exist.

---

## 15. Explanation

I used a `LEFT JOIN` from users to rides so that users without any rides are still included. I then summed the ride distance for each user, replaced null totals with zero, and sorted the result by distance traveled descending with the user's name as a tie-breaker.
