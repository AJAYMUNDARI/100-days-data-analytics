![Easy](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
# Number of Users Who Gave a Like on June 6, 2020

---

## Business Context

This analysis is commonly used in **product analytics and engagement reporting**. Social platforms frequently measure how many unique users perform a specific action on a given day.

The metric helps teams:

* track daily engagement,
* monitor feature adoption,
* compare engagement across dates,
* evaluate campaign impact,
* analyze user interaction behavior.

Counting **distinct users** is important because one user may perform the action multiple times.

---

## Problem Statement

Determine how many **different users** performed the action `like` on **2020-06-06`.

Return the result as:

* `num_users_gave_like`

---

## Input Tables

### events

| Column     | Type     |
| ---------- | -------- |
| user_id    | INTEGER  |
| created_at | DATETIME |
| action     | VARCHAR  |
| platform   | VARCHAR  |

---

## Expected Output

| Column              | Type    |
| ------------------- | ------- |
| num_users_gave_like | INTEGER |

---

## 1. Understand the Requirement

We need to count the number of **unique users** who:

1. Performed the action `like`,
2. On the date `2020-06-06`.

Multiple likes by the same user on that date should count only once.

---

## 2. Understand the Tables

| Table  | Purpose                     | Key Columns                 |
| ------ | --------------------------- | --------------------------- |
| events | Stores user activity events | user_id, created_at, action |

---

## 3. Clarify the Grain

* `events`: one row per user action event.
* Final output: a single aggregated value.

---

## 4. Identify Relationships

No joins are required because all required information exists in the `events` table.

---

## 5. Determine the Driving Table

**Driving table:** `events`

Reason: user actions and timestamps are stored directly in this table.

---

## 6. Think About Join Type

No join is required.

---

## 7. Find the “No Match” Condition

**Not required for this problem.**

---

## 8. Interpret Special Conditions / Notes

Important business rules:

* Filter to `action = 'like'`.
* Filter to date `2020-06-06`.
* Count distinct users, not total like events.

---

## 9. Final Calculation Logic

1. Filter events to `2020-06-06`.
2. Filter events where action is `like`.
3. Count distinct `user_id`.

---

## 10. SQL Solution

```sql id=
SELECT COUNT(DISTINCT user_id) AS num_users_gave_like
FROM events
WHERE DATE(created_at) = DATE("2020-06-06")
      AND action = "like"
```
