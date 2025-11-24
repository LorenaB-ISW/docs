# Playground User Guide

The Playground is Simba Intelligence's interactive interface for querying your data using natural language. Instead of writing SQL queries, you simply ask questions in plain English and receive instant answers with explanations, visualizations, and raw data.

## Overview

### What is the Playground?

Think of the Playground as having a conversation with an expert data analyst who has immediate access to all your databases. You ask questions like *"What were our top-selling products last quarter?"* and get back not just the data, but explanations of what the data means and how it was found.

### Key Capabilities

**🗣️ Natural Language Querying**
- Ask questions in plain English about your data
- No SQL knowledge required
- Context-aware follow-up questions

**⚡ Real-Time Results**
- Streaming responses as queries execute
- Live progress updates during processing
- Immediate feedback on query success or issues

**📊 Multi-Format Results**
- **Summary**: Natural language explanations of your data

**🎯 Smart Suggestions**
- AI-generated question suggestions based on your data
- Context-aware recommendations
- Learn what's possible with your data sources

---

## Prerequisites

Before using the Playground, ensure you have:

### 1. Data Sources Available
- At least one data source configured and accessible
- Created via [Data Agent](data-agent-user-guide.md) or manual configuration
- Proper permissions to query the data

### 2. AI Provider Configuration
- LLM provider configured (Google Vertex AI, OpenAI, AWS Bedrock)
- Sufficient API quota/credits for query processing
- Network connectivity to AI service endpoints

### 3. User Permissions
- Basic user access to Simba Intelligence (all users can access Playground)
- Query permissions for your target data sources
- Network access to the Playground interface

> **💡 Getting Started**: If you don't have data sources yet, visit the [Data Agent](data-agent-user-guide.md) to create one using AI assistance.

---

## Accessing the Playground

### Navigation
1. **Log into Simba Intelligence**
2. **Click "Playground"** in the main navigation menu
3. **Or visit directly**: `http://your-domain/playground`

### Initial Setup
When you first access the Playground:
- The system loads all available data sources you can query
- AI provider connectivity is verified
- Query suggestions are generated for available data

---

## Selecting Data Sources

### Data Source Dropdown

The Playground header contains a **data source selector** that shows:
- **All data sources** you have permission to query
- **Source descriptions** and metadata
- **Connection status** indicators

### Choosing the Right Data Source

**For specific datasets:**
- Select the data source that contains the information you need
- Each data source typically represents a specific business area or database

**For exploring:**
- Start with data sources that have rich suggestions
- Look for sources with clear, descriptive names
- Try sources created recently or specifically for your use case

### URL State Management

The Playground maintains your data source selection in the URL:
- `http://your-domain/playground?sourceId=12345`
- Bookmarkable URLs for specific data source contexts
- Shareable links with colleagues (permissions permitting)

---

## Writing Effective Natural Language Queries

### Query Input Interface

**Text Input Features:**
- **Auto-expanding text area** adapts to query length
- **Enter key submission** (Shift+Enter for new lines)
- **Query history** accessible via browser back/forward
- **Real-time validation** of query readiness

### Query Types and Examples

#### Basic Data Exploration
```
What data is available in this source?
Show me the first 10 rows of data
What are the column names and types?
How many records are in this dataset?
```

#### Analytical Questions
```
What are our top 5 best-selling products this year?
Show me sales trends by month for the last 6 months
Which regions have the highest customer satisfaction scores?
Compare revenue between Q3 and Q4 of this year
```

#### Comparative Analysis
```
How does this year's performance compare to last year?
What's the difference in sales between our top and bottom performing stores?
Which product categories grew the most year-over-year?
Compare customer acquisition costs across marketing channels
```

#### Filtered and Conditional Queries
```
Show me all customers with orders over $1000 in the last 30 days
What are the sales figures for the electronics category in California?
Find all orders that were delivered late in Q4
Show me employees hired after January 2023 with salaries above $75k
```

### Writing Tips for Better Results

