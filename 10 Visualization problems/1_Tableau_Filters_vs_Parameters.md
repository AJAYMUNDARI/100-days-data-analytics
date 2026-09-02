# Tableau Filters vs Parameters

## Business Context

**Scenario:**
Build an interactive Tableau dashboard for Fabletics' merchandising team to monitor **weekly sales performance across product categories and regions**.

The key requirement is to allow business users to explore the data while also providing flexibility for more advanced analysis.

---

## 1. Understand the Requirement

Before choosing between a filter and a parameter, ask:

> **Do we want to restrict the data being displayed, or do we want the user to control the dashboard's logic?**

This is the simplest way to distinguish the two.

---

## 2. Tableau Filter

### What is a Filter?

A **filter controls which data is included or displayed in the visualization**.

It answers:

> **"Which data do I want to see?"**

### Merchandising Examples

Filters would be appropriate for:

* Product Category → Activewear
* Region → West
* Week → Week 35
* Product → Leggings
* Store/Market → California

For example:

> If the merchandising team selects **Activewear + West Region**, the dashboard shows sales only for those selections.

### When I would use Filters

Use filters when the business user needs to:

* Focus on a specific region
* Analyze a particular product category
* Select a time period
* Drill down into a specific product
* Exclude irrelevant records

**Simple rule:**

> **Filter = Slice the data.**

---

## 3. Tableau Parameter

### What is a Parameter?

A **parameter is a user-controlled input that can be used in calculations, filters, or dashboard logic**.

It answers:

> **"How do I want the dashboard to behave?"**

Unlike a normal filter, a parameter does not inherently remove data from the visualization. It provides a value that can be used by calculated fields or other Tableau functionality.

### Merchandising Examples

#### Example 1 — Metric Selector

Create a parameter:

**Select Metric**

* Sales
* Profit
* Units Sold

The user can select **Sales**, and the visualization dynamically displays sales. They can then switch to **Profit** without creating separate charts.

#### Example 2 — Sales Target

Create a parameter:

**Sales Target = $100,000**

A calculated field can then identify whether a category has achieved the target:

```text
IF SUM([Sales]) >= [Sales Target]
THEN "Target Achieved"
ELSE "Below Target"
END
```

This could be used to highlight categories or regions that are meeting or missing their targets.

#### Example 3 — Top N Products

Create a parameter:

**Top N = 10**

The merchandising team can change it to:

* Top 5
* Top 10
* Top 20

and dynamically control how many products are displayed.

---

## 4. Filter vs Parameter

| Aspect                 | Filter                 | Parameter                          |
| ---------------------- | ---------------------- | ---------------------------------- |
| Primary purpose        | Restrict data          | Control logic                      |
| Main question          | "What data do I see?"  | "How should the dashboard behave?" |
| Example                | Select West region     | Select Sales vs Profit             |
| Removes/excludes data? | Yes                    | Not by itself                      |
| Used in calculations?  | Can influence results  | Yes, commonly                      |
| Typical use            | Category, region, date | Target, Top N, metric selector     |

---

## 5. How I Would Use Them on the Fabletics Dashboard

I would use **both**, because they solve different business needs.

### Filters

Use filters for:

**Region + Product Category + Week**

Example:

> Region = West
> Category = Activewear
> Week = Week 35

This allows the merchandising team to **slice the sales data**.

### Parameters

Use parameters for:

**Metric Selector + Sales Target + Top N**

Example:

> Metric = Sales
> Target = $100K
> Top N = 10

This allows the merchandising team to **change the analysis or dashboard behavior dynamically**.

---

## 6. Decision Framework

When deciding between the two, I would ask:

```text
Does the user want to select WHICH DATA to see?
                ↓
              FILTER

Does the user want to control HOW THE ANALYSIS behaves?
                ↓
            PARAMETER
```

---

## 7. Answer

> "The simplest way I differentiate them is that a filter controls **what data I want to see**, while a parameter controls **how I want the dashboard or calculation to behave**.
>
> For the Fabletics merchandising dashboard, I would use filters for things like region, product category, and week because the team needs to slice the sales data. I would use parameters for things like selecting between Sales and Profit, setting a sales target, or dynamically choosing Top N products.
>
> So, for example, if a merchandiser wants to see only Activewear sales in the West region, I would use filters. But if they want to change the dashboard from Sales to Profit or set a target threshold dynamically, I would use a parameter.
>
> In short, **filters are primarily for data selection, while parameters are for user-driven analytical logic and flexibility.**"
