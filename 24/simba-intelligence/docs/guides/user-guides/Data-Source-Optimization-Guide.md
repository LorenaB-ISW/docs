# Data Source Optimization Guide

Designing high‑quality data sources is the single most important factor in achieving fast, accurate natural language query results in Simba Intelligence. This guide provides practical, field‑tested recommendations for structuring, configuring, and tuning data sources so the Playground can interpret user questions reliably and return results quickly.

---

## Overview

### Why Optimization Matters
A well‑optimized data source:
- **Improves answer quality** – Clear, descriptive field labels help the AI map user language to the correct data elements
- **Reduces query latency** – Lean schemas and proper caching minimize database work
- **Prevents ambiguity** – Curated fields and derived metrics remove guesswork
- **Enhances usability** – Business‑friendly naming and formatting increase adoption

### Core Principles
1. Model only what users actually need
2. Make business intent obvious through labels & descriptions
3. Reduce noise (hide non-analytical / technical fields)
4. Precompute or define reusable logic as derived fields or custom metrics
5. Constrain default scope (time + filters) for performance; let users override explicitly
6. Cache wisely to avoid repetitive expensive queries

---

## Quick Start Checklist
Use this fast audit after creating or importing a new data source:

| Area | Question | Goal |
|------|----------|------|
| Scope | Are only required tables joined? | Minimal, purposeful joins |
| Fields | Are unused / opaque columns hidden? | Reduce noise |
| Labels | Would a business user recognize every visible field name? | Natural language alignment |
| Formatting | Are numbers, currencies, dates properly formatted? | Readability & correct interpretation |
| Derived Logic | Are common calculations exposed as metrics? | Reuse & consistency |
| Caching | Is caching enabled with sensible TTL? | Performance |
| Time Range | Is default time window narrow enough? | Faster queries |
| Global Filters | Are baseline quality / security filters applied? | Data relevance & governance |

---

## 1. Scope the Data Source Carefully

The data source configuration has the largest impact on both **query quality** and **performance**. Avoid the temptation to expose an entire schema.

### Best Practices
- **Start from the user questions**, not from tables. Identify entities (e.g., Orders, Customers, Products) that directly answer expected questions.
- **Join only required tables**. Every extra join increases potential ambiguity and execution cost.
- **Remove exploratory / staging tables** – If users *might* ask about them later, add them in a controlled iteration.
- **Flatten where appropriate** – Pre-joining small, static lookup tables (e.g., country codes) can simplify queries.

### Anti‑Patterns
- Broad star schemas with dozens of dimension joins “just in case”
- Including large fact tables that do not serve a defined analytical purpose yet
- Exposing system or operational tables (audit logs, metadata tables) in general-purpose sources

> 💡 Tip: If a data source begins to feel “catch‑all”, split it into focused thematic sources (e.g., `Sales Performance`, `Customer Lifecycle`).

---

## 2. Curate Relationships and Joins

Only create joins that support real analytical pathways.

### Guidelines
- Use **clear, single-purpose joins** – Avoid multi-hop relationship chains if a simpler materialized view can replace them.
- Ensure **key cardinalities** reflect actual data (1:Many, Many:1) to help semantic interpretation.
- Avoid circular or redundant paths between entities.
- Verify that join conditions are selective and indexed in the underlying database.

### Validation Checklist
- Do row counts explode unexpectedly after joins?
- Do joins introduce duplicate metric inflation? (If so, consider aggregation or de-duplication logic.)
- Are there nullable foreign keys producing sparse relationships? Hide unused links.

---

## 3. Use the Fields Tab Strategically

The Fields tab is where you shape the *semantic surface* the AI can work with.

### Visibility Management
- **Hide non-analytical fields**: internal IDs, hash keys, technical timestamps, raw JSON blobs
- **Show friendly identifiers**: e.g., `Order Number`, `Customer Name`
- **Minimize similar synonyms**: having `amount`, `total_amount`, and `gross_amount` visible confuses intent—pick one and clarify with label + description

### Descriptive Labels
The "Label" property should reflect how users *ask* for the concept.

| Raw Column | Poor Label | Good Label |
|------------|------------|------------|
| `name_2` | Secondary Name | Country Name |
| `cust_id` | CustID | Customer ID |
| `rev_gross_amt` | RevAmt | Gross Revenue |
| `dt_trx` | trx_dt | Transaction Date |

> ✅ Aim for **Title Case**, business wording, and explicit meaning.

### Field Descriptions (if available)
Add short clarifications where ambiguity exists (e.g., "Includes cancelled orders" or "Net of discounts"). These reinforce semantic disambiguation.

### Formatting
Proper formatting helps both humans and AI interpretation:
- **Numbers**: Set decimal precision appropriate to scale (e.g., 2 decimals for currency)
- **Percentages**: Store as numeric but label with % expectation (e.g., `Conversion Rate`)
- **Dates**: Ensure consistent timezone handling; label if date is snapshot vs event date
- **Currency fields**: If multi-currency, expose a `Currency Code` dimension and clarify conversion basis

---

## 4. Add Derived Fields & Custom Metrics

If a calculation represents a reusable business concept, **define it explicitly** rather than expecting users to compose it via natural language each time.

### When to Create a Derived Field / Metric
- Calculation appears in dashboards or recurring conversations
- Users naturally ask for it ("average order value", "churn rate")
- Logic requires multiple base fields or conditional handling
- You want consistent governance (e.g., margin = (Revenue - Cost) / Revenue)

### Examples
| Concept | Type | Definition (Example) | Benefit |
|---------|------|----------------------|---------|
| Average Order Value | Metric | `SUM(order_amount) / COUNT(DISTINCT order_id)` | Consistency |
| Active Customers | Derived Field | Customers with order in last 90 days | Clear cohort logic |
| Net Revenue | Metric | `gross_revenue - discounts - refunds` | Alignment |
| Customer Tenure Days | Derived Field | `DATEDIFF(day, signup_date, CURRENT_DATE)` | Lifecycle analysis |

