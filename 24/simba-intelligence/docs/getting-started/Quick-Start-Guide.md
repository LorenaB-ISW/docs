# Quick Start Guide (Production Focus) 

This guide is for enterprise teams standing up Simba Intelligence in a production (or production-like) environment. It intentionally avoids developer workstation / sample data instructions and instead walks you through the minimum sequence to become operational with your own infrastructure and data.

## Overview of the Flow

1. Deploy the platform
2. Configure an approved LLM / AI provider
3. Register at least one production data connector and create a data connection
4. Generate an initial governed data source using the Data Source Agent
5. Enable business users to explore via the Playground
6. Validate operational readiness & plan next hardening steps

---

## 1. Deploy the Platform

Deployment steps (container orchestration, Helm, configuration files, secrets, scaling parameters, ingress, TLS, etc.) are documented in the dedicated Installation Guide.

➡️ Follow: [Installation Guide](./Installation-Guide)

When complete you should have:

- Platform services healthy
- Administrative (supervisor) credentials provisioned
- Network access from the platform to required outbound AI endpoints and internal data sources

---

## 2. Configure an LLM / AI Provider

LLM configuration enables natural language understanding, semantic modeling, and embeddings. Use production credentials (not personal developer keys).

1. Sign in with a supervisor / administrator role.
2. Navigate to: `/llm-configuration` in the UI.
3. Select your provider (e.g., Vertex AI, Azure OpenAI, Bedrock).
4. Provide the required credentials (service account JSON, API key + endpoint, or similar) via the secure form.
5. Enable the ncessary Chat + Embeddings services and save.

Configuration Reference (detailed parameters & environment keys):

