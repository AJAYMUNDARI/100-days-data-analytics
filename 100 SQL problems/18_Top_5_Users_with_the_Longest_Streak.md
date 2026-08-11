# Top 5 Users with the Longest Continuous Visit Streak

---

## Business Context

This analysis is commonly used in **product analytics, retention analysis, and engagement tracking**. Companies monitor consecutive-day activity to identify highly engaged users, evaluate habit formation, and measure platform stickiness.

The result helps teams:

* identify power users,
* design loyalty or streak-based rewards,
* measure retention quality,
* analyze daily active usage patterns,
* target highly engaged users for premium features.

A longer streak indicates stronger user engagement with the platform.

---

## Problem Statement

Find the **top 5 users with the longest continuous streak** of visiting the platform.

A streak is continuous when a user visits the platform **at least once on consecutive calendar days**.

---

## Input Tables

### events

| Column     | Type     |
| ---------- | -------- |
| user_id    | INTEGER  |
| created_at | DATETIME |
| url        | VARCHAR  |

---

## Expected Output

| Column        | Type    |
| ------------- | ------- |
| user_id       | INTEGER |
| streak_length | INTEGER |

---

## 1. Understand the Requirement

For each user:

1. Consider unique visit dates.
2. Identify consecutive-day sequences.
3. Calculate the length of each sequence.
4. Keep the longest streak per user.
5. Return the top 5 users with the highest streak lengths.

---

## 2. Understand the Tables

| Table  | Purpose                  | Key Columns         |
| ------ | ------------------------ | ------------------- |
| events | Stores user visit events | user_id, created_at |

---

## 3. Clarify the Grain

* `events`: one row per event.
* Deduplicated visits: one row per user per date.
* Intermediate streaks: one row per user per streak group.
* Final output: one row per user.

---

## 4. Identify Relationships

No joins are required because all required information is present in a single table.

---

## 5. Determine the Driving Table

**Driving table:** `events`

Reason: visit timestamps are stored directly in this table.

---

## 6. Think About Join Type

No join is required.

---

## 7. Find the “No Match” Condition

**Not required for this problem.**

---

## 8. Interpret Special Conditions / Notes

Important business rules:

* Multiple visits on the same day count as **one day**.
* Streaks are based on **calendar dates**, not timestamps.
* Consecutive means exactly one-day gaps between dates.

---

## 9. Final Calculation Logic

1. Remove duplicate visits within the same day.
2. Assign a row number per user ordered by visit date.
3. Subtract the row number from the visit date to create a stable streak group key.
4. Count rows within each group to get streak lengths.
5. Select the maximum streak per user.
6. Order by streak length descending and return the top 5 users.

---

## 10. SQL Solution

```sql id=
WITH grouped AS (
    SELECT 
        DATE(DATE_ADD(created_at, INTERVAL -ROW_NUMBER() 
            OVER (PARTITION BY user_id ORDER BY created_at) DAY)) AS grp,
        user_id, 
        created_at 
    FROM (
        SELECT * 
        FROM events 
        GROUP BY created_at, user_id) dates
)
SELECT 
    user_id, streak_length 
FROM (
    SELECT user_id, COUNT(*) as streak_length
    FROM grouped
    GROUP BY user_id, grp
    ORDER BY COUNT(*) desc) c
GROUP BY user_id
LIMIT 5
```
