Edit

# User Session Duration in 2020

## Business Context

This analysis helps businesses understand **customer engagement duration** within a specific year. By measuring the number of days between a user’s first and last session in 2020, product and marketing teams can identify:

* Highly engaged users who returned over a long period.

* Users who were active only briefly.

* Retention patterns for a given year.

* Customer lifecycle behavior for reporting and segmentation.

This metric is commonly used in **retention analysis, engagement analysis, and customer lifecycle analytics**.

---

## 1. Understand the Requirement

We need to calculate, for each user, the number of days between:

* their **first session in 2020**, and

* their **last session in 2020**.

The expected output should contain:

* `user_id`

* `no_of_days` (difference in days between first and last session)

---

## 2. Understand the Tables

| Table         | Purpose                               | Key Columns                     |
| ------------- | ------------------------------------- | ------------------------------- |
| user_sessions | Stores every user login/session event | session_id, user_id, created_at |

---

## 3. Clarify the Grain

* **user_sessions:** one row per user session.

* **Final output:** one row per user.

---

## 4. Identify Relationships

Only one table is involved, so no joins are required.

---

## 5. Determine the Driving Table

**Driving table:** `user_sessions`

Reason: The required metric (first and last session dates) is derived directly from the session records.

---

## 6. Think About Join Type

Not required because only a single table is used.

---

## 7. Find the “No Match” Condition (if applicable)

**Not required for this problem.**

---

## 8. Interpret Special Conditions / Notes

* Consider only sessions where `created_at` falls in the year **2020**.

* Users with only one session in 2020 will have `no_of_days = 0`.

* `DATEDIFF(end_date, start_date)` returns the difference in days.

---

## 9. Final Calculation Logic

For each `user_id`:

1. Filter rows to year 2020.

2. Find the earliest session date using `MIN(created_at)`.

3. Find the latest session date using `MAX(created_at)`.

4. Calculate the day difference using `DATEDIFF()`.

---

## 10. SQL Solution

```
SELECT
    user_id,
    DATEDIFF(MAX(created_at), MIN(created_at)) AS no_of_days
FROM user_sessions
WHERE YEAR(created_at) = 2020
GROUP BY user_id;
```

---

## 11. Output Explanation

<table><tbody><tr><td><span>Column</span></td><td><span>Meaning</span></td></tr><tr><td><span>user_id</span></td><td><span>Unique identifier of the user</span></td></tr><tr><td><span>no_of_days</span></td><td><span>Number of days between the user’s first and last session in 2020</span></td></tr></tbody></table>

---

## 12. Key SQL Concepts Used

* `WHERE`

* Date filtering with `YEAR()`

* `GROUP BY`

* Aggregate functions (`MIN`, `MAX`)

* `DATEDIFF`

---

## 13. Explanation 

I first filtered the dataset to sessions that occurred in 2020. Then I grouped the data by user and used `MIN(created_at)` and `MAX(created_at)` to identify each user’s first and last session dates. Finally, I calculated the difference in days using `DATEDIFF` to measure user engagement duration during the year.
