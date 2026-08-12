# Days Between First and Last Session in 2020

---

## Business Context

This analysis is commonly used in **user engagement, retention, and customer lifecycle analytics**. Product teams want to understand how long users remain active within a specific time period.

The metric helps businesses:

* measure user retention,
* identify highly engaged users,
* analyze customer activity span,
* compare engagement across cohorts,
* support churn prediction and lifecycle modeling.

A larger number of days generally indicates that a user stayed active for a longer portion of the year.

---

## Problem Statement

For each user, calculate the number of days between their **first session** and **last session** during the year **2020**.

Return:

* `user_id`
* `no_of_days`

---

## Input Tables

### user_sessions

| Column     | Type     |
| ---------- | -------- |
| session_id | INTEGER  |
| created_at | DATETIME |
| user_id    | INTEGER  |

---

## Expected Output

| Column     | Type    |
| ---------- | ------- |
| user_id    | INTEGER |
| no_of_days | INTEGER |

---

## 1. Understand the Requirement

We need to:

1. Consider only sessions that occurred in **2020**.
2. Find each user’s earliest session date in 2020.
3. Find each user’s latest session date in 2020.
4. Calculate the difference in days between those two dates.

---

## 2. Understand the Tables

| Table         | Purpose                      | Key Columns         |
| ------------- | ---------------------------- | ------------------- |
| user_sessions | Stores user session activity | user_id, created_at |

---

## 3. Clarify the Grain

* `user_sessions`: one row per session.
* Final output: one row per user.

---

## 4. Identify Relationships

No joins are required because all required information exists in a single table.

---

## 5. Determine the Driving Table

**Driving table:** `user_sessions`

Reason: session timestamps and user identifiers are stored directly in this table.

---

## 6. Think About Join Type

No join is required.

---

## 7. Find the “No Match” Condition

**Not required for this problem.**

---

## 8. Interpret Special Conditions / Notes

Important business rule:

* Only sessions from the year **2020** should be considered.

This is implemented using:

```sql id=
WHERE YEAR(created_at)=2020
```


## 9. SQL Solution

```sql id=
SELECT user_id,
DATEDIFF(MAX(created_at),MIN(created_at)) AS no_of_days
FROM user_sessions
WHERE YEAR(created_at)=2020
GROUP BY user_id
```
