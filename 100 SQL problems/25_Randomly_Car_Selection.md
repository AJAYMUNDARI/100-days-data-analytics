![Easy](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)
# Randomly Select a Car Manufacturer with Equal Probability

---

## Business Context

This type of query can be useful in **experimentation, sampling, testing, and randomized analysis**.

For example, a business may want to:

* randomly select a manufacturer for testing,
* create unbiased samples,
* distribute test cases across manufacturers,
* perform randomized QA or exploratory analysis.

The important requirement is that **each manufacturer should have the same probability of being selected**, regardless of how many cars that manufacturer has in the table.

---

## Problem Statement

Given a `cars` table, randomly select **one manufacturer** such that every distinct manufacturer has an **equal probability** of being selected.

For example, even though Honda has 3 cars and Toyota has 2 cars, the probability should be:

```text
Ford   → 1/3
Toyota → 1/3
Honda  → 1/3
```

---

## Input Tables

### cars

| Column | Type    |
| ------ | ------- |
| id     | INTEGER |
| make   | VARCHAR |

---

## Expected Output

| Column | Type    |
| ------ | ------- |
| make   | VARCHAR |

---

## 1. Understand the Requirement

We need to:

1. Get all unique manufacturer names.
2. Randomly order those names.
3. Select one manufacturer.
4. Ensure every unique manufacturer has an equal chance of being selected.

---

## 2. Understand the Tables

| Table | Purpose                    | Key Columns |
| ----- | -------------------------- | ----------- |
| cars  | Stores vehicle information | id, make    |

---

## 3. Clarify the Grain

* `cars`: one row per car.
* After `DISTINCT`: one row per manufacturer.
* Final output: one randomly selected manufacturer.

---

## 4. Identify Relationships

No joins are required because all required information exists in the `cars` table.

---

## 5. Determine the Driving Table

**Driving table:** `cars`

Reason: manufacturer names are stored directly in this table.

---

## 6. Think About Join Type

No join is required.

---

## 7. Find the “No Match” Condition

**Not required for this problem.**

---

## 8. Interpret Special Conditions / Notes

The key requirement is **equal probability by manufacturer**, not by car.

Therefore, we must first use:

```sql
DISTINCT make
```

before applying random ordering.

If we randomly ordered all cars first, manufacturers with more cars would have a higher probability of being selected.

---

## 9. Final Calculation Logic

1. Select distinct manufacturer names.
2. Randomly order the manufacturers.
3. Return the first manufacturer.

---

## 10. SQL Solution

```sql
SELECT DISTINCT
    make
FROM cars
ORDER BY RAND()
LIMIT 1;
```

---

## 11. Why `DISTINCT` Is Important

Consider the input:

| make   |
| ------ |
| Ford   |
| Toyota |
| Toyota |
| Honda  |
| Honda  |
| Honda  |

Without `DISTINCT`, Honda appears 3 times, so it would have a higher probability of being selected.

With:

```sql
DISTINCT make
```

we get:

| make   |
| ------ |
| Ford   |
| Toyota |
| Honda  |

Now each manufacturer has an equal probability of selection.

---

## 12. Output Explanation

The query returns **one randomly selected manufacturer**.

Possible results:

```text
Ford
```

or

```text
Toyota
```

or

```text
Honda
```

Each has approximately a **33.33% probability**.

---

## 13. Key SQL Concepts Used

* `DISTINCT`
* `ORDER BY RAND()`
* `LIMIT`
* Random sampling
* Deduplication

---

## 14. Edge Cases Considered

* Duplicate cars from the same manufacturer do not increase its probability.
* If only one manufacturer exists, it will always be selected.
* If the table is empty, no row will be returned.
* `RAND()` produces a different random ordering each execution in MySQL.

---

## 15. Explanation

I first selected distinct manufacturer names so that the number of cars from a manufacturer does not affect its probability. Then I randomly ordered those unique manufacturers using `RAND()` and selected the first one with `LIMIT 1`.
