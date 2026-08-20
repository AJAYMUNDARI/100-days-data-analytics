![Medium](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
# Find the Two Students with the Closest SAT Scores

---

## Business Context

This type of analysis can be useful in **education analytics and student performance analysis**. Identifying students with similar test scores can help with:

* creating balanced study groups,
* peer comparison,
* tutoring and mentoring programs,
* analyzing score distributions,
* identifying students with nearly identical performance.

The key analytical challenge is finding the **minimum score difference between any two different students**.

---

## Problem Statement

Given a `scores` table, find the **two students whose SAT scores are closest together**.

Requirements:

* Compare every possible pair of different students.
* Calculate the absolute difference between their scores.
* Return the pair with the smallest score difference.
* If multiple pairs have the same minimum difference, select the pair whose student-name combination comes first alphabetically.
* Return the student names and score difference.

---

## Input Tables

### scores

| Column  | Type    |
| ------- | ------- |
| id      | INTEGER |
| student | VARCHAR |
| score   | INTEGER |

---

## Expected Output

| Column        | Type    |
| ------------- | ------- |
| one_student   | VARCHAR |
| other_student | VARCHAR |
| score_diff    | INTEGER |

---

## 1. Understand the Requirement

We need to:

1. Compare students against other students.
2. Calculate the absolute score difference for each pair.
3. Avoid comparing a student with themselves.
4. Avoid generating duplicate pairs.
5. Sort pairs by score difference.
6. Use the student name as the tie-breaker.
7. Return only the closest pair.

---

## 2. Understand the Tables

| Table  | Purpose                         | Key Columns        |
| ------ | ------------------------------- | ------------------ |
| scores | Stores each student's SAT score | id, student, score |

---

## 3. Clarify the Grain

* `scores`: one row per student.
* Self-join result: one row per unique pair of students.
* Final output: exactly one student pair.

---

## 4. Identify Relationships

No traditional foreign-key relationship exists.

Instead, the table is **self-joined**:

```sql
scores s1
JOIN scores s2
```

Each row from `s1` is compared against another row from `s2`.

---

## 5. Determine the Driving Table

**Driving table:** `scores s1`

The same table is joined to itself as `s2` to generate student pairs.

---

## 6. Think About Join Type

Use an **INNER JOIN** because we only want valid pairs of different students.

```sql
JOIN scores s2
```

---

## 7. Find the "No Match" Condition

A student cannot be compared with themselves.

Therefore:

```sql
s1.id != s2.id
```

However, this alone would create duplicate pairs:

```text
Alice → Bob
Bob → Alice
```

So the additional condition:

```sql
s1.id < s2.id
```

ensures each pair appears only once.

---

## 8. Interpret Special Conditions / Notes

The key challenge is avoiding duplicate student combinations.

For example, without:

```sql
s1.id < s2.id
```

we could get:

```text
Alice, Bob
Bob, Alice
```

Both represent the same pair.

Using:

```sql
s1.id < s2.id
```

keeps only one representation.

The score difference is calculated using:

```sql
ABS(s1.score - s2.score)
```

---

## 9. Final Calculation Logic

1. Self-join the `scores` table.
2. Ensure the two IDs are different.
3. Use `s1.id < s2.id` to eliminate duplicate pairs.
4. Calculate the absolute score difference.
5. Sort by score difference ascending.
6. Use student name as the tie-breaker.
7. Return the first row.

---

## 10. SQL Solution

```sql
SELECT
    s1.student AS one_student,
    s2.student AS other_student,
    ABS(s1.score - s2.score) AS score_diff
FROM scores AS s1
JOIN scores AS s2
    ON s1.id < s2.id
ORDER BY
    score_diff ASC,
    one_student ASC
LIMIT 1;
```

---

## 11. Why `s1.id < s2.id`?

This condition performs two jobs:

### Prevents self-comparison

A student's ID cannot be less than itself.

### Removes duplicate pairs

Instead of:

```text
Alice → Bob
Bob → Alice
```

we only keep one pair.

For example:

```text
s1.id = 1
s2.id = 2
```

is valid because:

```text
1 < 2
```

But:

```text
s1.id = 2
s2.id = 1
```

is excluded because:

```text
2 < 1
```

is false.

---

## 12. Tie-Breaking Logic

Suppose two pairs have the same score difference:

| one_student | other_student | score_diff |
| ----------- | ------------- | ---------: |
| Alice       | David         |          5 |
| Bob         | Charlie       |          5 |

The requirement says to select the student-name combination that is **higher in the alphabet**.

This part is important because the exact interpretation of "higher in the alphabet" can vary depending on the SQL question's expected ordering.

The provided solution uses:

```sql
ORDER BY score_diff, one_student
```

which selects the alphabetically earlier `one_student`.

If the requirement literally means **alphabetically later**, use:

```sql
ORDER BY score_diff ASC, one_student DESC
```

For reproducing the provided solution, use the first version.

---

## 13. Output Explanation

| Column        | Meaning                                      |
| ------------- | -------------------------------------------- |
| one_student   | First student in the selected pair           |
| other_student | Second student in the selected pair          |
| score_diff    | Absolute difference between their SAT scores |

---

## 14. Key SQL Concepts Used

* Self join
* `INNER JOIN`
* Pair generation
* `ABS()`
* Duplicate-pair elimination
* `ORDER BY`
* `LIMIT`
* Tie-breaking

---

## 15. Edge Cases Considered

* A student is never compared with themselves.
* Duplicate student pairs are removed.
* Students with identical scores have a difference of `0`.
* If multiple pairs have the same minimum difference, the specified alphabetical ordering determines the result.
* At least two students are assumed to exist.

---

## 16. Explanation

I used a self-join to generate every unique pair of students and calculated the absolute difference between their SAT scores. The condition `s1.id < s2.id` prevents self-comparisons and duplicate pairs. I then sorted by the score difference and applied the required alphabetical tie-breaker, returning the first pair.