**✅ Do:**
- Be specific about what you want to see
- Include time ranges when relevant ("last quarter", "this year")
- Mention specific values or thresholds ("over $1000", "top 10")
- Use business terminology familiar to your domain

**❌ Avoid:**
- Overly complex questions combining multiple unrelated analyses
- Ambiguous time references ("recently", "a while ago")
- Technical database terminology unless necessary
- Questions that require data not in the selected source

**🔍 Example Progression:**
1. **Start broad**: *"What sales data do we have?"*
2. **Get specific**: *"Show me monthly sales totals for 2024"*
3. **Dive deeper**: *"Which months had sales below the average?"*
4. **Analyze patterns**: *"What factors contributed to low sales in those months?"*

---

## Understanding Results

The Playground presents results in three distinct views, each optimized for different use cases:

### Summary Tab (Natural Language)

**What it contains:**
- AI-generated explanations of your results
- Key insights and patterns identified in the data
- Business context and interpretation
- Markdown-formatted text with emphasis on important findings

**When to use:**
- Quick understanding of what the data shows
- Sharing insights with non-technical stakeholders
- Getting AI interpretation of complex patterns
- Understanding business implications of the data

**Example output:**
```
Based on your sales data analysis, here are the key findings:

**Top Performing Products:**
- Product A led sales with $2.4M revenue (32% of total)
- Product B and C showed strong growth at 15% and 12% respectively

**Notable Trends:**
- Q4 showed 23% growth over Q3, likely due to holiday seasonality
- Western region outperformed others by 18%

**Recommendations:**
Consider increasing inventory for top performers and investigating 
factors behind Western region success.
```

### Data Tab (Structured Results)

**What it contains:**
- JSON formatted data output of the data used to answer the question
- Raw numerical results and calculated metrics

**When to use:**
- Detailed data and verification
- Validating AI interpretations against raw data

---

## Query Suggestions

### Smart Recommendations

The Playground automatically generates relevant question suggestions based on:
- **Data source content**: Fields, relationships, and data patterns
- **Common analysis patterns**: Typical business questions for your data type
- **Usage context**: Questions similar to what others ask about this data
- **AI insights**: Interesting patterns the AI identifies in your data

### Using Suggestions Effectively

**Suggestion interface:**
- Clickable question buttons below the query input
- Organized by relevance and complexity
- Updated dynamically based on your selected data source

**Suggestion types:**

**📊 Exploratory Questions:**
- "What data is available in this source?"
- "Show me a sample of the data"
- "What are the key metrics I can analyze?"

**📈 Analytical Questions:**
- "What are the trends over time?"
- "Which categories perform best?"
- "How does performance vary by region?"

**🔍 Specific Deep Dives:**
- Custom questions based on your specific data schema
- Business-relevant queries for your industry
- Complex analytical patterns found in your data

### Learning from Suggestions

Use suggestions to:
- **Discover capabilities**: Learn what questions are possible
- **Improve query writing**: See examples of effective natural language queries
- **Find patterns**: Identify analytical approaches you hadn't considered
- **Get unstuck**: Find inspiration when you're not sure what to ask

---

## Interactive Features

### Rating Query Results

**Purpose**: Help improve AI performance and share feedback

**Rating options:**
- **👍 Thumbs Up**: Query worked well and results were helpful
- **👎 Thumbs Down**: Results were incorrect, incomplete, or unhelpful

**When ratings help most:**
- Results don't match your expectations
- AI misinterpreted your question
- Query was too slow or failed
- Results were particularly insightful or useful

### Exporting Data

**Export capabilities:**
- **CSV download**: Raw data for use in Excel, analysis tools, or databases

### Follow-Up Questions

**Continuing analysis:**
- Each query maintains context for follow-up questions
- Reference previous results: *"Show me more details about the top product"*
- Drill down deeper: *"What are the monthly trends for that region?"*
- Compare and contrast: *"How does that compare to last year?"*

---

## Advanced Usage Patterns

### Complex Query Construction

**Breaking down complex questions:**

