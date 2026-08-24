![Medium](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
# Pivot Exam Scores from Rows into Columns

---

## Business Context

This is a common **education analytics and reporting** problem.

The raw exam data is stored in a **long format**, where each exam is a separate row. For reporting, however, we often want a **wide format**, where each exam becomes a separate column.

This structure makes it easier to:

* compare a student's performance across exams,
* identify missing exams,
* calculate total or average scores,
* build student performance dashboards,
* prepare data for downstream analysis.

The same SQL pattern is widely used in business analytics when converting transactional/category-level data into reporting columns.

---

## Problem Statement

Students must take four exams:

```text
Exam 1
Exam 2
Exam 3
Exam 4
```

Given the `exam_scores` table, create one row per student with:

```text
student_name
exam_1
exam_2
exam_3
exam_4
```

If a student has not taken an exam, the corresponding value should remain `NULL`.

---

## Input Table

### exam_scores

| Column       | Type    |
| ------------ | ------- |
| student_id   | INTEGER |
| student_name | VARCHAR |
| exam_id      | INTEGER |
| score        | INTEGER |

---

## Expected Output

| Column       | Type    |
| ------------ | ------- |
| student_name | VARCHAR |
| exam_1       | INT     |
| exam_2       | INT     |
| exam_3       | INT     |
| exam_4       | INT     |

---

# 1. Understand the Requirement

We need to transform data from:

```text
One row per student + exam
```

into:

```text
One row per student
```

For example:

```text
Anna | 1 | 71
Anna | 2 | 72
Anna | 3 | 73
Anna | 4 | 74
```

should become:

```text
Anna | 71 | 72 | 73 | 74
```

This is essentially a **pivot operation using conditional aggregation**.

---

# 2. Understand the Table

| Table         | Purpose                           | Important Columns                |
| ------------- | --------------------------------- | -------------------------------- |
| `exam_scores` | Stores each student's exam result | `student_id`, `exam_id`, `score` |

There is only one table, so no joins are required.

---

# 3. Clarify the Grain

### Input grain

One row represents:

> One student's score for one exam.

For example:

```text
Anna + Exam 1
Anna + Exam 2
Anna + Exam 3
```

### Output grain

One row represents:

> One student and all of their exam scores.

---

# 4. Identify Relationships

There are no table relationships or joins.

The relationship we care about is:

```text
student
   ↓
exam_id
   ↓
score
```

We need to turn the different `exam_id` values into columns.

---

# 5. Determine the Driving Table

The driving table is:

```sql
exam_scores
```

All information required to create the output exists in this table.

---

# 6. Think About Join Type

No JOIN is required.

Instead, we use **conditional aggregation**.

The pattern is:

```sql
SUM(
    CASE
        WHEN condition THEN value
    END
)
```

---

# 7. Find the "No Match" Condition

Suppose Brian only took Exam 1:

```text
Brian | Exam 1 | 65
```

There are no rows for Exams 2, 3, or 4.

Therefore:

```text
exam_1 → 65
exam_2 → NULL
exam_3 → NULL
exam_4 → NULL
```

The `CASE` expression naturally produces `NULL` when the exam doesn't match.

---

# 8. Interpret the Conditional Logic

For Exam 1:

```sql
CASE
    WHEN exam_id = 1 THEN score
    ELSE NULL
END
```

For Exam 2:

```sql
CASE
    WHEN exam_id = 2 THEN score
    ELSE NULL
END
```

And similarly for Exams 3 and 4.

Because each student takes each exam only once, aggregation is safe.

---

# 9. Final Calculation Logic

### Step 1

Group records by student:

```sql
GROUP BY student_id
```

### Step 2

For Exam 1, keep only the Exam 1 score:

```sql
CASE WHEN exam_id = 1 THEN score END
```

### Step 3

Aggregate it:

```sql
SUM(...)
```

### Step 4

Repeat for Exams 2, 3, and 4.

This effectively converts:

```text
Rows → Columns
```

---

# 10. SQL Solution

```sql
SELECT
    student_name,
    SUM(CASE WHEN exam_id = 1 THEN score ELSE NULL END) AS exam_1,
    SUM(CASE WHEN exam_id = 2 THEN score ELSE NULL END) AS exam_2,
    SUM(CASE WHEN exam_id = 3 THEN score ELSE NULL END) AS exam_3,
    SUM(CASE WHEN exam_id = 4 THEN score ELSE NULL END) AS exam_4
FROM exam_scores
GROUP BY student_id, student_name;
```

### Small improvement over the provided solution

I recommend grouping by both:

```sql
student_id,
student_name
```

rather than only:

```sql
student_id
```

because `student_name` is being selected and should logically be part of the grouping.

---

# 11. Example Walkthrough

Input:

| student_id | student_name | exam_id | score |
| ---------: | ------------ | ------: | ----: |
|        100 | Anna         |       1 |    71 |
|        100 | Anna         |       2 |    72 |
|        100 | Anna         |       3 |    73 |
|        100 | Anna         |       4 |    74 |
|        101 | Brian        |       1 |    65 |

For Anna:

```text
Exam 1 → 71
Exam 2 → 72
Exam 3 → 73
Exam 4 → 74
```

For Brian:

```text
Exam 1 → 65
Exam 2 → NULL
Exam 3 → NULL
Exam 4 → NULL
```

Result:

| student_name | exam_1 | exam_2 | exam_3 | exam_4 |
| ------------ | -----: | -----: | -----: | -----: |
| Anna         |     71 |     72 |     73 |     74 |
| Brian        |     65 |   NULL |   NULL |   NULL |

---

# 12. Why Use `SUM()`?

At first, `SUM()` might look strange because we aren't really trying to add exam scores.

The purpose is **conditional aggregation**.

For Anna's Exam 1:

```text
71
NULL
NULL
NULL
```

`SUM()` returns:

```text
71
```

For Brian's Exam 2:

```text
NULL
```

So the result remains:

```text
NULL
```

Because the question guarantees that each student took each exam only once, there is only one matching score per exam.

---

# 13. Alternative Using `MAX()`

Since there is only one score per student per exam, `MAX()` can also be used:

```sql
SELECT
    student_name,
    MAX(CASE WHEN exam_id = 1 THEN score END) AS exam_1,
    MAX(CASE WHEN exam_id = 2 THEN score END) AS exam_2,
    MAX(CASE WHEN exam_id = 3 THEN score END) AS exam_3,
    MAX(CASE WHEN exam_id = 4 THEN score END) AS exam_4
FROM exam_scores
GROUP BY student_id, student_name;
```

For this problem, `MAX()` arguably communicates the intention better:

> "Give me the score associated with this exam."

But both approaches work because each student takes each exam only once.

---

# 14. Key SQL Concepts Used

* Conditional aggregation
* `CASE WHEN`
* `SUM()`
* `MAX()`
* `GROUP BY`
* Pivoting rows into columns
* Handling `NULL`
* Data transformation

---

# 15. Edge Cases Considered

### Student hasn't taken an exam

Result:

```text
NULL
```

### Student has taken all four exams

All four columns contain scores.

### Student takes each exam only once

No duplicate aggregation issue.

### Score is 0

The result remains `0`, not `NULL`, because `0` is a valid score.

---

# 16. Explanation

> "The source data is in long format, with one row per student and exam. I use conditional aggregation to pivot the exam IDs into separate columns. For each exam, the CASE expression keeps the corresponding score, and the aggregation returns that value while naturally leaving missing exams as NULL."

---

# 17. Summary

The core SQL pattern is:

```text
CASE WHEN exam_id = 1 THEN score END
                ↓
            SUM / MAX
                ↓
             exam_1
```

Repeated for each exam:

```text
exam_id = 1 → exam_1
exam_id = 2 → exam_2
exam_id = 3 → exam_3
exam_id = 4 → exam_4
```

### **Key takeaway**

This is a classic **SQL pivot / conditional aggregation** problem. When you see a question asking you to turn **categories stored as rows into separate columns**, immediately think:

> **`CASE WHEN` + aggregate function + `GROUP BY`**.
