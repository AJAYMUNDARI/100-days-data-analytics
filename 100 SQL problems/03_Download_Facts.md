# Average Downloads by Day and Account Type

## Business Context

This query is a common **product analytics / subscription analytics** problem. Companies often compare engagement between **free users and paying customers** to understand product adoption and monetization effectiveness.

The result can be used to:

* measure customer engagement,
* compare usage intensity of free vs paid users,
* evaluate the value delivered to paying customers,
* monitor daily product activity,
* support pricing and conversion strategy decisions.

For example, if paying customers consistently download significantly more content than free users, it may indicate strong product value among subscribers.

---

## 1. Understand the Requirement

We need to calculate the **average number of downloads per account** for:

* free accounts (`paying_customer = FALSE`)
* paying accounts (`paying_customer = TRUE`)

The average must be calculated **for each day separately**.

Additional rule:

* Only accounts that have at least one download should be included. Since the `downloads` table contains download records, joining with this table naturally excludes accounts with no downloads.

The average should be rounded to **2 decimal places**.

---

## 2. Understand the Tables

| Table       | Purpose                         | Key Columns                                |
| ----------- | ------------------------------- | ------------------------------------------ |
| `accounts`  | Stores account type information | `account_id`, `paying_customer`            |
| `downloads` | Stores daily download activity  | `account_id`, `download_date`, `downloads` |

---

## 3. Clarify the Grain

* `accounts`: one row per account.
* `downloads`: one row per account per download date.
* Final output: one row per `(download_date, paying_customer)` combination.

---

## 4. Identify Relationships

* `accounts.account_id = downloads.account_id`

One account can have many download records across different dates.

---

## 5. Determine the Driving Table

**Driving table:** `downloads`

Reason: the analysis is based on download activity, and only accounts with download records should be included.

---

## 6. Think About Join Type

Use an **INNER JOIN** because we only need accounts that have download records.

* `JOIN` automatically removes accounts with no downloads.

---

## 7. Find the “No Match” Condition

Not required for this problem because accounts without downloads are intentionally excluded by the inner join.

---

## 8. Interpret Special Conditions / Notes

Important business rule:

> Consider only accounts that have had at least one download.

The join satisfies this rule because only matching download records are included.

Another requirement is rounding the average to two decimal places using `ROUND(..., 2)`.

---

## 9. Final Calculation Logic

1. Join accounts with downloads using `account_id`.
2. Group records by `download_date` and `paying_customer`.
3. Calculate the average downloads within each group.
4. Round the result to two decimal places.

---

## 10. SQL Solution

```sql
SELECT
    download_date,
    paying_customer,
    ROUND(AVG(downloads), 2) AS average_downloads
FROM accounts a
JOIN downloads b
    ON a.account_id = b.account_id
GROUP BY download_date, paying_customer;
```

---

## 11. Output Explanation

| Column              | Meaning                                              |
| ------------------- | ---------------------------------------------------- |
| `download_date`     | Date of download activity                            |
| `paying_customer`   | Indicates whether the account is free or paying      |
| `average_downloads` | Average downloads for that account type on that date |

---

## 12. Key SQL Concepts Used

* `INNER JOIN`
* Aggregation with `AVG`
* Grouping with `GROUP BY`
* Numeric formatting with `ROUND`

---

## 13. Explanation

I started from the `downloads` table because only accounts with download activity should be included. I joined it with `accounts` to identify whether each account is free or paying, grouped the data by date and account type, and calculated the average downloads rounded to two decimal places.