Instead of: *"Show me a comprehensive analysis of customer behavior patterns including demographics, purchase history, satisfaction scores, and predictive lifetime value across different segments for the past two years with seasonal adjustments"*

Try this approach:
1. *"What customer data do we have available?"*
2. *"Show me customer demographics breakdown"*
3. *"How do purchase patterns vary by demographic group?"*
4. *"What are the satisfaction scores for each customer segment?"*
5. *"How has customer behavior changed over the past two years?"*

### Working with Multiple Data Sources

**Source switching workflow:**
1. Start with one data source for core analysis
2. Note insights that might benefit from additional data
3. Switch to complementary data source
4. Cross-reference findings and build comprehensive picture

---

## Troubleshooting Common Issues

### "No Data Sources Available"

**Problem**: Dropdown shows no available data sources
**Solutions:**
1. Check that you have appropriate permissions
2. Verify data sources exist and are properly configured
3. Contact administrator if you should have access
4. Create new data sources using the [Data Agent](data-agent-user-guide.md)

### "LLM Configuration Required"

**Problem**: Queries fail due to missing AI provider setup
**Solutions:**
1. Navigate to LLM Configuration page
2. Configure at least one AI provider (requires admin access)
3. Contact administrator if you don't have configuration permissions
4. Wait for configuration to be completed by your team

### Query Execution Failures

**Common causes and solutions:**

**"Query timeout":**
- Simplify your question to focus on specific data
- Try breaking complex questions into smaller parts
- Contact administrator about database performance

**"No results found":**
- Verify your filters and conditions are appropriate
- Check date ranges and spelling of specific values
- Try broader queries to explore available data

**"Permission denied":**
- Confirm you have access to the selected data source
- Check with administrator about query permissions
- Verify data source is properly configured

### Slow Query Performance

**Optimization strategies:**
1. **Be specific**: Use date ranges and filters to limit data scope
2. **Focus queries**: Ask about specific metrics rather than "everything"
3. **Use appropriate data sources**: Choose sources optimized for your question type
4. **Time of day**: Consider database load during peak business hours

### Unexpected Results

**When results don't match expectations:**
1. **Verify data source**: Ensure you're querying the right dataset
2. **Clarify your question**: Rephrase with more specific terms
3. **Use follow-up questions**: Ask the AI to explain its interpretation
4. **Provide feedback**: Use rating buttons to improve future results

---

## Best Practices

### Query Writing Guidelines

**Start simple and iterate:**
```
Step 1: "Show me sales data"
Step 2: "Show me monthly sales totals for 2024"  
Step 3: "Which months had the highest sales?"
Step 4: "What products drove those high sales months?"
```

**Use business terminology:**
- Reference metrics as they're known in your organization
- Use familiar date ranges ("last quarter", "fiscal year 2024")
- Include relevant business context in questions

**Be explicit about what you want:**
- "Show me the top 10..." instead of "Show me the best..."
- "Sales figures for Q4" instead of "recent sales"
- "Revenue by product category" instead of "product performance"

### Effective Data Exploration

**Understanding your data:**
1. Start with exploratory questions to understand data structure
2. Identify key dimensions and metrics available
3. Explore data quality and completeness
4. Build complexity gradually based on initial findings

**Validating results:**
- Cross-reference AI summaries with raw data in Data tab
- Spot-check calculations and trends
- Use domain knowledge to validate insights
- Question results that seem too good (or bad) to be true

### Collaboration and Sharing

**Sharing insights:**
- Export data for use in presentations or reports
- Document successful query patterns for team reuse
- Share interesting findings via Summary tab explanations
- Use bookmarkable URLs to share specific data source contexts

**Building team knowledge:**
- Document query examples that work well for your data
- Share data source recommendations with colleagues
- Rate queries to improve AI performance for your team
- Collaborate on complex analysis by sharing intermediate results

---

*Ready to start exploring your data with natural language? Head to the [Playground](./playground) and ask your first question. Remember: there's no such thing as a bad question when you're learning about your data!*
