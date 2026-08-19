![Medium](https://img.shields.io/badge/Difficulty-Medium-yellow?style=for-the-badge)
# 🗺️ Google Maps — Product Sense

**Category:** Product Improvement / Product Sense

**Framework:** **Segment → Use Case → Pain Point → Feature → Prioritize (Impact × Effort)**

---

## ❓ Question

Let's say that you're the PM on Google Maps.

1. How would you improve Google Maps?
2. What metrics would you check to see if your feature improvements are successful?

---

# 💡 Solution

## Step 1: Segment — Who Are We Improving It For?

Google Maps serves multiple user segments:

* **Consumers** → Use Maps to navigate, discover places, and plan trips.
* **Businesses** → Use Maps to get discovered and manage their listings.
* **Developers** → Use Google Maps APIs to build applications and services.

### 🎯 Target Segment

I would focus on **consumers**, since they represent the largest user base and have the broadest interaction with Google Maps.

Within consumers, I would focus on two major use cases:

1. **Routing** — Getting from Point A to Point B.
2. **Exploration** — Finding restaurants, stores, services, attractions, etc.

---

# Step 2: Use Case → Pain Points

## 🚗 Use Case 1: Routing

### Pain Point 1 — ETA can be inaccurate

Google Maps may show an ETA based on available traffic data, but unexpected events such as construction, accidents, or road closures can cause significant delays.

**User problem:**

> "Google said 25 minutes, but the actual trip took 40 minutes."

---

### Pain Point 2 — Poor route understanding in unfamiliar areas

Users may not understand the type or difficulty of a route before starting navigation.

For example:

* Mountain roads
* Narrow roads
* Steep elevation
* Poor road conditions
* Limited road access

**User problem:**

> "The route is technically shorter, but I didn't know it would take me through a difficult mountain road."

---

### Pain Point 3 — Limited proactive road information

Google Maps generally responds to incidents such as accidents or traffic after they occur.

A user could benefit from knowing important **route characteristics before starting the trip**.

---

# 🍽️ Use Case 2: Exploration

## Pain Point 1 — Historical busy hours don't show current wait times

Google Maps can show popular/busy hours, but historical patterns don't necessarily tell users how long they will wait **right now**.

**User problem:**

> "The restaurant looks fine according to the historical data, but there is actually a 40-minute wait."

---

## Pain Point 2 — Parking uncertainty

A user may:

1. Search for a restaurant.
2. Select the restaurant.
3. Start navigation.
4. Reach the destination.
5. Then realize parking is difficult.

This creates unnecessary friction at the end of the journey.

---

# Step 3: Feature

Each feature should directly solve a previously identified pain point.

---

## 🅿️ Feature 1 — Live Parking Availability

### Problem

Users often know where they want to go but don't know where they can park.

### Solution

Show parking availability near the destination directly inside Google Maps.

For example:

> **3 parking garages nearby**
> 🟢 Garage A — Available
> 🟡 Garage B — Limited availability
> 🔴 Garage C — Full

### Possible Data Sources

* Parking garage operators
* Municipal parking systems
* IoT/sensor data
* Crowdsourced signals

### MVP

Start with **parking garages**, where occupancy data is easier to obtain.

Street parking can be introduced later.

---

# 🍽️ Feature 2 — Live Restaurant Wait Times

### Problem

Historical "busy hours" don't tell users how long they will actually wait.

### Solution

Provide an estimated current wait time.

Example:

> **Current wait: ~20 minutes**

### Possible Data Sources

* Restaurant partners
* Reservation platforms
* Aggregated location signals
* Historical foot-traffic models
* Restaurant-provided data

The initial version could leverage Google's existing location and foot-traffic infrastructure rather than creating an entirely new system.

---

# 🛣️ Feature 3 — Route Condition & Difficulty Information

### Problem

Users may select a route without understanding its characteristics.

### Solution

Show route-level information before navigation.

Example:

> **Mountain Route**
> ⛰️ High elevation
> 🛣️ Narrow roads
> ⚠️ Limited lighting
> 📈 Steep sections

This would be particularly useful when traveling through unfamiliar areas.

---

# Step 4: Prioritize — Impact × Effort

A feature shouldn't be prioritized only because it helps many users.

We need to consider both:

* **Impact** → How many users benefit and how painful is the problem?
* **Effort** → Data, partnerships, engineering complexity, and operational requirements.

| Feature                       | Impact      | Effort | Priority |
| ----------------------------- | ----------- | ------ | -------- |
| 🅿️ Live Parking Availability | High        | Medium | **P1**   |
| 🍽️ Live Wait Times           | High        | Medium | **P2**   |
| 🛣️ Route Conditions          | Medium–High | High   | **P3**   |

### 🥇 P1 — Live Parking Availability

**Why?**

* Clear user pain point.
* Directly connected to the navigation journey.
* Can start with parking garages.
* Easier to create an MVP before expanding to street parking.
* Strong opportunity to reduce post-navigation friction.

---

# Step 5: Success Metrics

The success of the feature should be measured using a combination of **North Star / primary metrics, supporting metrics, and guardrail metrics**.

## 🅿️ Live Parking Availability — P1

### Primary Metric

**Post-navigation parking search rate**

> % of navigation sessions where users subsequently search for parking near their destination.

**Expected outcome:** ↓

If users no longer need to perform a separate parking search, the feature is reducing friction.

### Supporting Metrics

* Parking availability card CTR
* % of users selecting a recommended parking location
* Navigation → parking conversion rate
* Time from destination arrival → parking selection
* Parking-related searches per navigation session

### Guardrail Metrics

* Navigation completion rate
* User complaints
* Incorrect parking availability reports
* Overall Maps session retention

---

# 🍽️ Live Wait Times — P2

### Primary Metric

**Restaurant detail → visit/conversion rate**

Expected outcome: **↑**

If users have better information about the wait time, more users should confidently proceed with their visit.

### Supporting Metrics

* Wait-time card CTR
* Restaurant direction requests
* Calls/reservations
* Restaurant detail → navigation conversion
* Search abandonment rate

### Guardrail Metrics

* Accuracy of estimated wait time
* User complaints
* Restaurant partner satisfaction

---

# 🛣️ Route Conditions — P3

### Primary Metric

**Route abandonment / rerouting rate**

Expected outcome: **↓**

If users understand the route before starting, fewer users should unexpectedly abandon or repeatedly reroute.

### Supporting Metrics

* Route condition card engagement
* Route selection rate
* Alternative route selection
* Rerouting frequency
* Navigation completion rate

### Guardrail Metrics

* ETA accuracy
* Navigation errors
* User complaints
* Route completion time

---

# 📊 Overall Metrics Framework

| Metric Type    | Example                                      |
| -------------- | -------------------------------------------- |
| **Primary**    | Feature-specific behavior/conversion         |
| **Engagement** | CTR, feature usage, interaction rate         |
| **Outcome**    | Reduced friction, increased conversion       |
| **Quality**    | Accuracy of predictions/information          |
| **Guardrail**  | Retention, complaints, navigation completion |

---

# 🎯 Final Recommendation

I would prioritize **Live Parking Availability** as the first feature.

The reasoning is:

**Clear pain point → measurable demand → feasible MVP → direct integration with navigation → measurable user impact**

I would initially launch it for **parking garages**, measure adoption and reduction in post-navigation parking searches, and then expand toward street-parking availability if the MVP demonstrates strong user value.

---

# 🧠 Framework to Remember

For product improvement questions, use:

> **Segment → Use Case → Pain Point → Feature → Prioritize (Impact × Effort) → Metrics**

### Two rules:

**1. Never propose a feature before identifying the pain point it solves.**

**2. Never prioritize using impact alone — always consider effort.**

This framework can be applied to almost any product-sense interview question.