> 🔧 Treat these like a semantic contract—changing them later impacts interpretations; version consciously.

---

## 5. Configure Caching

Caching reduces repetitive load and response time for frequent or similar questions.

### Recommendations
- **Enable caching** in the Cache tab for read-heavy analytical sources

### When Not to Cache
- Highly volatile transactional sources requiring up-to-the-second accuracy
- Data sources used primarily for one-off, ad-hoc exploration with low repeat patterns

---

## 6. Control Default Time Scope

In the Global Settings → Time Bar Settings, the **Default Time Range** constrains queries automatically unless a user explicitly specifies a different period.

### Defaults
Data sources have different default time ranges depending on how they are created:
- **Manually created data sources**: Default to a **1-week** time range
- **Agent-created data sources**: Default to a **1-year** time range

Pay close attention to these defaults and identify which field is being used for time filtering. Modify the time range as needed to ensure the data source contains an appropriate range of data for answering expected questions.

### Best Practices
- Choose a **recent window** that reflects typical analytical focus (e.g., last 30, 60, or 90 days)
- Avoid unbounded defaults ("All Time")—they increase costs and latency
- Align with business reporting cadence (monthly, quarterly)

### Override Behavior
If a user asks: *"What were sales in Q1 2024?"* – their explicit time frame overrides the default. Otherwise the default silently applies, ensuring performance.

> 🕒 Tip: Periodically audit whether the default window still aligns with usage patterns.

---

## 7. Apply Global Filters Thoughtfully

Global Filters (in Global Settings) let you enforce baseline data constraints across all queries.

### Common Use Cases
- **Quality filtering**: `revenue > 0`, `status = 'Completed'`
- **Soft security**: Exclude internal/test accounts (e.g., `email NOT LIKE '%@example-internal%'`)
- **Regulatory segmentation**: Limit to allowed regions by default

### Guidelines
- Keep them **transparent**—document in data source description so users understand default context
- Avoid over-restricting; filters should narrow noise, not hide critical data unexpectedly
- Use stable predicates (no volatile expressions that cause cache churn)

### Override Model
If a user explicitly requests records outside the filter scope (*"Include cancelled orders"*), the AI can adjust—otherwise defaults apply.

---

## 8. Performance Validation Routine

After initial configuration, run a lightweight QA pass:

| Step | Action | Expected Outcome |
|------|--------|------------------|
| 1 | Issue broad summary question ("What data is available?") | Fast field enumeration |
| 2 | Ask for core metric aggregation ("Total revenue last 30 days") | < 5–10s response |
| 3 | Ask drill-down ("Revenue by product category last 30 days") | Stable latency |
| 4 | Ask historical comparison ("Compare revenue last 30 vs prior 30 days") | Correct time logic |
| 5 | Request derived metric ("Average order value last 30 days") | Uses defined metric |

If anomalies appear (slow steps, unexpected duplications), revisit joins, hidden fields, or caching settings.

---

## 9. Governance & Change Management

Treat optimized data sources as governed semantic assets.

### Change Guidelines
- **Document adjustments**: Keep a short changelog (date, change, rationale)
- **Version impactful metrics**: Introduce `Gross Margin v2` rather than silently altering logic
- **Review monthly**: Retire unused derived metrics / fields
- **Monitor usage**: Identify high-traffic queries to justify further optimization

### Collaboration Pattern
1. Draft / prototype in a sandbox data source
2. Validate with sample user questions
3. Promote to shared environment
4. Communicate additions to derived metrics & filters

---

## 10. Example End-to-End Tuning Scenario

**Initial State:** Sales data source includes 18 tables, 240 fields. Users report slow queries and inconsistent “average order value” answers.

**Optimization Actions:**
1. Removed 7 unused staging tables; limited to 6 core fact/dimension tables
2. Hid 96 technical columns (IDs, hash keys, system flags)
3. Standardized labels (e.g., `rev_gross_amt` → `Gross Revenue`)
4. Added derived metrics: `Average Order Value`, `Net Revenue`, `Order Count`
5. Enabled 15‑minute cache TTL
6. Set default time range to last 60 days
7. Added global filter `order_status = 'Completed'`

**Result:**
- Median query latency reduced from 28s → 9s
- User follow-up clarification rate dropped 35%
- Adoption increased (40% more distinct querying users week over week)

---

## Frequently Asked Questions

**Q: Should I create one massive “enterprise” data source?**  
A: No. Create focused, purpose-built sources aligned to business domains. This improves clarity and performance.

**Q: Users ask for a calculation that changes monthly—should it be a derived metric?**  
A: Only if the logic is stable. Otherwise keep it ad-hoc until finalized.

**Q: How many derived metrics is too many?**  
A: If users struggle to choose between similar metrics (e.g., 4 revenue variants), consolidate. Aim for clarity over exhaustiveness.

**Q: What if performance is still slow after optimization?**  
A: Investigate database indexing, materialized views, and confirm queries benefit from caching. Consider splitting very large fact domains.

---

## Summary & Next Steps

An optimized data source narrows ambiguity, accelerates answers, and encodes shared business logic so natural language queries behave predictably. Start by auditing visibility and labels, then layer in derived metrics, caching, and scoped defaults.

**Suggested follow-up actions:**
- Review existing high-traffic sources against this checklist
- Add missing derived metrics for top 5 repeated calculations
- Enable/adjust caching where latency > 15s
- Document current global filters and default time ranges

*Ready to refine your existing data sources? Open a data source configuration now and apply the checklist above—small changes compound into major user-facing improvements.*
