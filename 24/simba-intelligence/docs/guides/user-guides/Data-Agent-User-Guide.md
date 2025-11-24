# Data Agent User Guide

The Data Agent is Simba Intelligence's AI-powered tool for automatically creating data sources. Instead of manually configuring database connections and schemas, you simply describe what you need or upload a screenshot of your desired dashboard, and the AI agent handles the technical setup for you.

## Overview

### What is the Data Agent?

Think of the Data Agent as your intelligent data assistant. Just as you might describe a project to a colleague and have them set up the necessary resources, the Data Agent listens to your data requirements and automatically:

- **Analyzes your needs** from text descriptions or dashboard images
- **Connects to your databases** using available data connections  
- **Creates optimized data sources** ready for querying
- **Configures relationships and fields** based on your requirements

### When to Use the Data Agent

**Perfect for:**
- Creating new data sources for dashboards or reports
- Prototyping data connections quickly
- Converting dashboard mockups into working data sources
- Exploring new databases without manual schema analysis

**Example scenarios:**
- *"I need to analyze our sales performance by region and product category"*
- *"Create a data source for this customer dashboard mockup"*
- *"Set up connections to track inventory levels across our warehouses"*

---

## Prerequisites

Before using the Data Agent, ensure you have:

### 1. Required Permissions
- **ROLE_CREATE_SOURCES** permission or higher
- Access to the Data Agent interface (typically at `/data-source-agent`)

### 2. LLM Configuration
- At least one AI provider configured (Google Vertex AI, OpenAI, or AWS Bedrock)
- Active AI provider with sufficient quota/credits

### 3. Data Connections
- At least one data connection configured and available
- Network connectivity from Simba Intelligence to your target databases
- Appropriate database permissions for the data connection credentials

> **💡 Tip**: If you don't have these prerequisites, the Data Agent will guide you through the setup process or direct you to the appropriate configuration pages.

---

## Getting Started

### Step 1: Access the Data Agent

1. **Navigate to the Data Agent**:
   - Go to the main Simba Intelligence interface
   - Click **"Data Agent"** in the navigation menu
   - Or visit directly: `http://your-domain/data-source-agent`

2. **Verify prerequisites**:
   - The system automatically checks for LLM configuration
   - If missing, you'll be prompted to configure an AI provider first

### Step 2: Initial Setup Check

When you first access the Data Agent, it performs automatic validation:

- ✅ **LLM Provider**: Confirms AI services are configured and accessible
- ✅ **Data Connections**: Verifies available database connections
- ✅ **Permissions**: Ensures you have rights to create data sources

If any prerequisites are missing, you'll see clear instructions for resolving them.

---

## Creating Data Sources

The Data Agent supports two primary input methods for understanding your requirements:

### Method 1: Text Description

**Best for**: Clear data requirements, existing knowledge of your data structure

1. **Click "Get Started"** on the Data Agent welcome screen

2. **Describe your data needs** in the text area (optional). Be specific about:
   - **What data you want to analyze**
   - **Key metrics or fields of interest**
   - **Intended use case or analysis goals**

**Example descriptions:**

```
Sales Analysis:
I need to analyze our sales performance data including:
- Revenue by product category and region
- Monthly and quarterly trends
- Customer segmentation analysis
- Top-performing sales representatives

Include data from our CRM and ERP systems where possible.
```

```
Inventory Management:
Create a data source for warehouse inventory tracking:
- Current stock levels by location
- Product movement history
- Reorder points and alerts
- Supplier performance metrics

Focus on our main distribution centers.
```

### Method 2: Image Analysis (Dashboard Mockups)

**Best for**: Visual dashboard designs, mockups, screenshots of desired outputs

1. **Upload an image** of your desired dashboard or data visualization

2. **Supported image formats**: PNG, JPG, JPEG (max 10MB)

3. **Drag and drop or click** to browse for your image file

4. **Add context** (optional): Include additional text description to clarify requirements

**What the AI analyzes in images:**
- Chart types and visualizations
- Data labels and field names
- Filters and interactive elements
- Layout and grouping patterns
- Implied data relationships

> **🔍 Image Analysis Tip**: The AI works best with clear, well-labeled dashboard mockups. Hand-drawn sketches work too, as long as labels and relationships are visible.

### Method 3: Combined Approach

For best results, combine both methods:
1. Upload a dashboard mockup or screenshot
2. Add text description with additional context and requirements
3. Specify any particular data sources or constraints

