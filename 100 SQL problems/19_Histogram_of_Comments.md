# Histogram of Comments per User in January 2020

---

## Business Context

This analysis is commonly used in **community analytics, product engagement, and social platform reporting**. Product teams often want to understand how actively users participate in discussions during a specific period.

A histogram of comments per user helps answer questions such as:

* How many users were inactive?
* How many users posted exactly 1 comment, 2 comments, 3 comments, etc.?
* Is engagement concentrated among a small group of highly active users?
* Did a campaign or product change increase participation?

Including users with **zero comments** is important because it measures overall engagement, not just active commenters.

---

## Problem Statement

Create a histogram showing the number of comments made per user during **January 2020**.

Requirements:

* Bucket size = 1 comment.
* Users with no comments in January 2020 must appear in the **0 bucket**.

---

## Input Tables

### users

| Column          | Type     |
| --------------- | -------- |
| id              | INTEGER  |
| name            | VARCHAR  |
| created_at      | DATETIME |
| neighborhood_id | INTEGER  |
| mail            | VARCHAR  |

### comments

| Column     | Type     |
| ---------- | -------- |
| user_id    | INTEGER  |
| body       | VARCHAR  |
| created_at | DATETIME |

---

## Expected Output

| Column        | Type    |
| ------------- | ------- |
| comment_count | INTEGER |
| frequency     | INTEGER |

---

## 1. Understand the Requirement

For every user:

1. Count comments created in January 2020.
2. Users with no January comments should receive a count of 0.
3. Group users by their comment count.
4. Count how many users fall into each bucket.

Example:

* 50 users made 0 comments.
* 20 users made 1 comment.
* 8 users made 2 comments.

---

## 2. Understand the Tables

| Table    | Purpose                 | Key Columns         |
| -------- | ----------------------- | ------------------- |
| users    | Stores user information | id                  |
| comments | Stores user comments    | user_id, created_at |

---

## 3. Clarify the Grain

* `users`: one row per user.
* `comments`: one row per comment.
* Intermediate result (`c1`): one row per user with January comment count.
* Final output: one row per comment-count bucket.

---

## 4. Identify Relationships

* `users.id = comments.user_id`

One user can have many comments.

---

## 5. Determine the Driving Table

**Driving table:** `users`

Reason: we must include users who made **zero comments**, so all users need to be retained.

---

## 6. Think About Join Type

Use a **LEFT JOIN** from `users` to `comments`.

Reason:

* Keeps all users.
* Users without January comments remain in the result with a count of 0.

The date filter is placed in the **JOIN condition**, not in the `WHERE` clause, so that users without matching comments are not removed.

---

## 7. Find the “No Match” Condition

Users with no January comments will have `NULL` values from the comments table.

`COUNT(c.user_id)` returns **0** for those users, which naturally creates the 0 bucket.

---

## 8. Interpret Special Conditions / Notes

Important business rules:

* Count only comments created in **January 2020**.
* Comments outside January 2020 must not contribute to the count.
* Bucket intervals are size 1, so every distinct comment count becomes a histogram bucket.

---

## 9. Final Calculation Logic

1. Start with all users.
2. Left join January 2020 comments.
3. Count comments per user.
4. Group users by comment count.
5. Count users in each bucket.

---

## 10. SQL Solution

```sql id=
with c1 as (
            SELECT u.id, count(c.user_id) as comment_count
            FROM users u 
            left join comments c on c.user_id = u.id and c.created_at between '2020-01-01' and '2020-01-31'
            group by 1
           )

select comment_count
       ,count(*) as frequency
from c1
group by 1
```