- Vertex AI: see [Configuration Reference – Google Vertex AI](../guides/admin-guides/LLM-Provider-Configuration#google-vertex-ai-configuration)
- Azure OpenAI: see [Configuration Reference – Azure OpenAI](../guides/admin-guides/LLM-Provider-Configuration#azure-openai-configuration)
- AWS Bedrock: see [Configuration Reference – AWS Bedrock](../guides/admin-guides/LLM-Provider-Configuration#aws-bedrock-configuration)

Use those sections for exact variable names, model identifiers, timeout and retry tuning options.

Security & Ops Recommendations:

- Store original secrets in your enterprise vault (e.g., HashiCorp Vault, AWS Secrets Manager); rotate per policy.
- Use least-privilege IAM role for service accounts.
- Monitor usage limits / quotas early to baseline cost.

(Optional) Multiple Providers: You can configure one primary provider at launch; add fallbacks later once baseline validation is complete.

---

## 3. Create a Data Connection

Connect an authoritative data source (e.g., Snowflake, PostgreSQL, SQL Server, BigQuery, etc.). Avoid pointing to non-governed sandbox datasets for your first production rollout.

1. Navigate to: `Data Connections` in the UI.
2. Click Create Connection.
3. Choose connector type.
4. Supply credentials (service user / managed identity preferred) + network parameters (host / account / warehouse / catalog / database / schema as applicable).
5. (Recommended) Scope access to read-only analytical schemas for the initial rollout.
6. Test Connection → Save.
7. Allow initial metadata discovery to complete (time depends on catalog size).

Guidance by common platforms (generic):

- Snowflake: Provide account locator / region, warehouse sized for concurrency testing, role with `USAGE` + `SELECT`.
- PostgreSQL / SQL Server: Prefer read replica endpoints to avoid production OLTP impact.
- BigQuery: Service account with dataset-level `roles/bigquery.dataViewer` and project `metadataViewer`.

More detail: [Data Connection Management Guide](../guides/user-guides/Data-Connection-Management)

---

## 4. Create an Initial AI-Generated Data Source

Use the Data Source Agent to translate business intent into a governed data source definition over your connected warehouse.

1. Navigate to: `Data Source Agent` (`/data-source-agent`).
2. Select the connection you just registered.
3. (Optional) Provide a clear objective (concise, business-oriented). Examples:

   ```
   Build a revenue performance dataset with metrics: total revenue, revenue by region, revenue by product category, year-over-year growth, top 10 customers by trailing 90 day revenue.
   ```

   ```
   Create a customer health dataset including: active subscriptions, churn flags, MRR, expansion vs contraction amounts, tenure buckets.
   ```
4. (Optional) List critical tables or schemas to prioritize (helps reduce exploration time).
5. Start the agent. It will inspect schema metadata, propose transformations / joins, and assemble the data source definition.
6. Review the generated fields; rename or hide any that should not be exposed. Approve to publish.

Operational Tips:

- Keep the first data source narrow; optimize clarity over breadth.
- Enforce naming conventions (e.g., `revenue_total`, `customer_churn_rate`).
- Capture data lineage output for audit (export if needed).

More detail: [Data Agent User Guide](../guides/user-guides/Data-Agent-User-Guide)

---

## 5. Explore with the Playground

The Playground allows natural language querying and iterative exploration over approved data sources.

1. Navigate to: `Playground` (`/playground`).
2. Select the published data source created above.
3. Ask domain-focused questions or use the suggested questions. Examples:
   - "What is total revenue this quarter vs the same quarter last year?"
   - "Show top 15 customers by trailing 90 day revenue and percent growth vs prior 90 days."
   - "Break down subscription churn by region and customer segment."
   - "List products with declining monthly revenue for the last 4 months."
   - "Identify anomalies in daily order volume over the past 30 days."
   - "Compare average customer lifetime value between enterprise and mid-market segments."

Recommendation: Establish a review loop with data owners for early queries to validate accuracy.

More detail: [Playground User Guide](../guides/user-guides/Playground-User-Guide)

---

## 6. Validate Operational Readiness

Confirm the platform is stable, secure, and delivering expected value before scaling access.

Readiness Checklist:

- ✅ LLM provider status healthy; usage metrics monitored
- ✅ At least one production data connection
- ✅ Initial curated data source published and reviewed
- ✅ Successful natural language queries returning accurate SQL and results
- ✅ Access controls / roles applied (admin vs analyst vs viewer)
- ✅ Logging & monitoring integrated (infrastructure + application + AI usage)
- ✅ Backup / recovery strategy documented (see Operations Guide)

Next Hardening Steps:

- Add additional connections (e.g., finance, customer success, product analytics) incrementally.
- Implement tagging / classification for sensitive fields (PII, financial, etc.).
- Configure performance monitoring & query cost dashboards.
- Set up automated model / provider health alerts.

---

## Troubleshooting (Production Context)

Common early issues:

**LLM Provider Authentication Fails**

- Verify service account / key has required model permissions.
- Confirm outbound firewall rules allow provider endpoints.

**Metadata Discovery Slow**

- Limit initial schema scope; exclude large archival schemas.
- Schedule off-peak discovery windows for very large catalogs.

**Unexpected Query Costs**

- Enable caching in each data source
- Add row-level / column filters to high-volume tables.

**Result Inconsistencies**

- Validate source table freshness; confirm no stale replicas.
- Check semantic field definitions for unintended aggregations.

For deeper diagnostics use: Operations Guide, Performance Monitoring guide, and central logging stack.

---

## Reference Links

- Installation: [Installation Guide](./Installation-Guide)
- Data Connection: [Data Connection Management](../guides/user-guides/Data-Connection-Management)
- Data Source Agent: [Data Agent User Guide](../guides/user-guides/Data-Agent-User-Guide)
- Playground: [Playground User Guide](../guides/user-guides/Playground-User-Guide)
- Administration: [Administrator Guide](../guides/admin-guides/Administrator-Guide)
- Operations: 
- Security & Auth: [Authentication & Security](../guides/admin-guides/Authentication-and-Security)

---

You're now ready to onboard additional teams. Keep the initial surface area small, validate quality, then scale connections, data sources, and user groups iteratively.