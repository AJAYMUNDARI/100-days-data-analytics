![Medium](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
# Top 3 Users by Daily Downloads

---

## Business Context

This analysis is useful in **product analytics, file-hosting platforms, and user engagement reporting**. Tracking the users with the highest daily downloads helps identify power users and understand usage patterns.

The analysis can help businesses:

* identify highly active users,
* monitor daily platform usage,
* understand download behavior,
* support usage-based segmentation,
* identify potential enterprise or high-value customers.

Using `RANK()` is particularly useful because users with the same number of downloads receive the **same rank**.

---

## Problem Statement

Using the `download_facts` table, find the **top 3 users by downloads for each day**.

Requirements:

* Rank users separately for each date.
* Use the `RANK()` window function.
* Include users whose rank is 1, 2, or 3.
* Order the final result by `date`, then `daily_rank`.

---

## Input Tables

### download_facts

| Column    | Type    |
| --------- | ------- |
| user_id   | INTEGER |
| date      | DATE    |
| downloads | INTEGER |

---

## Expected Output

| Column     | Type    |
| ---------- | ------- |
| daily_rank | INTEGER |
| user_id    | INTEGER |
| date       | DATE    |
| downloads  | INTEGER |

---

## 1. Understand the Requirement

We need to:

1. Rank users based on downloads for each day.
2. Highest downloads should receive rank 1.
3. Use `RANK()` so tied users receive the same rank.
4. Keep only users with rank ≤ 3.
5. Sort by date and daily rank.

---

## 2. Understand the Tables

| Table          | Purpose                                 | Key Columns              |
| -------------- | --------------------------------------- | ------------------------ |
| download_facts | Stores daily download activity per user | user_id, date, downloads |

---

## 3. Clarify the Grain

* `download_facts`: one row per user per day.
* Intermediate result: one row per user per day with a calculated rank.
* Final output: top-ranked users for each day.

---

## 4. Identify Relationships

No joins are required because all required information exists in `download_facts`.

---

## 5. Determine the Driving Table

**Driving table:** `download_facts`

Reason: the table already contains daily downloads for each user.

---

## 6. Think About Join Type

No join is required.

---

## 7. Find the “No Match” Condition

**Not required for this problem.**

---

## 8. Interpret Special Conditions / Notes

The important requirement is to rank users **within each date**, not across the entire dataset.

Therefore:

```sql
RANK() OVER (
    PARTITION BY date
    ORDER BY downloads DESC
)
```

creates a separate ranking for every day.

Also, `RANK()` allows tied users to receive the same rank.

---

## 9. Final Calculation Logic

1. Partition data by `date`.
2. Rank users by `downloads DESC`.
3. Assign rank 1 to the highest downloader.
4. Filter ranks ≤ 3.
5. Order by date and rank.

---

## 10. SQL Solution

```sql
SELECT
    user_id,
    date,
    downloads,
    daily_rank
FROM (
    SELECT
        user_id,
        date,
        downloads,
        RANK() OVER (
            PARTITION BY date
            ORDER BY downloads DESC
        ) AS daily_rank
    FROM download_facts
) AS ranked
WHERE daily_rank <= 3
ORDER BY date, daily_rank;
```

---

## 11. Why `RANK()` Instead of `ROW_NUMBER()`?

`RANK()` gives the same rank to users with equal downloads.

For example:

| user_id | downloads | RANK |
| ------- | --------: | ---: |
| 101     |       500 |    1 |
| 102     |       400 |    2 |
| 103     |       400 |    2 |
| 104     |       300 |    4 |

Notice that rank 3 is skipped because two users share rank 2.

This is different from `ROW_NUMBER()`, which would assign a unique number to every row.

---

## 12. Important Interview Insight

Because the requirement says **top 3 ranks**, not necessarily exactly 3 users, `RANK()` can return **more than three users on a day** when there is a tie at rank 3.

For example:

| user_id | downloads | rank |
| ------- | --------: | ---: |
| 1       |       500 |    1 |
| 2       |       400 |    2 |
| 3       |       300 |    3 |
| 4       |       300 |    3 |

Both users 3 and 4 should be included.

---

## 13. Output Explanation

| Column     | Meaning                              |
| ---------- | ------------------------------------ |
| daily_rank | User's download rank for that date   |
| user_id    | User identifier                      |
| date       | Download date                        |
| downloads  | Number of downloads made by the user |

---

## 14. Key SQL Concepts Used

* Window functions
* `RANK()`
* `PARTITION BY`
* `ORDER BY`
* Subquery / derived table
* Filtering window-function results
* Ranking with ties

---

## 15. Edge Cases Considered

* Tied download counts receive the same rank.
* A day may return more than 3 users because of ties.
* Ranking restarts for every date.
* Users with zero downloads can still be ranked if they exist in the table.

---

## 16. Explanation

I used `RANK()` partitioned by date so that users are ranked independently for each day based on downloads in descending order. Since window functions cannot normally be filtered directly in the `WHERE` clause, I calculated the rank in a subquery and then filtered for ranks up to 3.
