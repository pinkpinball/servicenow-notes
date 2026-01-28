# ServiceNow Reports Fundamentals

This document provides an overview of **reporting and analytics** on the Now Platform.

Use it to understand what reports are, when to use them, and how they fit into ServiceNow.

---

## What is a Report?

- A report is a **visualization of data from a table or set of tables**
- Reports can show trends, summarize information, or list records

> Reports are how you turn data into insights.

---

## When Should I Use a Report?

Use reports when:

- You need to see data from tables quickly
- You want to analyze trends
- You want to share information with others

Do not use reports for:

- Data enforcement
- Real-time form validation
- Automating processes (use flows/scripts instead)

---

## Types of Reports

### List Reports

- Show rows of records
- Sortable and filterable

**Example:**

- All open incidents

---

### Pivot / Aggregation Reports

- Summarize data by field(s)
- Show totals, averages, counts

**Example:**

- Count of incidents by priority

---

### Bar / Column / Pie Charts

- Visual representation of grouped data

**Example:**

- Number of incidents per department

---

### Calendar / Timeline Reports

- Show data over time

**Example:**

- Planned maintenance schedule

---

### Performance Analytics (Advanced)

- Trend analysis and forecasting
- KPI dashboards

> Performance Analytics is optional and often used in ITSM metrics

---

## How Reports Work

1. **Select Table / Data Source**
2. **Define Conditions / Filters**
3. **Choose Type of Report**
4. **Configure Visualization**
5. **Save and Share**

> Reports can be personal, shared, or global

---

## Dashboards

- Dashboards are **collections of reports**
- Can include multiple report types and layouts
- Useful for executives, teams, or daily operations

**Example:**

- Incident dashboard showing open, prioritized, and SLA-breached incidents

---

## Report Best Practices

- Keep it simple: focus on actionable insights
- Use meaningful groupings and filters
- Avoid overloading dashboards with too many reports
- Name reports clearly
- Schedule reports if recurring review is needed

---

## Common Mistakes

- Using reports to enforce rules
- Making dashboards too cluttered
- Forgetting to filter out irrelevant data
- Ignoring performance implications of large datasets

---

## 1-Min Example

**Requirement:**
See how many high-priority incidents were created last month per department

**Solution:**

1. Table: Incident
2. Conditions: priority = High, created last month
3. Group by: department
4. Visualization: Bar Chart
5. Save report and add to team dashboard

---

## Relationship to Other Tools

- **Forms/Lists:** source of raw data
- **Flows:** can create or update records, which are then reported
- **Scripts:** can prepare or calculate fields for reporting

---

## Mental Shortcut

> **Reports = data insight**
> **Dashboards = data at a glance**
> \*\*Scripts/Flows = data manipulation
