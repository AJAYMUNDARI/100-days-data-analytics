![Hard](https://img.shields.io/badge/Difficulty-Hard-red?style=for-the-badge)
# Detect Overlapping Completed Subscription Periods

---

## Business Context

This analysis is commonly used in **subscription management, billing systems, entitlement validation, and fraud monitoring**. Companies need to know whether a user’s subscription period overlaps with another completed subscription period.

The result can help teams:

* detect duplicate or conflicting subscriptions,
* validate billing rules,
* identify potential data quality issues,
* analyze concurrent subscription activity,
* support compliance and audit processes.

An overlap means two subscription periods share at least one common date.

---

## Problem Statement

For each user, return whether their completed subscription period overlaps with the completed subscription period of **any other user**.

Return:

* `user_id`
* `overlap` (`1` = overlap exists, `0` = no overlap)

Only subscriptions with a non-null `end_date` are considered completed.

---

## Input Tables

### subscriptions

| Column     | Type     |
| ---------- | -------- |
| user_id    | INTEGER  |
| start_date | DATETIME |
| end_date   | DATETIME |

---

## Expected Output

| Column  | Type    |
| ------- | ------- |
| user_id | INTEGER |
| overlap | INTEGER |

---

## 1. Understand the Requirement

For every subscription:

1. Consider only completed subscriptions (`end_date IS NOT NULL`).
2. Compare its date range with subscriptions of other users.
3. If any date ranges overlap, mark overlap = 1.
4. Otherwise mark overlap = 0.

---

## 2. Understand the Tables

| Table         | Purpose                         | Key Columns                   |
| ------------- | ------------------------------- | ----------------------------- |
| subscriptions | Stores subscription date ranges | user_id, start_date, end_date |

---

## 3. Clarify the Grain

* `subscriptions`: one row per subscription period.
* Final output: one row per user.

---

## 4. Identify Relationships

The query performs a **self-join**:

* `s1` = current user’s subscription.
* `s2` = other users’ subscriptions.

Relationship condition:

* `s1.user_id != s2.user_id`

---

## 5. Determine the Driving Table

**Driving table:** `subscriptions s1`

Reason: every user subscription must be evaluated for overlap.

---

## 6. Think About Join Type

Use a **LEFT JOIN** from `s1` to `s2`.

Reason:

* Keeps all users even when no overlapping subscription exists.
* Users without matches receive overlap = 0.

---

## 7. Solution

```sql id=
select s1.user_id,
       max(case when s2.user_id is not null then 1 else 0 end)overlap
from subscriptions s1 
left join subscriptions s2 on s2.user_id != s1.user_id
                      and s1.start_date <= s2.end_date
                      and s1.end_date   >= s2.start_date
group by s1.user_id
```