---

## The AI Agent Workflow

Once you provide your requirements, the Data Agent follows an intelligent workflow:

### Phase 1: Requirement Analysis
**What happens:**
- AI processes your text description and/or image
- Identifies key data entities, relationships, and metrics
- Determines optimal data source structure

**You'll see:**
- Real-time analysis progress updates
- Extracted requirements summary
- Identified data patterns and relationships

### Phase 2: Connection Selection
**What happens:**
- System presents available data connections
- You select which database connection to use

### Phase 3: Data Source Creation
**What happens:**
- AI agent connects to selected database
- Analyzes schema and available data
- Creates optimized data source configuration
- Tests queries and validates data access

**Real-time progress shows:**
- Database connection establishment
- Schema analysis progress
- Query generation and testing
- Data source optimization

### Phase 4: Results and Next Steps
**What happens:**
- Data source is created and ready for use
- System provides access options
- Suggests next steps for data exploration

---

## Understanding AI Agent Progress

The Data Agent provides detailed, real-time feedback during the creation process:

### Progress Event Types

**📋 Info Events**: General progress updates and status information

**🚀 Start Events**: Beginning of major workflow phases  

**✅ Complete Events**: Successful completion of tasks

**⚠️ Warning Events**: Non-critical issues or suggestions

**❌ Error Events**: Problems requiring attention

### Typical Progress Flow

1. **"Analyzing your requirements..."**
   - Processing text description or image
   - Identifying key data elements

2. **"Connecting to database..."**
   - Establishing connection via selected data connection
   - Validating access permissions

3. **"Exploring database schema..."**
   - Discovering available tables and relationships
   - Mapping to your requirements

4. **"Creating data source configuration..."**
   - Building optimized queries and views
   - Setting up field mappings and relationships

5. **"Testing and validating..."**
   - Running test queries
   - Verifying data accessibility

6. **"Data source created successfully!"**
   - Final configuration complete
   - Ready for querying

---

## Working with Results

### When Data Source Creation Succeeds

**You'll receive:**
- ✅ **Confirmation message** with data source details
- 🔗 **Direct links** to view or query the new data source
- 📊 **Summary** of created fields and relationships

**Available actions:**

1. **"Open Created Data Source"**
   - Takes you to the Data Sources management page
   - Shows your new data source highlighted
   - Allows for manual configuration

2. **"Go to Playground"**  
   - Opens the Playground interface with your data source pre-selected
   - Ready for immediate natural language querying
   - Perfect for testing and validation

### If Creation Encounters Issues

**Common scenarios and solutions:**

**Permission Issues:**
- Verify database user has appropriate read permissions
- Check that connection configuration is correct
- Contact your database administrator if needed

**Schema Complexity:**
- AI may need more specific guidance for complex databases
- Try breaking down requirements into smaller, focused data sources
- Consider creating multiple related data sources

**Connectivity Problems:**
- Ensure network connectivity between Simba Intelligence and target database
- Verify firewall rules allow database connections
- Check that database server is accessible and running

---

## Best Practices

### Writing Effective Descriptions

**✅ Do:**
- Be specific about metrics and dimensions you need
- Mention intended use cases (reports, dashboards, analysis)
- Include relevant business context
- Specify time ranges or filters if important

**❌ Avoid:**
- Vague requests like "connect to our database"
- Technical jargon without business context  
- Overly complex requirements in a single request
- Assumptions about database structure

### Image Upload Guidelines

**For best results:**
- Use clear, well-labeled mockups or screenshots
- Ensure text in images is readable
- Include legends and axis labels
- Show the full dashboard or visualization context
- Use high-contrast images (dark text on light background)

### Managing Complex Requirements

**Break down large projects:**
1. Start with core metrics and entities
2. Create foundational data sources first
3. Build additional sources for related data
4. Combine sources in the Playground interface

**Iterative approach:**
- Create a basic data source first
- Test with sample queries
- Refine or create additional sources as needed

---

## Advanced Features

### Image Analysis Details

When you upload an image, the AI performs sophisticated analysis:

**Visual Component Recognition:**
- Chart types (bar, line, pie, tables, etc.)
- Data labels and field names
- Filter controls and parameters
- Layout and grouping structures

**Structured Analysis Output:**
- Dashboard overview and purpose
- Individual visual components
- Required filters and parameters  
- Data source recommendations
- Implementation suggestions

You can view this detailed analysis in an expandable section after image processing.

