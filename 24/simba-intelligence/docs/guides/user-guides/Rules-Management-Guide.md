# Rules Management Guide

Rules let you shape how Simba Intelligence's AI agents behave—both the **Data Source Agent** (when creating new data sources) and the **Query Agent** (when interpreting and answering user questions). A rule is a short, plain‑English instruction that adds governance, tone, context, constraints, or business logic beyond what is inferred from schemas and data source configuration.

---

## Overview

### What Are Rules?
Think of rules as lightweight guardrails or behavioral hints you layer on top of your existing semantic model. They do **not** replace data source optimization (see the Data Source Optimization Guide), but they help:

- Emphasize important business definitions
- Enforce preferred terminology
- Prevent disallowed insights or perspectives
- Guide tone, style, or emphasis for generated explanations
- Remind the AI of contextual constraints (“Never expose raw cost fields unless explicitly asked”)  

### Rule Targets
A rule can apply to:
- **None (Global for Query)** – Applies to *all* user queries across every data source unless overridden by explicit user instruction
- **Specific Data Source** – Applies only when querying that data source
- **Data Source Agent** – Applies when the AI is *creating* new data sources via the Data Agent (useful for structural / naming / compliance guidance)

> 💡 If you choose **None**, the rule does *not* constrain creation logic directly—it's interpreted during natural language querying across any data source. To influence creation, target the Data Source Agent explicitly or define both types of rules.

### When to Use Rules
Use Rules when you need:
- Consistent phrasing (e.g., “Use ‘Customer’ instead of ‘Client’ in explanations”)
- Guardrails (“Do not include speculative forecasting unless user asks for a prediction”)
- Compliance reinforcement (“Do not surface personally identifiable information (PII) in summaries”)
- Analytical posture (“Prioritize variance explanations over raw totals when comparing periods”)
- Data Source creation standards (“Prefer concise, business-readable field labels; expand abbreviations”)  

---

## Key Concepts

| Attribute | Description |
|-----------|-------------|
| Name | Human-readable identifier shown in the Rules list |
| Content | The actual instruction (plain English) – required |
| Data Source | Optional selection; empty = global (query) scope; specific = scoped; agent = creation context (if UI supports explicit agent targeting) |
| Created At | Timestamp (UTC) when the rule was created |

> 🧩 The UI currently exposes: **Name**, **Content**, and **Data Source selector** with a `None` option. Selecting none yields a global query rule. Selecting a data source scopes it. (If/when an explicit “Data Source Agent” target option is added, include one rule per structural policy.)

### How the AI Uses Rule Names
The **Rule Name is not just cosmetic**. The AI performs lightweight semantic relevance matching using rule names as signals to decide which optional rules to load for a given task context (especially when many global rules exist). A weak or vague name can cause the system to overlook an otherwise useful rule.

**Implications:**
- Names should summarize *intent + domain* ("Privacy: Suppress PII" vs "PII Stuff")
- Avoid generic labels ("General", "Style", "Rule 1")
- Prefer consistent prefixes so related rules cluster during relevance scoring
- If two rules have overlapping names, disambiguate with scope or outcome wording (e.g., "Terminology: Gross vs Net" vs "Terminology: Acronym Expansion")

**Name Structure Template:**
`CATEGORY: Focus – Qualifier`

**Examples:**
- `PRIVACY: Suppress PII Fields`
- `TERMINOLOGY: Use “Gross Revenue” Label`
- `TONE: Executive Summary First`
- `STRUCTURE (Creation): Expand Abbreviations`
- `COMPARISON: Highlight Drivers Before Totals`

**Anti‑Patterns:**
- `Revenue` (too vague)
- `Tone Rule` (uninformative)
- `DSAgent` (unclear function)
- `AOV` (acronym with no context)

> 🎯 Goal: A reviewer (or the AI) should infer the rule’s purpose without reading its content.

---

## Accessing Rules Management

1. Navigate to **Rules** in the application navigation
2. The Rules list displays existing rules (Name, Data Source, Created At)
3. Use the **Search** box to filter by rule name
4. Click **Create Rule** to add a new one

---

## Creating a Rule

### Steps
1. Click **Create Rule**
2. Enter a **Rule Name** (concise, descriptive)
3. (Optional) Choose a **Data Source** from the dropdown:
   - Leave blank = global query rule
   - Select a data source = scoped rule for that source only
4. Enter **Rule Content** – write a direct instruction (avoid fluff)
5. Click **Create Rule**

### Content Writing Guidelines
**✅ Do:**
- Be explicit: “Always describe currency values in USD unless user specifies another currency.”
- Use imperative tone: “Prioritize…” “Avoid…” “Never…” “When comparing…”
- Keep one intent per rule; create multiple rules instead of a long paragraph
- Reference business terminology exactly as labeled in the data source

**❌ Avoid:**
- Vague phrasing: “Be smart about revenue stuff.”
- Overloaded instructions mixing tone, filtering, formatting in one rule
- Duplicating what the data source schema already makes obvious

### Example Rules
| Name | Scope | Content |
|------|-------|---------|
| Revenue Terminology | Global | Use the term “Gross Revenue” instead of generic “Revenue” when summarizing totals. |
| Cost Visibility | Global | Do not include cost or margin fields unless the user explicitly asks for cost, margin, or profitability. |
| Sales Source Tone | Sales Performance Source | Summaries should emphasize drivers of change (top 3 product categories and regions) before totals. |
| Naming Standards | Data Source Agent | When creating data sources, expand abbreviations in labels (e.g., `rev_gross_amt` → `Gross Revenue`). |
| Currency Clarification | Sales Performance Source | If user asks for revenue without specifying currency, clarify that figures are in USD. |

---

## Editing a Rule

