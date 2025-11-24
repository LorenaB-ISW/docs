# LLM Provider Configuration

This guide explains how to configure Large Language Model (LLM) providers in Simba Intelligence. LLM providers power the AI capabilities that enable natural language querying, data source creation, and intelligent analysis features.

## Overview

### What are LLM Providers?

LLM providers are external AI services that power Simba Intelligence's natural language capabilities. Think of them as the "brains" behind the system that understand your questions, analyze your data requirements, and generate intelligent responses.

When you ask *"What were our top-selling products last quarter?"* in the Playground, an LLM provider:
1. **Understands** your natural language question
2. **Analyzes** your available data sources  
3. **Generates** appropriate SQL queries
4. **Interprets** the results into meaningful insights

### Supported LLM Providers

Simba Intelligence supports multiple LLMs with [structured output](https://python.langchain.com/docs/how_to/structured_output/) support.
Older models like GPT-3.5 do not support this and cannot be used.

The below LLMs have been tested with the Data Source Agent and Query.

| Provider        | Model                 | Status     | Quality  | Cost | Notes                         |
| --------------- | --------------------- | ---------- | -------- | ---- | ----------------------------- |
| 🟢 Vertex AI    | Gemini 2.0 Flash      | ✅ Works   | Standard | $    | Most tested, reliable         |
| 🟢 Vertex AI    | Gemini 2.5 Flash      | ✅ Works   | High     | $$   | Newer, more expensive ¹       |
| 🟢 Vertex AI    | Gemini 2.5 Flash Lite | ❌ Limited | Unstable | $    | Fast and cheap, but unstable  |
| 🟢 Vertex AI    | Gemini 2.5 Pro        | ✅ Works   | High     | $$$  | High quality and cost, slow ² |
| 🔵 Azure OpenAI | GPT-4o                | ✅ Works   | Standard | $$$  | Higher cost                   |
| 🔵 Azure OpenAI | GPT-4o-mini           | ❌ Limited | Unstable | $    | Unreliable query generation   |
| 🔵 Azure OpenAI | GPT-4.1               | ✅ Works   | High     | $$   | Solid performance             |
| 🔵 Azure OpenAI | GPT-4.1-mini          | ✅ Works   | Standard | $    | Quality trade-offs            |
| 🟠 AWS Bedrock  | Nova Pro              | ✅ Works   | Standard | $$   | Adequate performance          |
| 🟠 AWS Bedrock  | Claude Sonnet 4       | ✅ Works   | High     | $$$  | High quality, high cost       |

<details>
  <summary>Gemini 2.5 Configuration Notes</summary>

1. **Gemini 2.5 Flash**: Set `thinking_budget` to `0`
2. **Gemini 2.5 Pro**: Set `thinking_budget` between `128` and `32767`

</details>

💡 **Recommendation**: Start with Vertex AI Gemini 2.0 Flash for development and testing, 2.5 Flash for production.

### Provider Capabilities

**💬 Chat Capability**
- Natural language understanding and generation
- Query interpretation and response creation
- Business insight generation

**🔍 Embeddings Capability**  
- Semantic search and similarity matching
- Intelligent caching of similar queries
- Content understanding for better results

**👁️ Vision Capability**
- Dashboard image analysis
- Screenshot-to-data-source conversion
- Visual mockup interpretation

---

## Prerequisites

### Required Permissions

**To configure LLM providers, you need:**
- **Supervisor role** in Simba Intelligence
- **Access to LLM Configuration** interface (`/llm-configuration`)
- **Administrative privileges** for system-wide AI configuration

### AI Provider Account Requirements

**For each provider you want to use:**

**Google Vertex AI:**
- Google Cloud Platform account with billing enabled
- Vertex AI API enabled in your GCP project
- Service account with Vertex AI permissions
- Sufficient API quota for your expected usage

**Azure OpenAI:**
- Azure OpenAI resource
- API key with appropriate usage limits
- Sufficient credits/quota for your organization's needs

**AWS Bedrock:**
- AWS account with Bedrock access enabled
- IAM user or role with Bedrock permissions
- Model access granted for Claude or other desired models
- Appropriate usage quotas configured

---

## Accessing LLM Configuration

### Navigation to Configuration Interface

1. **Log into Simba Intelligence** with supervisor credentials
2. **Navigate to LLM Configuration**:
   - **Option 1**: Visit directly at `http://your-domain/llm-configuration`
   - **Option 2**: User menu → "LLM Configuration"
3. **Verify access**: You should see a tabbed interface with available providers

### Configuration Interface Overview

The LLM Configuration interface provides:
- **Provider tabs**: Separate tabs for each supported AI provider
- **Configuration forms**: Provider-specific credential and parameter forms
- **Capability management**: Enable/disable different AI capabilities per provider
- **Testing tools**: Built-in connection testing and validation
- **Status indicators**: Real-time status of provider configurations

---

## Google Vertex AI Configuration

Google Vertex AI provides the most comprehensive AI capabilities, including advanced vision analysis for dashboard image processing.

### Prerequisites for Vertex AI

**Google Cloud Platform setup:**
1. **Create or select a GCP project** with billing enabled
2. **Enable Vertex AI API**:
   ```bash
   gcloud services enable aiplatform.googleapis.com
   ```
3. **Create a service account**:
   ```bash
   gcloud iam service-accounts create simba-intelligence-ai \
     --display-name="Simba Intelligence AI Service"
   ```
4. **Grant necessary permissions**:
   ```bash
   gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
     --member="serviceAccount:simba-intelligence-ai@YOUR_PROJECT_ID.iam.gserviceaccount.com" \
     --role="roles/aiplatform.user"
   ```
5. **Generate service account key**:
   ```bash
   gcloud iam service-accounts keys create simba-intelligence-key.json \
     --iam-account=simba-intelligence-ai@YOUR_PROJECT_ID.iam.gserviceaccount.com
   ```

### Vertex AI Configuration Process

1. **Access Vertex AI tab** in LLM Configuration interface

2. **Enter service account JSON**:
   - Copy the complete contents of your service account JSON file
   - Paste into the credentials text area
   - The JSON should include all required fields:

   ```json
   {
     "type": "service_account",
     "project_id": "your-gcp-project-id",
     "private_key_id": "your-private-key-id",
     "private_key": "-----BEGIN PRIVATE KEY-----\nYour-Private-Key-Content-Here\n-----END PRIVATE KEY-----\n",
     "client_email": "simba-intelligence-ai@your-project.iam.gserviceaccount.com",
     "client_id": "your-client-id",
     "auth_uri": "https://accounts.google.com/o/oauth2/auth",
     "token_uri": "https://oauth2.googleapis.com/token",
     "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
     "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/simba-intelligence-ai%40your-project.iam.gserviceaccount.com"
   }
   ```

3. **Configure global parameters**:
   ```
   Location: us-central1 (or your preferred region)
   Default timeout: 120 seconds
   Max retries: 3
   ```

4. **Enable and configure capabilities**:

   **Chat Capability:**
   ```
   Model Name: gemini-2.0-flash
   Temperature: 0.7 (creativity level, 0.0-1.0)
   Max Tokens: 8192
   Location: us-central1
   ```

   **Embeddings Capability:**
   ```
   Model Name: text-embedding-004
   Location: us-central1
   Dimensions: 768 (default)
   ```

   **Vision Capability:**
   ```
   Model Name: gemini-2.0-flash
   Location: us-central1
   Max Images: 10 per request
   ```

5. **Save configuration**:
   - Click **"Save"** to store the configuration
   - Verify all capabilities show as "Active"

### Vertex AI Cost Management

**Understanding costs:**
- **Chat usage**: Charged per input/output token
- **Embeddings**: Charged per text embedding generated  
- **Vision**: Charged per image analyzed
- **Model selection**: Different models have different pricing

**Cost optimization tips:**
- Monitor usage in Google Cloud Console
- Set up billing alerts for unexpected usage
- Use semantic caching to reduce redundant API calls
- Choose appropriate models for different use cases

---

## Azure OpenAI Configuration

Azure OpenAI offers the proven GPT models with enterprise-grade security and compliance.

### Azure OpenAI Configuration

**Prerequisites:**
- Azure subscription with OpenAI resource created
- Deployed models in your Azure OpenAI resource
- API key and endpoint details

**Configuration steps:**

1. **Access Azure OpenAI tab** in LLM Configuration interface

2. **Enter Azure-specific details**:
   ```
   API Key: your-azure-openai-api-key
   Azure Endpoint: https://your-resource-name.openai.azure.com/
   API Version: 2023-05-15 (or latest supported)
   ```

3. **Configure deployments**:
   ```
   Chat Deployment Name: your-gpt4-deployment-name
   Embeddings Deployment Name: your-embeddings-deployment-name (optional)
   ```

4. **Set capability parameters**:
   ```
   Temperature: 0.7
   Max Tokens: 4000
   Timeout: 120 seconds
   ```

---

## AWS Bedrock Configuration

AWS Bedrock provides access to various foundation models including Anthropic's Claude, with enterprise-grade AWS integration.

### Prerequisites for AWS Bedrock

**AWS account setup:**
1. **AWS account** with Bedrock access enabled in your region
2. **IAM credentials** with appropriate Bedrock permissions
3. **Model access** granted for desired models (Claude, etc.)
4. **Sufficient service quotas** for your expected usage

**Required IAM permissions:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream",
        "bedrock:GetModel",
        "bedrock:ListFoundationModels"
      ],
      "Resource": "*"
    }
  ]
}
```

### Bedrock Configuration Process

1. **Access AWS Bedrock tab** in LLM Configuration interface

2. **Enter IAM credentials**:
   ```
   Access Key ID: AKIA...your-access-key
   Secret Access Key: your-secret-access-key
   Region: us-east-1 (or your preferred region)
   Session Token: optional-if-using-temporary-credentials
   ```

3. **Configure global parameters**:
   ```
   Default Region: us-east-1
   Request Timeout: 120 seconds
   Max Retries: 3
   ```

4. **Enable chat capability**:
   ```
   Model ID: anthropic.claude-3-sonnet-20240229-v1:0
   Temperature: 0.7
   Max Tokens: 4000
   ```

5. **Configure embeddings** (if available):
   ```
   Model ID: amazon.titan-embed-text-v1
   Dimensions: 1536
   ```

### Bedrock Model Selection

**Available model families:**
- **Anthropic Claude**: Excellent for reasoning and analysis
- **AI21 Labs Jurassic**: Strong language understanding
- **Cohere Command**: Multilingual capabilities
- **Amazon Titan**: AWS-native models with competitive performance

**Model selection criteria:**
- **Performance requirements**: Response quality and speed
- **Cost considerations**: Different models have different pricing
- **Regional availability**: Not all models available in all regions
- **Compliance requirements**: Some models may have specific compliance certifications

---

## Managing Capabilities

### Understanding Capability Types

**Chat Capability:**
- **Purpose**: Natural language understanding and generation
- **Used for**: Query interpretation, response generation, insight creation
- **Configuration**: Model selection, temperature, token limits

**Embeddings Capability:**
- **Purpose**: Converting text to numerical representations for similarity matching
- **Used for**: Semantic caching, query similarity detection, content understanding
- **Configuration**: Model selection, dimension settings

**Vision Capability**:
- **Purpose**: Image analysis and understanding
- **Used for**: Dashboard mockup analysis, screenshot interpretation
- **Configuration**: Model selection, image processing limits

### Capability Configuration Best Practices

**Chat capability optimization:**
- **Temperature settings**: 
  - `0.0-0.3`: Deterministic, factual responses
  - `0.4-0.7`: Balanced creativity and accuracy (recommended)
  - `0.8-1.0`: Creative but potentially less accurate
- **Token limits**: Set based on expected query/response complexity
- **Model selection**: Balance performance, cost, and feature requirements

**Embeddings optimization:**
- **Dimension selection**: Higher dimensions = better accuracy but higher cost
- **Model compatibility**: Ensure embedding model matches your use case
- **Caching strategy**: Configure appropriate cache retention for embeddings

**Multi-provider strategy:**
- **Primary provider**: Choose most reliable provider for critical capabilities
- **Fallback providers**: Configure secondary providers for redundancy
- **Capability specialization**: Use different providers for different capabilities

---

## Testing and Validation

### Manual Validation Procedures

**After configuring providers:**

1. **Test basic chat capability**:
   - Go to Playground interface
   - Ask a simple question about your data
   - Verify natural language response is generated

2. **Test embeddings** (if configured):
   - Ask similar questions multiple times  
   - Verify responses improve with semantic caching
   - Check cache hit rates in monitoring

3. **Test vision capability** (Vertex AI):
   - Go to Data Agent interface
   - Upload a dashboard screenshot
   - Verify image analysis produces relevant insights

4. **Performance validation**:
   - Monitor response times during testing
   - Check API usage in provider dashboards
   - Verify no rate limiting or quota issues

### Troubleshooting Common Issues

**Authentication failures:**
```
Error: "Invalid API key" or "Authentication failed"
Solutions:
- Verify API key is correctly copied and not truncated
- Check API key permissions and scope
- Ensure billing is enabled and account is in good standing
- For service accounts, verify JSON formatting is correct
```

**Model access issues:**
```
Error: "Model not found" or "Access denied"
Solutions:
- Verify model name/ID is correct for your provider
- Check that model access is granted in provider console
- Ensure your region supports the requested model
- For AWS Bedrock, confirm model access is explicitly granted
```

**Rate limiting or quota issues:**
```
Error: "Rate limit exceeded" or "Quota exceeded"
Solutions:
- Check usage in provider dashboard
- Implement request throttling or retries
- Upgrade service tier if needed
- Distribute load across multiple providers
```

---

## Security Best Practices

### Credential Management

**Secure storage:**
- **Never store credentials in code** or configuration files
- **Use environment variables** or secure secret management systems
- **Encrypt credentials at rest** using appropriate encryption
- **Limit credential access** to authorized personnel only

**Regular rotation:**
- **API keys**: Rotate every 90 days or according to security policy
- **Service accounts**: Regenerate keys quarterly
- **Access review**: Regularly audit who has access to credentials
- **Emergency procedures**: Have procedures for immediate credential revocation

### Network Security

**Data privacy:**
- **Understand data flow**: Know what data is sent to each provider
- **Regional compliance**: Use appropriate regions for data sovereignty
- **Data retention**: Understand provider data retention policies

### Access Control

**Role-based access:**
- **Limit configuration access**: Only supervisors can configure LLM providers
- **Separate development/production**: Use different credentials for different environments
- **Audit configuration changes**: Log all changes to LLM provider configurations
- **Emergency access**: Maintain emergency procedures for credential issues

---

## Cost Management and Optimization

### Understanding AI Provider Costs

**Cost factors:**
- **Token usage**: Both input (prompts) and output (responses) tokens are charged
- **Model selection**: Premium models cost more than basic models
- **Capability type**: Chat, embeddings, and vision have different pricing
- **Request frequency**: High-volume usage may trigger different pricing tiers

### Cost Optimization Strategies

**Provider selection:**
- **Cost comparison**: Compare pricing across providers for your use cases
- **Feature optimization**: Use expensive features (like vision) only when necessary
- **Load balancing**: Distribute load to take advantage of different pricing models

---

*Ready to configure your first LLM provider? Start with the provider that best matches your organization's requirements and follow the step-by-step instructions above. Remember: a well-configured AI foundation is essential for optimal Simba Intelligence performance.*
