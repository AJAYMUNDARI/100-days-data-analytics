# Booking Counts for Last 90 Days, Last 365 Days, and Overall

# Business Context

This query is commonly used in **travel, hospitality, and reservation analytics**. Businesses track booking activity across different time windows to understand recent demand trends and overall booking performance.

The metrics help teams:

* monitor short-term booking momentum,
* compare recent activity with annual performance,
* support revenue forecasting,
* evaluate marketing campaign impact,
* detect seasonality in booking behavior.

For example, a decline in the last 90 days relative to the last 365 days may indicate weakening recent demand.

---

# Problem Statement

Return three metrics from the `bookings` table:

* total bookings in the **last 90 days**,
* total bookings in the **last 365 days**,
* total bookings overall.

Assume **today is 2022-01-01**.

---

# Input Tables

## Table: bookings

| Column         | Type    |
| -------------- | ------- |
| reservation_id | INTEGER |
| guest_id       | INTEGER |
| check_in_date  | DATE    |
| check_out_date | DATE    |

---

# Expected Output

| Column                | Type    |
| --------------------- | ------- |
| num_bookings_last90d  | INTEGER |
| num_bookings_last365d | INTEGER |
| num_bookings_total    | INTEGER |

---

# 1. Understand the Requirement

We need a single-row result containing three booking counts:

1. Bookings with check-in dates within the last 90 days before 2022-01-01.
2. Bookings with check-in dates within the last 365 days before 2022-01-01.
3. All bookings regardless of date.

---

# 2. Understand the Tables

| Table    | Purpose                        | Important Columns             |
| -------- | ------------------------------ | ----------------------------- |
| bookings | Stores reservation information | reservation_id, check_in_date |

---

# 3. Clarify the Grain

* `bookings`: one row per reservation.
* Final output: a single aggregated row.

---

# 4. Identify Relationships

No joins are required because all required information is present in the `bookings` table.

---

# 5. Determine the Driving Table

**Driving table:** `bookings`

Reason: booking dates and reservation identifiers are stored directly in this table.

---

# 6. Think About Join Type

No join is required.

---

# 7. Find the “No Match” Condition

**Not required for this problem.**

---

# 8. Interpret Special Conditions / Notes

Important business rule:

* Treat **2022-01-01** as the current date.

The last-90-day window begins on **2021-10-03** and the last-365-day window begins on **2021-01-01**.

A clearer approach is to compare `check_in_date` directly with these threshold dates.

---

# 9. Data Quality Considerations

Potential considerations:

* Null `check_in_date` values are excluded from the conditional counts.
* Duplicate reservation rows would increase counts.
* Future bookings after 2022-01-01 should not be included in historical windows.

---

# 10. Final Calculation Logic

1. Count bookings where `check_in_date >= '2021-10-03'`.
2. Count bookings where `check_in_date >= '2021-01-01'`.
3. Count all reservations.

---

# 11. SQL Solution

```sql id=
SELECT 
SUM(CASE WHEN DATE_ADD(check_in_date, INTERVAL 3 MONTH) >= '2022-01-01' THEN 1 ELSE 0 END) AS num_bookings_last90d, 
SUM(CASE WHEN DATE_ADD(check_in_date, INTERVAL 1 YEAR) >= '2022-01-01' THEN 1 ELSE 0 END) AS num_bookings_last365d,
COUNT(reservation_id) AS num_bookings_total
FROM bookings;
```