1. In the Rules table, click the **Edit** (pencil) icon
2. Adjust **Name**, **Content**, or **Data Source**
3. Click **Save Changes**
4. Confirm success via toast message

> ✏️ Editing scope (changing from global to specific or vice versa) changes where the rule applies immediately.

---

## Deleting a Rule

1. Click the **Delete** (trash) icon next to a rule
2. Confirm in the deletion modal
3. Rule is permanently removed; behavior reverts accordingly

> 🛑 This cannot be undone—capture critical governance rules elsewhere (internal documentation / version control).

---

## Searching and Filtering

Use the search input at the top of the page to filter by **Rule Name**. (Search currently does not inspect rule content.) For large catalogs of rules, adopt consistent naming prefixes (e.g., `GOV:`, `TONE:`, `CALC:`).

**Naming Pattern Suggestions:**
- `GOV: Data Privacy`
- `FORMAT: Currency`
- `TONE: Executive Summaries`
- `MODEL: Abbreviation Expansion`

---

## Best Practices

### Keep Rules Atomic
One rule = one behavioral constraint. Easier to maintain, disable, or refine.

### Use Hierarchical Intent
- Rely on **data source configuration** for structural clarity
- Use **rules** for behavioral overlays (tone, selective suppression, explanation priorities)

### Avoid Rule Overlap
Too many similar or conflicting rules can cause diluted adherence.

### Optimize Names for AI Selection
Because the AI scores rule names for contextual relevance:
- **Front‑load the category** so related rules cluster (`PRIVACY:`, `TONE:`, `FORMAT:`)
- **Use strong action verbs** ("Suppress", "Highlight", "Use", "Avoid", "Explain")
- **Disambiguate variants** (e.g., `TERMINOLOGY: Margin vs Markup` distinct from `TERMINOLOGY: Acronym Expansion`)
- **Flag creation-specific rules** with a suffix or parenthetical `(Creation)` or `(Data Source Agent)` until dedicated targeting UI exists
- **Refactor legacy names** that are overly short—rename rather than layering new rules on top

**Before / After Examples:**
| Before | After |
|--------|-------|
| Cost Rule | PRIVACY: Suppress Cost & Margin Unless Asked |
| Naming Standards | STRUCTURE (Creation): Expand Abbreviations in Labels |
| AOV | TERMINOLOGY: Use “Average Order Value” (Not AOV) |
| Style | TONE: Executive Summary Before Detail |

**Removal Checklist for Weak Names:**
Delete or rename if the name is: single-word generic, ambiguous acronym, or duplicates another rule with only minor differences.

### Review Regularly
Quarterly or after major schema/label changes, audit rules for redundancy or obsolescence.

### Traceability
Maintain a simple changelog externally (date, rule added/retired, rationale). This supports governance and auditability.

---

## Common Use Case Patterns

| Scenario | Example Rule |
|----------|--------------|
| Suppress sensitive fields | Do not surface user email addresses in summaries unless explicitly asked. |
| Force explanation structure | Begin summaries with a variance explanation before totals if a comparison is implied. |
| Enforce naming conventions | Replace internal acronym “AOV” with “Average Order Value” in all user-facing wording. |
| Clarify assumptions | If timeframe not specified, clarify the default time range before answering. |
| Guide creation style | When creating data sources, prefer descriptive business names over system codes. |
| Limit speculation | Do not generate forward-looking forecasts unless user explicitly asks for predictions. |

---

## Troubleshooting

| Issue | Possible Cause | Resolution |
|-------|----------------|-----------|
| Rule appears ignored | Too vague; conflicts with stronger schema signals | Rewrite using specific verbs and domain terms |
| Conflicting behaviors | Multiple rules say opposite things | Consolidate; keep the clearer one |
| Hard to locate rule | Inconsistent naming | Adopt prefixes / naming taxonomy |
| Overly verbose summaries | Tone or structure rules encourage narration | Tighten rules to focus on insight hierarchy |
| Sensitive data surfaced | No rule suppressing it; field visible | Hide field OR add suppression rule |

---

## Governance Recommendations

1. **Baseline Set**: Create a minimal core pack (Privacy, Tone, Terminology, Structural Guidance).
2. **Expansion Policy**: Require justification for each new rule (what problem does it solve?).
3. **Sunset Review**: Mark candidate rules for removal if unused or superseded.
4. **Change Communication**: Notify analysts when global rules change—they may observe shifts in summarization style.

---

## FAQ

**Q: Do rules change underlying query logic?**  
A: They influence interpretation and summarization, not raw database access—underlying permissions and schema still govern data availability.

**Q: Can a scoped rule override a global rule?**  
A: Yes—scoped rules provide additional, more specific context when that data source is queried.

**Q: How many rules is too many?**  
A: If you need more than ~25 active rules, consider consolidation or moving structural guidance into data source configuration.

**Q: Should I duplicate a global rule inside every data source?**  
A: No. Use a single global rule unless a source needs a tailored variation.

**Q: What if users explicitly request something a rule discourages?**  
A: Well‑written rules typically allow explicit user intent to override (e.g., “unless the user explicitly asks”). Phrase accordingly.

---

## Summary & Next Steps

Rules complement (not replace) strong data modeling. Start with a focused set that enforces privacy, terminology, and analytical framing. Expand carefully, review regularly, and retire obsolete rules.

**Suggested follow-up actions:**
- Draft a core governance ruleset (4–6 rules)
- Audit for redundant or unclear existing rules
- Add a naming taxonomy prefix if absent
- Pair this with the Data Source Optimization checklist for holistic quality control

*Ready to add or refine rules? Open the Rules interface and apply these patterns—each clear rule compounds overall AI reliability and user trust.*
