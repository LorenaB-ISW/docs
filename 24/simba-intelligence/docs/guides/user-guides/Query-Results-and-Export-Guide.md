# Query Results and Export Guide

This guide explains how to work with query results in Simba Intelligence, including understanding different output formats, using interactive features, and exporting data for further analysis.

## Overview

### Understanding Query Results

When you ask questions in Simba Intelligence, the system provides comprehensive results in multiple formats to meet different analytical needs. Think of it like asking a question to a data analyst who gives you:

- **A verbal explanation** of what they found (Summary)
- **The actual data** that was used for the summary
- **The data source** used to get the data

### Result Processing Flow

```
Your Question → AI Processing → Database Query → Result Assembly → Multi-Format Output
     ↓              ↓              ↓              ↓              ↓
Natural Language → Query → Query Engine → AI Analysis → Summary + Data
```

**Real-time streaming:** Results appear progressively as they're processed, so you see progress updates and can start reviewing answers before processing is complete.

---

## Result Format Types

Simba Intelligence presents every query result in three distinct formats, each optimized for different use cases:

### Summary Tab: Natural Language Insights

**What it contains:**
- AI-generated explanations in plain English
- Key findings and patterns identified in your data
- Business context and interpretation
- Markdown formatting with emphasis on important points

**When to use:**
- **Executive presentations**: Share insights without technical details
- **Quick understanding**: Get immediate comprehension of results
- **Business reporting**: Focus on implications rather than raw numbers
- **Stakeholder communication**: Explain findings to non-technical audiences

**Example Summary output:**
```markdown
## Sales Performance Analysis

**Key Findings:**
- Q4 revenue reached $2.4M, representing a **23% increase** over Q3
- The Electronics category drove 45% of total growth
- Western region outperformed all others with 18% higher sales

**Notable Trends:**
- Mobile device sales peaked in December (holiday season)
- B2B customers showed 31% higher average order values
- Customer retention improved to 87% from 82% last quarter

**Recommendations:**
- Increase inventory for top-performing electronics products
- Investigate success factors in Western region for replication
- Focus marketing efforts on B2B segment expansion
```

### Data Tab: Structured Results

**What it contains:**
- JSON data with actual values from your data source
- Data may be from one or more underlying databases
- Calculated metrics and aggregations
- Downloadable data in CSV format

---

## Real-Time Result Streaming

### Streaming Experience

Simba Intelligence uses Server-Sent Events (SSE) to provide live updates during query processing:

**Processing stages you'll see:**
1. **Query validation**: "Analyzing your question..."
2. **AI processing**: "Generating database queries..."
3. **Query execution**: "Retrieving data from your databases..."
4. **Result analysis**: "Analyzing results and generating insights..."
5. **Completion**: "Results ready!"

### Progress Indicators

**Visual feedback includes:**
- **Animated loading spinner** during active processing
- **Progress text updates** describing current processing stage
- **Partial results** appearing as they become available
- **Error notifications** if issues occur during processing
- **Completion status** when all processing is finished

### Managing Long-Running Queries

**For complex queries that take longer:**
- **Progress updates** keep you informed of processing stages
- **Partial streaming** shows results as they become available
- **Cancel option** available during execution (Shift+F5 or stop button)
- **Timeout protection** prevents indefinitely running queries
- **Retry capability** for queries that encounter temporary issues

**Performance expectations:**
- **Simple queries**: 2-10 seconds
- **Moderate complexity**: 10-30 seconds  
- **Complex aggregations**: 30-60 seconds
- **Large datasets**: 1-2 minutes

---

## Interactive Features

### Rating and Feedback System

**Purpose**: Help improve AI accuracy and provide feedback to administrators

**Rating options:**
- **👍 Thumbs Up**: Query worked well and results were helpful
- **👎 Thumbs Down**: Results were incorrect, incomplete, or unhelpful

**When to rate queries:**
- **Always rate** significant queries to improve system learning
- **Thumbs up** when results match your expectations and provide value
- **Thumbs down** when results are wrong, confusing, or unhelpful
- **Rate edge cases** to help improve AI handling of complex scenarios

**What your ratings accomplish:**
- Improve AI model performance for similar future queries
- Help administrators identify common issues or successful patterns
- Contribute to system optimization for your team's use cases
- Enable better semantic caching for similar questions

---

## Data Export Capabilities

### CSV Export Functionality

**What gets exported:**
- **Complete result sets**: All data from your query
- **Formatted data**: Properly structured with headers and data types
- **Calculated fields**: Any metrics computed by the query are included
- **Filtered results**: Only data matching any applied filters

**Export process:**
1. **Run your query** and wait for complete results
2. **Switch to Data tab** to verify the data you want to export
3. **Click "Export" button** (appears when results are complete)
4. **Choose CSV format** (additional formats may be available)
5. **Download begins** with descriptive filename based on your query

