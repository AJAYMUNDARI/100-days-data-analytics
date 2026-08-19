![Medium](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
# Weighted Random Selection of a Driver

---

## Business Context

This analysis is useful in **ride-hailing platforms such as Uber**, where a matching system may need to select drivers based on a weighted probability rather than treating every driver equally.

A weighting score could represent factors such as:

* driver availability,
* reliability,
* proximity,
* service quality,
* business-defined matching priority.

The objective is to ensure that a driver with a higher weight has a **higher probability of being selected**, while still allowing lower-weight drivers to be selected.

---

## Input Tables

### drivers

| Column    | Type    |
| --------- | ------- |
| id        | INTEGER |
| weighting | INTEGER |

---

## Expected Output

| Column | Type    |
| ------ | ------- |
| id     | INTEGER |

---

## 1. Understand the Requirement

We need to:

1. Calculate each driver's weight as a percentage of the total weight.
2. Calculate a cumulative probability for each driver.
3. Generate a random number between 0 and 1.
4. Select the first driver whose cumulative probability is greater than the random number.

---

## 2. Understand the Tables

| Table   | Purpose                                              | Key Columns   |
| ------- | ---------------------------------------------------- | ------------- |
| drivers | Stores available drivers and their selection weights | id, weighting |

---

## 3. Clarify the Grain

* `drivers`: one row per driver.
* First intermediate result: one row per driver containing normalized weight.
* Second intermediate result: one row per driver containing cumulative probability.
* Final output: one randomly selected driver.

---

## 4. Identify Relationships

No joins are required because all required information exists in the `drivers` table.

---

## 5. Determine the Driving Table

**Driving table:** `drivers`

Reason: driver IDs and their weighting values are stored directly in this table.

---

## 6. Think About Join Type

No join is required.

---

## 7. Find the “No Match” Condition

**Not required for the normal case.**

The cumulative probability should eventually reach 1, so a random value between 0 and 1 should find a matching driver.

---

## 8. Interpret Special Conditions / Notes

The most important idea is **normalizing the weights**.

Suppose:

```text
Total weighting = 10 + 20 + 70 = 100
```

Then:

```text
Driver 1 = 10 / 100 = 0.10
Driver 2 = 20 / 100 = 0.20
Driver 3 = 70 / 100 = 0.70
```

Next, calculate cumulative probability:

| Driver | Weight Probability | Cumulative Probability |
| ------ | -----------------: | ---------------------: |
| 1      |               0.10 |                   0.10 |
| 2      |               0.20 |                   0.30 |
| 3      |               0.70 |                   1.00 |

Now imagine `RAND()` generates:

```text
0.25
```

The first cumulative probability greater than `0.25` is `0.30`, so **driver 2** is selected.

---

## 9. Final Calculation Logic

The query works in three conceptual steps:

### Step 1 — Normalize the weighting

```sql
weighting / SUM(weighting) OVER()
```

This converts each driver's weight into a probability.

### Step 2 — Calculate cumulative probability

```sql
SUM(weight_prob) OVER(ORDER BY id)
```

This creates probability intervals.

### Step 3 — Generate random number and select driver

```sql
WHERE cum_weight > RAND()
ORDER BY id
LIMIT 1
```

`RAND()` generates a random value between 0 and 1.

The first driver whose cumulative probability exceeds that value is selected.

---

## 10. SQL Solution

```sql
SELECT id
FROM (
    SELECT
        id,
        SUM(weight_prob) OVER (
            ORDER BY id
        ) AS cum_weight
    FROM (
        SELECT
            id,
            weighting / SUM(weighting) OVER() AS weight_prob
        FROM drivers
    ) AS a1
) AS a2
WHERE cum_weight > RAND()
ORDER BY id
LIMIT 1;
```

---

## 11. Step-by-Step Example

Suppose the table contains:

| id | weighting |
| -: | --------: |
|  1 |        10 |
|  2 |        20 |
|  3 |        70 |

### Step 1: Calculate total weight

```text
10 + 20 + 70 = 100
```

### Step 2: Normalize weights

| id | weighting | weight_prob |
| -: | --------: | ----------: |
|  1 |        10 |        0.10 |
|  2 |        20 |        0.20 |
|  3 |        70 |        0.70 |

### Step 3: Calculate cumulative weight

| id | weight_prob | cum_weight |
| -: | ----------: | ---------: |
|  1 |        0.10 |       0.10 |
|  2 |        0.20 |       0.30 |
|  3 |        0.70 |       1.00 |

The probability intervals are effectively:

```text
0.00 ───── 0.10 ───────── 0.30 ───────────────────────── 1.00
   Driver 1       Driver 2                 Driver 3
```

If:

```text
RAND() = 0.17
```

then:

```text
0.10 < 0.17
0.30 > 0.17  ← first match
```

Therefore, **driver 2** is selected.

---

## 12. Why Cumulative Probability Is Required

We cannot simply compare the weighting with `RAND()`.

The weights need to be converted into **ranges on the 0–1 probability scale**.

For:

```text
10%, 20%, 70%
```

we create:

```text
Driver 1 → 0.00–0.10
Driver 2 → 0.10–0.30
Driver 3 → 0.30–1.00
```

The size of each range represents that driver's probability.

---

## 13. Why `ORDER BY id` Is Important

The cumulative sum needs a deterministic order:

```sql
SUM(weight_prob) OVER (ORDER BY id)
```

Without an ordering, the database cannot reliably construct the cumulative probability intervals.

The actual driver IDs do not determine the probability; they simply provide the order in which the probability ranges are constructed.

---

## 14. Key SQL Concepts Used

* Window functions
* `SUM() OVER()`
* Cumulative window aggregation
* Probability normalization
* `RAND()`
* Nested subqueries
* Random sampling
* `ORDER BY`
* `LIMIT`

---

## 15. Edge Cases Considered

### Zero total weighting

If all drivers have:

```text
weighting = 0
```

then:

```sql
weighting / SUM(weighting) OVER()
```

causes a division-by-zero problem.

The business logic should therefore ensure that the total weighting is greater than zero.

### Negative weighting

Negative weights should generally not be allowed because they do not represent valid probabilities.

### Duplicate driver IDs

`id` should ideally be unique because each row represents one driver.

---

## 16. Explanation

I first normalized each driver's weighting by dividing it by the total weighting, converting the values into probabilities. Then I calculated a cumulative probability using a window function and generated a random number between 0 and 1; the first driver whose cumulative probability exceeded that random number was selected.