### Context Combination

The most powerful approach combines multiple inputs:

```
Image: Upload dashboard mockup
Text: "This dashboard should focus on Q3 and Q4 data for our 
       European operations. Include budget vs. actual comparisons 
       and highlight products with >10% variance."
```

This gives the AI both visual structure and business context for optimal results.

### Data Source Refinement

After creation, you can:
- **Modify field selections** through the Data Sources interface
- **Adjust relationships** between tables
- **Add calculated fields** or custom metrics
- **Configure security filters** for different user groups

---

## Troubleshooting Common Issues

### "No LLM Configuration Found"

**Problem**: AI provider not configured or accessible
**Solution**: 
1. Navigate to LLM Configuration page
2. Add at least one AI provider (Vertex AI, OpenAI, etc.)
3. Verify credentials and test connection
4. Return to Data Agent

### "No Data Connections Available"

**Problem**: No database connections configured
**Solution**:
1. Go to Data Connections page
2. Add connection to your database (SQL Server, PostgreSQL, Snowflake)
3. Test connectivity
4. Return to Data Agent

### "Data Agent Execution Failed"

**Possible causes and solutions:**

**Network connectivity:**
- Check database server is accessible
- Verify firewall rules and network configuration
- Ensure connection has proper network access

**Database permissions:**
- Verify connection user has SELECT permissions on required tables
- Check database user exists and password is correct
- Ensure database allows connections from Simba Intelligence

**Resource constraints:**
- AI provider may be rate-limited or out of quota
- Database may be under heavy load
- Check system resources and retry

### "Incomplete Data Source Configuration"

**Problem**: AI couldn't fully analyze or configure requirements
**Solution**:
- Review and simplify your description
- Try breaking complex requirements into smaller pieces
- Ensure uploaded images are clear and well-labeled
- Provide additional context or constraints

---

## Security and Governance

### Data Access Control

The Data Agent respects existing security configurations:
- **Database permissions**: Only accesses data the connector user can read
- **User roles**: Honors your Simba Intelligence permission level
- **Data source security**: Applies any configured row-level or column-level security

### Audit and Compliance

All Data Agent activities are logged:
- **Creation requests**: Who created what data source and when
- **AI interactions**: Queries sent to AI providers (without sensitive data)
- **Database access**: What tables and schemas were analyzed
- **Success/failure events**: Complete audit trail of all operations

### Privacy Considerations

**Data processing:**
- Text descriptions are sent to AI providers for analysis
- Database schema information (table/column names) may be shared with AI
- Actual data values are never sent to external AI services
- Image uploads are processed by AI vision services

**Security measures:**
- All AI provider communications use encrypted connections
- Credentials and sensitive configuration stored securely
- Database connections use provided authentication

---

## Integration with Other Features

### Using Created Data Sources

After creating a data source with the Data Agent:

**In the Playground:**
- Natural language querying against your data
- Real-time results with explanations
- Export capabilities for further analysis

**In Data Sources Management:**
- Fine-tune field mappings and relationships
- Configure additional security or performance settings
- Monitor usage and performance metrics

**Via API:**
- Programmatic access to your created data sources
- Integration with custom applications
- Automated query execution and result processing

### Combining Multiple Data Sources

Create related data sources and combine them:
1. Use Data Agent to create core data source
2. Create additional sources for related data
3. Use Playground to query across multiple sources
4. Build comprehensive analyses combining different data sets

---

## Tips for Success

### Getting Better Results

1. **Start simple**: Create basic data sources before tackling complex requirements
2. **Iterate**: Use initial results to refine and create additional sources
3. **Test immediately**: Verify data sources work as expected using Playground
4. **Document requirements**: Keep notes on successful patterns for future use

### Performance Optimization

- **Focus requirements**: Specific, focused data sources perform better than broad ones
- **Use appropriate data connection**: Match data source to optimal database connection
- **Monitor query performance**: Check how created sources perform in practice
- **Refine as needed**: Adjust data source configuration based on actual usage

### Collaboration

- **Share successful patterns**: Document effective description templates
- **Coordinate data connection usage**: Ensure team members don't overwhelm database connections
- **Review created sources**: Have team members validate AI-generated configurations
- **Build incrementally**: Create foundational sources that others can extend

---

*Ready to create your first AI-powered data source? Head to the [Data Agent](./data-source-agent) and describe what you need. The AI will handle the technical details while you focus on your analysis goals.*
