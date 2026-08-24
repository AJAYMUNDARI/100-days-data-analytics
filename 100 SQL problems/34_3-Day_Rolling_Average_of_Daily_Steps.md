![Medium](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
# 3-Day Rolling Average of Daily Steps

---

## Business Context

For a fitness app, a rolling average helps smooth out **day-to-day fluctuations in user activity** and provides a better view of a user's recent activity trend.

This metric can be useful for:

* monitoring user engagement,
* identifying changes in exercise behavior,
* evaluating fitness goals,
* powering personalized recommendations,
* detecting sustained increases or decreases in activity.

For example, a user may walk 4,000 steps one day and 10,000 the next. A **3-day rolling average** gives a more stable picture of their recent activity.

---

## Problem Statement

Given a `daily_steps` table, calculate the **3-day rolling average of steps for each user**.

Requirements:

* Calculate the average using the current day and previous 2 days.
* The user must have step-count records for all 3 days.
* Exclude the first 2 days for each user.
* Round the average to the nearest whole number.

---

## Input Table

### daily_steps

| Column  | Type    |
| ------- | ------- |
| id      | INTEGER |
| user_id | INTEGER |
| steps   | INTEGER |
| date    | DATE    |

---

## Expected Output

| Column    | Type    |
| --------- | ------- |
| user_id   | INTEGER |
| date      | DATE    |
| avg_steps | INTEGER |

---

## 1. Understand the Requirement

We need to calculate:

```text
Current day steps
+ Previous day steps
+ Two-days-ago steps
--------------------------------
              3
```

For example:

| Date  | Steps | 3-Day Average |
| ----- | ----: | ------------: |
| Jan 1 | 5,000 |             — |
| Jan 2 | 6,000 |             — |
| Jan 3 | 7,000 |         6,000 |
| Jan 4 | 8,000 |         7,000 |

The first two days are excluded because they don't have three days of step data.

---

## 2. Understand the Table

| Table         | Purpose                  | Important Columns          |
| ------------- | ------------------------ | -------------------------- |
| `daily_steps` | Stores daily step counts | `user_id`, `date`, `steps` |

---

## 3. Clarify the Grain

The table is expected to have:

> **One row per user per day.**

The final result is also:

> **One row per user per day where a 3-day average can be calculated.**

---

## 4. Identify Relationships

There is only one table, but we need to compare different dates for the **same user**.

Therefore, we use a **self-join**:

```text
daily_steps t1 → current day
daily_steps t2 → previous day
daily_steps t3 → two days ago
```

---

## 5. Determine the Driving Table

`daily_steps t1` is the driving table.

`t1` represents the **current day** for which we want to calculate the rolling average.

---

## 6. Think About Join Type

We can use `LEFT JOIN` to find the previous two days:

```sql
t1 → t2 → t3
```

Then the `WHERE` clause removes rows where either previous day is missing.

This effectively ensures that only complete 3-day windows remain.

---

## 7. Find the "No Match" Condition

A valid 3-day rolling average requires:

```text
Previous day exists
AND
Two-days-ago exists
```

Therefore:

```sql
WHERE t2.steps IS NOT NULL
  AND t3.steps IS NOT NULL
```

If either is missing, the row is excluded.

---

## 8. Interpret the Date Conditions

For the current day `t1`:

### Previous day

```sql
t1.date = DATE_ADD(t2.date, INTERVAL 1 DAY)
```

This means:

```text
t2.date = t1.date - 1 day
```

### Two days ago

```sql
t1.date = DATE_ADD(t3.date, INTERVAL 2 DAY)
```

This means:

```text
t3.date = t1.date - 2 days
```

So:

```text
t3        t2        t1
↓         ↓         ↓
2 days    1 day     current
ago       ago       day
```

---

## 9. Final Calculation Logic

For every user:

### Step 1

Take the current day's steps:

```sql
t1.steps
```

### Step 2

Find the previous day's steps:

```sql
t2.steps
```

### Step 3

Find the steps from two days ago:

```sql
t3.steps
```

### Step 4

Calculate:

```text
(t1.steps + t2.steps + t3.steps) / 3
```

### Step 5

Round to the nearest whole number.

---

## 10. SQL Solution

```sql
SELECT
    t1.user_id,
    t1.date,
    ROUND(
        (t1.steps + t2.steps + t3.steps) / 3,
        0
    ) AS avg_steps
FROM daily_steps AS t1

LEFT JOIN daily_steps AS t2
    ON t1.user_id = t2.user_id
    AND t1.date = DATE_ADD(t2.date, INTERVAL 1 DAY)

LEFT JOIN daily_steps AS t3
    ON t1.user_id = t3.user_id
    AND t1.date = DATE_ADD(t3.date, INTERVAL 2 DAY)

WHERE t2.steps IS NOT NULL
  AND t3.steps IS NOT NULL;
```

---

## 11. Example Walkthrough

Suppose User 1 has:

| Date  | Steps |
| ----- | ----: |
| Jan 1 | 4,000 |
| Jan 2 | 5,000 |
| Jan 3 | 6,000 |
| Jan 4 | 7,000 |

For Jan 3:

```text
t3 = Jan 1 → 4,000
t2 = Jan 2 → 5,000
t1 = Jan 3 → 6,000
```

Therefore:

```text
(4,000 + 5,000 + 6,000) / 3
= 5,000
```

For Jan 4:

```text
(5,000 + 6,000 + 7,000) / 3
= 6,000
```

Output:

| user_id | date  | avg_steps |
| ------: | ----- | --------: |
|       1 | Jan 3 |     5,000 |
|       1 | Jan 4 |     6,000 |

---

## 12. Why Not Include the First Two Days?

For Jan 1:

```text
Previous day → doesn't exist
Two days ago → doesn't exist
```

For Jan 2:

```text
Previous day → exists
Two days ago → doesn't exist
```

Therefore, neither has the required **3-day window**.

The first valid calculation is Jan 3.

---

## 13. Important Interview Insight

There is an important distinction between:

> **Previous 2 rows**

and

> **Previous 2 calendar days**

This question specifically requires **3 previous days of step counts**, so the self-join approach checks actual calendar dates.

For example:

| Date  | Steps |
| ----- | ----: |
| Jan 1 | 5,000 |
| Jan 2 | 6,000 |
| Jan 4 | 8,000 |

For Jan 4, we **cannot** calculate a 3-day average because Jan 3 is missing.

A simple `ROWS BETWEEN 2 PRECEDING` window could incorrectly treat Jan 1, Jan 2, and Jan 4 as three rows.

---

## 14. Alternative Window Function Approach

If the requirement guarantees **one row per user per calendar day with no missing dates**, a window function provides a cleaner solution:

```sql
SELECT
    user_id,
    date,
    ROUND(
        AVG(steps) OVER (
            PARTITION BY user_id
            ORDER BY date
            ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
        ),
        0
    ) AS avg_steps
FROM daily_steps;
```

But because this question emphasizes **three actual days**, the self-join solution is safer when dates may be missing.

---

## 15. Key SQL Concepts Used

* Self Join
* `LEFT JOIN`
* Date arithmetic
* `DATE_ADD()`
* `ROUND()`
* Rolling average
* Time-series analysis
* Handling missing dates
* `IS NOT NULL`

---

## 16. Edge Cases Considered

* First two days are excluded.
* Missing previous days prevent calculation.
* Users are kept separate using `user_id`.
* The calculation uses exactly three calendar days.
* Multiple users can have independent rolling averages.
* Results are rounded to the nearest whole number.

---

## 17. Explanation

> “I treated `t1` as the current day and self-joined the table to find the previous one and two days for the same user. I then calculated the average of those three step counts and excluded rows where either previous day was missing. This ensures that the rolling average is calculated only when we have a complete three-day window.”

---

## 18. Final Answer Summary

The main SQL pattern is:

```text
Self Join
   ↓
Current Day + Previous Day + Two Days Ago
   ↓
Validate all 3 days exist
   ↓
Calculate Average
   ↓
ROUND()
```

**Key takeaway:** This is a classic **time-series SQL problem**, and the important part is understanding whether "previous 2 days" means **previous 2 rows** or **previous 2 calendar days**.
