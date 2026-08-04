# Count Users Who Gave a Like on 6 June 2020

## Business Context

This query helps product and marketing teams measure **daily user engagement**. Counting unique users who performed a **like** action on a specific date is a common KPI used in social media and content platforms to track active users, campaign performance, and engagement trends.

---

## 1. Understand the Requirement

We need to find how many **different users** performed the action **like** on **2020-06-06**.

Expected output:

* One row with the count of unique users.
* Output column: `num_users_gave_like`.

---

## 2. Understand the Tables

| Table  | Purpose                                              | Key Columns                           |
| ------ | ---------------------------------------------------- | ------------------------------------- |
| events | Stores every user activity performed on the platform | user_id, created_at, action, platform |

---

## 3. Clarify the Grain

* **events:** one row per user event/action.
* **Final output:** one row containing the total number of distinct users who liked on the given date.

---

## 4. Identify Relationships

No relationships are required because only one table is used.

---

## 5. Determine the Driving Table

**Driving table:** `events`

Reason: The required metric is directly derived from event records.

---

## 6. Think About Join Type

No joins are required for this problem.

---

## 7. Find the “No Match” Condition (if applicable)

**Not required for this problem.**

---

## 8. Interpret Special Conditions / Notes

* Consider only events where `action = 'like'`.
* Consider only events that occurred on **2020-06-06**.
* A user may like multiple times; each user should be counted only once.

---

## 9. Final Calculation Logic

1. Filter rows for the date **2020-06-06**.
2. Filter rows where the action is **like**.
3. Count distinct `user_id` values.

---

## 10. SQL Solution

```sql
SELECT COUNT(DISTINCT user_id) AS num_users_gave_like
FROM events
WHERE DATE(created_at) = DATE('2020-06-06')
  AND action = 'like';
```

---

## 11. Output Explanation

| Column              | Meaning                                                           |
| ------------------- | ----------------------------------------------------------------- |
| num_users_gave_like | Number of unique users who performed a like action on 6 June 2020 |

---

## 12. Key SQL Concepts Used

* `WHERE`
* Date filtering using `DATE()`
* `COUNT(DISTINCT ...)`
* Aggregate functions

---

## 13. Explanation

I first filtered the events table for records where the action was **like** and the event date was **2020-06-06**. Then I used `COUNT(DISTINCT user_id)` to ensure each user was counted only once, even if they performed multiple like actions on that day.
