![Medium](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
# Sessionize User Events — 60-Minute Session Window

---

## Business Context

Sessionization is a very common concept in **product and web analytics**.

Instead of analyzing individual events independently, we group consecutive user activities into **sessions**. This allows a business to understand:

* how often users visit the platform,
* how long users stay active,
* how many events happen per session,
* user engagement patterns,
* conversion rates per session,
* session-level funnels.

For example, a product analyst might ask:

> "How many sessions does an average user have per day?"

or:

> "What percentage of sessions result in a purchase?"

To answer these questions, we first need to identify the boundaries between sessions.

---

## Problem Statement

Given an `events` table, assign a **session number** to every event.

A new session starts when the gap between consecutive events for the same user is **greater than 60 minutes**.

### Example

```text
00:01
00:30
01:01
```

The gaps are:

```text
00:01 → 00:30 = 29 minutes
00:30 → 01:01 = 31 minutes
```

Therefore:

```text
Session 1
```

But:

```text
00:01
00:30
01:31
```

has:

```text
00:01 → 00:30 = 29 minutes
00:30 → 01:31 = 61 minutes
```

Therefore:

```text
Session 1
Session 1
Session 2
```

---

## Input Table

### events

| Column     | Type     |
| ---------- | -------- |
| id         | INTEGER  |
| created_at | DATETIME |
| user_id    | INTEGER  |
| event      | VARCHAR  |

---

## Expected Output

| Column     | Type     |
| ---------- | -------- |
| created_at | DATETIME |
| user_id    | INTEGER  |
| event      | VARCHAR  |
| session_id | INTEGER  |

---

# 1. Understand the Requirement

We need to:

1. Look at events separately for each user.
2. Sort each user's events chronologically.
3. Compare each event with the **next event**.
4. Identify where a new session starts.
5. Assign the same session number to all events belonging to that session.
6. Generate the session number using a cumulative sum.

The key condition is:

```text
Gap > 60 minutes → New session
Gap ≤ 60 minutes → Same session
```

---

# 2. Understand the Table

| Table    | Purpose              | Important Columns                |
| -------- | -------------------- | -------------------------------- |
| `events` | Stores user activity | `user_id`, `created_at`, `event` |

No joins are required.

---

# 3. Clarify the Grain

The original table has:

> **One row per user event.**

The final result maintains the same grain:

> **One row per user event**, with an additional `session_id`.

For example:

| user_id | created_at | event    | session_id |
| ------: | ---------- | -------- | ---------: |
|       1 | 00:01      | login    |          1 |
|       1 | 00:30      | click    |          1 |
|       1 | 01:01      | purchase |          1 |
|       1 | 03:00      | logout   |          2 |

---

# 4. Identify Relationships

There are no table relationships because this is a **single-table analytical problem**.

The important relationship is between:

```text
Current event
       ↓
Next event for the same user
```

---

# 5. Determine the Driving Table

The driving table is:

```sql
events
```

We analyze events independently for each `user_id`.

---

# 6. Think About Join Type

No JOIN is required.

Instead, we use a **window function**:

```sql
LEAD()
```

`LEAD()` allows us to look at the next event for the same user.

---

# 7. Find the "No Match" / New Session Condition

A new session occurs when:

```text
Gap between events > 60 minutes
```

There is also an important edge case:

> The final event for a user has no next event.

Therefore, the final event must also be treated as a session boundary.

Conceptually:

```text
IF gap > 60 minutes
    → new session

OR

IF there is no next event
    → session boundary
```

---

# 8. Interpret the Important Condition

The query uses:

```sql
TIMESTAMPDIFF(
    MINUTE,
    next_event,
    current_event
)
```

Because the events are processed in descending order, the `LEAD()` event represents the **previous chronological event**.

For example:

```text
Events:
03:00
02:30
01:30
```

With descending ordering:

```text
03:00 → LEAD = 02:30
02:30 → LEAD = 01:30
01:30 → LEAD = NULL
```

Therefore, the time difference can be used to identify session boundaries.

---

# 9. Final Calculation Logic

### Step 1 — Look at the neighboring event

Use:

```sql
LEAD(created_at)
OVER (
    PARTITION BY user_id
    ORDER BY created_at DESC
)
```

---

### Step 2 — Calculate the time difference

Compare the current event with the next chronological event.

---

### Step 3 — Mark session boundaries

Create:

```text
is_new_session = 1
```

when:

```text
gap > 60 minutes
```

or when there is no next event.

Otherwise:

```text
is_new_session = 0
```

---

### Step 4 — Generate session IDs

Use a cumulative `SUM()`:

```sql
SUM(is_new_session)
OVER (
    PARTITION BY user_id
    ORDER BY created_at
)
```

This converts the session-start flags into session numbers.

---

# 10. SQL Solution

```sql
WITH session_starts AS (
    SELECT
        created_at,
        user_id,
        event,
        CASE
            WHEN TIMESTAMPDIFF(
                MINUTE,
                LEAD(created_at) OVER (
                    PARTITION BY user_id
                    ORDER BY created_at DESC
                ),
                created_at
            ) > 60
            OR LEAD(created_at) OVER (
                PARTITION BY user_id
                ORDER BY created_at DESC
            ) IS NULL
            THEN 1
            ELSE 0
        END AS is_new_session
    FROM events
)

SELECT
    created_at,
    user_id,
    event,
    SUM(is_new_session) OVER (
        PARTITION BY user_id
        ORDER BY created_at
    ) AS session_id
FROM session_starts
ORDER BY user_id, created_at;
```

---

# 11. Understand `LEAD()`

This is the most important concept in the question.

Suppose User 1 has:

| created_at |
| ---------- |
| 00:01      |
| 00:30      |
| 01:01      |
| 02:31      |

We can think of the data as:

| Current Event | Next Event |    Gap |
| ------------- | ---------- | -----: |
| 00:01         | 00:30      | 29 min |
| 00:30         | 01:01      | 31 min |
| 01:01         | 02:31      | 90 min |
| 02:31         | NULL       |      — |

Therefore:

```text
00:01 → same session
00:30 → same session
01:01 → same session
02:31 → new session
```

---

# 12. Understand the Cumulative `SUM()`

Suppose we generate:

| Event | is_new_session |
| ----- | -------------: |
| 00:01 |              0 |
| 00:30 |              0 |
| 01:01 |              0 |
| 02:31 |              1 |

A cumulative sum gives:

| Event | Flag | Session ID |
| ----- | ---: | ---------: |
| 00:01 |    0 |          0 |
| 00:30 |    0 |          0 |
| 01:01 |    0 |          0 |
| 02:31 |    1 |          1 |

Depending on the exact session numbering convention, you may want the first session to be numbered **1 rather than 0**.

A cleaner implementation is therefore to explicitly mark the first chronological event as a session start.

---

# 13. Recommended Version

For a GitHub SQL portfolio, I would actually recommend this version because the logic is easier to explain and session IDs naturally start at **1**:

```sql
WITH event_gaps AS (
    SELECT
        user_id,
        created_at,
        event,
        LAG(created_at) OVER (
            PARTITION BY user_id
            ORDER BY created_at
        ) AS previous_event_time
    FROM events
),

session_flags AS (
    SELECT
        user_id,
        created_at,
        event,
        CASE
            WHEN previous_event_time IS NULL
                 OR TIMESTAMPDIFF(
                        MINUTE,
                        previous_event_time,
                        created_at
                    ) > 60
            THEN 1
            ELSE 0
        END AS new_session
    FROM event_gaps
)

SELECT
    created_at,
    user_id,
    event,
    SUM(new_session) OVER (
        PARTITION BY user_id
        ORDER BY created_at
    ) AS session_id
FROM session_flags
ORDER BY user_id, created_at;
```

### Why I prefer this version

It follows the logic directly:

```text
Current Event
      ↓
Previous Event
      ↓
Calculate Gap
      ↓
Gap > 60 min?
      ↓
Yes → New Session
No  → Same Session
      ↓
Cumulative SUM
      ↓
Session ID
```

`LAG()` is also more intuitive here because we're naturally asking:

> **"How long has it been since this user's previous event?"**

---

# 14. Example Walkthrough

Suppose:

| user_id | created_at |
| ------: | ---------- |
|       1 | 00:01      |
|       1 | 00:30      |
|       1 | 01:01      |
|       1 | 02:31      |

After `LAG()`:

| created_at | previous_event |    gap |
| ---------- | -------------- | -----: |
| 00:01      | NULL           |      — |
| 00:30      | 00:01          | 29 min |
| 01:01      | 00:30          | 31 min |
| 02:31      | 01:01          | 90 min |

Session flags:

| created_at | gap | new_session |
| ---------- | --: | ----------: |
| 00:01      |   — |           1 |
| 00:30      |  29 |           0 |
| 01:01      |  31 |           0 |
| 02:31      |  90 |           1 |

Cumulative sum:

| created_at | new_session | session_id |
| ---------- | ----------: | ---------: |
| 00:01      |           1 |          1 |
| 00:30      |           0 |          1 |
| 01:01      |           0 |          1 |
| 02:31      |           1 |          2 |

---

# 15. Key SQL Concepts Used

* `LAG()`
* Window functions
* `PARTITION BY`
* `ORDER BY`
* `TIMESTAMPDIFF()`
* `CASE WHEN`
* Cumulative `SUM()`
* Sessionization
* Time-series analysis

---

# 16. Edge Cases Considered

### First event for a user

There is no previous event.

```text
previous_event_time = NULL
```

Therefore, it starts **Session 1**.

### Gap exactly 60 minutes

The requirement says events **within 60 minutes** belong to the same session.

Therefore:

```text
60 minutes → same session
> 60 minutes → new session
```

### Gap greater than 60 minutes

Starts a new session.

### Multiple users

`PARTITION BY user_id` ensures each user gets independent session numbers.

---

# 17. Explanation

> "I used `LAG()` to find each user's previous event and calculated the time gap between the current and previous event. If the gap is greater than 60 minutes, or the event is the user's first event, I mark it as a new session. Finally, I use a cumulative `SUM()` of that flag partitioned by user to assign a session ID to every event."

---

# 18. Summary

The core **sessionization pattern** is:

```text
LAG()
  ↓
Find previous event
  ↓
Calculate time gap
  ↓
CASE WHEN gap > 60 minutes
  ↓
Create session-start flag
  ↓
Cumulative SUM()
  ↓
Session ID
```