**File naming convention:**
- Format: `simba-intelligence-export-YYYY-MM-DD-HHMMSS.csv`
- Example: `simba-intelligence-export-2024-12-15-143022.csv`

### Export Data Structure

**CSV file contains:**
- **Header row**: Column names from your query results
- **Data rows**: All result records with proper formatting
- **Preserved data types**: Numbers as numbers, dates as dates, text as text
- **Consistent formatting**: Standardized number and date formats

**Example exported CSV structure:**
```csv
Product Name,Total Revenue,Order Count,Average Order Value,Growth Rate
Electronics Pro,245600.50,1247,197.03,15.3%
Home Essentials,198750.25,892,222.76,8.7%
Sports Equipment,156890.75,743,211.23,22.1%
```

### Working with Exported Data

**Compatible tools:**
- **Microsoft Excel**: Opens directly with proper formatting
- **Google Sheets**: Import via File → Import → Upload
- **Business Intelligence tools**: Tableau, Power BI, Looker
- **Database tools**: Import into databases for further processing
- **Programming languages**: Python pandas, R, etc.

**Best practices for exported data:**
- **Verify completeness**: Check row counts match expected results
- **Preserve formatting**: Maintain date and number formats when importing
- **Document context**: Note the query and parameters used to generate data
- **Version control**: Keep track of when data was exported for reproducibility

---

## Troubleshooting Common Issues

### Result Display Problems

**"No results found"**
- **Verify filters**: Check that date ranges and conditions are appropriate
- **Broaden criteria**: Try less restrictive search terms or conditions
- **Check data availability**: Ensure data exists for your query timeframe
- **Review permissions**: Confirm you have access to the queried data

**"Results incomplete or cut off"**
- **Check pagination**: Look for next/previous page controls
- **Use export**: CSV export contains complete results even if display is limited
- **Refine query**: More specific queries often return more focused, complete results

### Export Issues

**"Export button not available"**
- **Wait for completion**: Export only available after query fully completes
- **Check result size**: Very large results might require different export approach
- **Verify permissions**: Ensure you have export permissions for the data source

**"Export file corrupted or incomplete"**
- **Retry export**: Network issues can cause incomplete downloads
- **Check file size**: Compare downloaded file size to expected data volume
- **Verify format**: Ensure your tool can properly read CSV format

### Performance Issues

**"Query taking too long"**
- **Check query complexity**: Complex questions require more processing time
- **Monitor progress**: Watch streaming updates to understand processing stage
- **Consider query scope**: Large datasets or wide date ranges increase processing time
- **Use specific criteria**: More focused queries generally perform better

**"Results appear slowly"**
- **Network connectivity**: Ensure stable internet connection for streaming
- **Browser performance**: Close unnecessary tabs and refresh browser if needed
- **Database load**: Performance varies with database server load and concurrent users

---

## Best Practices

### Optimizing Query Results

**Write effective queries:**
1. **Be specific**: "Top 10 products by revenue this quarter" vs "show me products"
2. **Include timeframes**: Specify relevant date ranges for your analysis
3. **Use business terms**: Reference metrics as they're known in your organization
4. **Focus questions**: Ask about specific aspects rather than broad overviews

**Validate results systematically:**
1. **Review Summary**: Understand AI interpretation of your data
2. **Check Data tab**: Verify raw numbers align with expectations
3. **Cross-reference**: Compare results with known benchmarks or previous analyses

### Effective Export Workflows

**Planning your exports:**
1. **Define purpose**: Know how you'll use exported data before exporting
2. **Verify completeness**: Check Data tab to ensure you're getting expected results
3. **Document context**: Note query details for future reference
4. **Test with samples**: Export small datasets first to verify format compatibility

**Managing exported data:**
1. **Organize files**: Use consistent naming and folder structures
2. **Version control**: Track when data was exported and query parameters used
3. **Share appropriately**: Consider data sensitivity when sharing exported files
4. **Clean up regularly**: Remove old export files to manage storage

### Collaborative Analysis

**Sharing results effectively:**
- **Use Summary tab** for executive and business stakeholder communication
- **Share Data exports** for detailed technical analysis with colleagues
- **Reference Query tab** when collaborating with database professionals
- **Rate queries** to improve results for your entire team

**Building on results:**
- **Document successful query patterns** for reuse by team members
- **Share interesting findings** via Summary explanations
- **Use exports as input** for deeper analysis in specialized tools
- **Create query libraries** of frequently used questions and approaches

---

## Integration with Other Tools

### API Integration

**For developers and advanced users:**
- **Programmatic querying**: Use API endpoints to automate query execution
- **Result processing**: Build applications that consume Simba Intelligence results
- **Export automation**: Programmatically export data for integration workflows

---

*Ready to start working with query results? Head to the [Playground](./playground) to run your first query and explore the different result formats. Remember: the best insights come from combining AI interpretation (Summary) with detailed data validation (Data tab) and understanding the underlying methodology (Query tab).*