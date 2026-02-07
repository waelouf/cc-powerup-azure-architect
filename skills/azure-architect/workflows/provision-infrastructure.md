<required_reading>
<title>Required Reading Before Starting</title>
<description>
Read these reference files to understand infrastructure provisioning best practices, service options, naming conventions, and security patterns before creating resources.
</description>

<reference file="references/infrastructure-as-code.md">
Bicep vs Terraform comparison, Azure Verified Modules, state management strategies, template organization
</reference>

<reference file="references/compute-services.md">
Decision matrix for choosing compute services (App Service, AKS, Functions, VMs, Container Apps) with pricing and use cases
</reference>

<reference file="references/resource-organization.md">
Naming conventions, tagging strategies, resource group design, management group hierarchy
</reference>

<reference file="references/security-identity.md">
Managed Identity patterns, Key Vault integration, RBAC role assignments, zero-credentials architecture
</reference>
</required_reading>

<objective>
Provision production-grade Azure infrastructure from scratch using Infrastructure as Code (Bicep or Terraform). Creates resource groups, compute resources (App Service, AKS, Functions), databases (Azure SQL, Cosmos DB), storage accounts, networking components, and monitoring infrastructure with proper naming conventions, tagging strategies, security configurations (Managed Identity, HTTPS-only, Private Endpoints), and cost-appropriate SKU sizing for each environment (dev/staging/prod). All resources are defined in version-controlled IaC templates following 2024-2025 Azure best practices.
</objective>

<prerequisites>
<prerequisite>Azure CLI installed and authenticated (az login completed successfully)</prerequisite>
<prerequisite>Azure subscription with Contributor or Owner role assigned to your account</prerequisite>
<prerequisite>For Bicep: Azure CLI includes Bicep (no separate install needed)</prerequisite>
<prerequisite>For Terraform: Terraform CLI installed (version 1.0 or higher)</prerequisite>
<prerequisite>Git repository initialized for version controlling IaC templates</prerequisite>
<prerequisite>Text editor or IDE with IaC support (VS Code with Bicep/Terraform extensions recommended)</prerequisite>
</prerequisites>

<process>
<step number="1" name="gather_requirements">
<title>Gather Requirements</title>
<description>
Collect comprehensive information about the application, infrastructure needs, constraints, and compliance requirements to guide architecture design, service selection, and resource sizing decisions.
</description>

<questions>
<question name="application_type">
<prompt>What type of application are you building?</prompt>
<options>
<option>Web application (frontend + backend)</option>
<option>REST/GraphQL API</option>
<option>Microservices architecture</option>
<option>Batch processing / background jobs</option>
<option>Data processing pipeline</option>
<option>Static website</option>
<option>Other (describe)</option>
</options>
</question>

<question name="traffic_load">
<prompt>What is the expected traffic/load?</prompt>
<details>
- Number of users per day
- Requests per second (peak and average)
- Data volume processed
- Concurrent connections needed
</details>
<purpose>Helps determine appropriate SKU sizing and auto-scaling configuration</purpose>
</question>

<question name="azure_services">
<prompt>Are there specific Azure services required?</prompt>
<examples>
- Azure SQL Database (relational data)
- Cosmos DB (NoSQL/multi-model)
- Azure Cache for Redis (caching/sessions)
- Azure Storage (blobs/files/queues)
- Service Bus (messaging)
- Event Hubs (streaming)
- Azure Functions (serverless)
</examples>
</question>

<question name="budget_constraints">
<prompt>What are the budget constraints?</prompt>
<considerations>
- Monthly spending limit
- Cost optimization priorities
- Acceptable vs prohibited services
- Reserved Instance commitments possible?
</considerations>
</question>

<question name="environment">
<prompt>Which environment are we provisioning?</prompt>
<options>
<option value="dev">Development (minimal cost, auto-shutdown)</option>
<option value="staging">Staging (production-like, testing)</option>
<option value="prod">Production (high availability, monitoring, DR)</option>
</options>
<impact>Affects SKU selection, instance counts, auto-scaling, monitoring depth</impact>
</question>

<question name="regions">
<prompt>What Azure region(s) do you need?</prompt>
<considerations>
- Primary region for resources
- Secondary region for disaster recovery (prod only)
- Data residency requirements
- Latency to users
</considerations>
<recommendation>Use East US or West US for lowest costs unless specific region needed</recommendation>
</question>

<question name="compliance">
<prompt>Any compliance or regulatory requirements?</prompt>
<examples>
- HIPAA (healthcare data)
- PCI-DSS (payment card data)
- GDPR (EU personal data)
- SOC 2
- Data residency rules
</examples>
<impact>Affects service choices, encryption requirements, audit logging, network isolation</impact>
</question>
</questions>

<instruction>
Wait for user responses to all questions before proceeding. Use their answers to tailor service recommendations, SKU sizing, and architecture design in subsequent steps.
</instruction>
</step>

<step number="2" name="choose_iac_tool">
<title>Choose Infrastructure as Code Tool</title>
<description>
Select between Bicep and Terraform based on project requirements, team expertise, multi-cloud needs, and infrastructure complexity. Both tools provide declarative infrastructure provisioning with version control and repeatability.
</description>

<tool_comparison>
<tool name="bicep">
<summary>Azure-native declarative language, compiles to ARM templates</summary>

<pros>
<pro>Native Azure tool - newest features available on day one</pro>
<pro>No state file management - Azure tracks resource state</pro>
<pro>Better error messages for Azure-specific issues</pro>
<pro>Simpler syntax for Azure resources (less verbose than ARM JSON)</pro>
<pro>Built into Azure CLI - no separate installation</pro>
<pro>Free Azure support included</pro>
<pro>Modules published to Azure Bicep Registry</pro>
<pro>Type safety with parameter validation</pro>
</pros>

<cons>
<con>Azure-only - cannot manage AWS, GCP, or other cloud providers</con>
<con>Smaller community compared to Terraform</con>
<con>Newer tool (released 2020) - fewer mature patterns available</con>
<con>Limited third-party tooling ecosystem</con>
</cons>

<use_when>
<scenario>Building Azure-only infrastructure</scenario>
<scenario>Team prefers native Azure tooling and Azure-first approach</scenario>
<scenario>Want simplest path for Azure resource provisioning</scenario>
<scenario>No multi-cloud requirements now or in future</scenario>
<scenario>Need newest Azure features immediately (preview features)</scenario>
<scenario>Want to avoid state file management complexity</scenario>
</use_when>

<learning_curve>Low - familiar ARM template concepts with cleaner syntax</learning_curve>
<reference>references/infrastructure-as-code.md (section: Bicep Deep Dive)</reference>
</tool>

<tool name="terraform">
<summary>Multi-cloud IaC tool by HashiCorp with provider ecosystem</summary>

<pros>
<pro>Multi-cloud support - manage Azure + AWS + GCP + 2000+ providers</pro>
<pro>Mature ecosystem with extensive community modules</pro>
<pro>Large community - more examples, patterns, troubleshooting help</pro>
<pro>Advanced state management with locking and remote backends</pro>
<pro>Drift detection shows infrastructure changes outside Terraform</pro>
<pro>Import existing resources into Terraform state</pro>
<pro>Rich provider ecosystem (Datadog, PagerDuty, GitHub, etc.)</pro>
<pro>Terraform Cloud for team collaboration</pro>
</pros>

<cons>
<con>State file management adds complexity and security concerns</con>
<con>Azure provider may lag behind newest Azure features (3-6 month delay)</con>
<con>Requires separate tool installation and maintenance</con>
<con>State file can become corrupted or out of sync</con>
<con>More verbose syntax for some Azure resources</con>
</cons>

<use_when>
<scenario>Multi-cloud or hybrid cloud environments</scenario>
<scenario>Need to manage non-Azure resources (AWS, Datadog, GitHub, DNS providers)</scenario>
<scenario>Require advanced state management and drift detection</scenario>
<scenario>Team has existing Terraform expertise and workflows</scenario>
<scenario>Need Terraform Cloud for team collaboration</scenario>
<scenario>Want access to large community module library</scenario>
</use_when>

<learning_curve>Medium - need to understand HCL syntax and state management</learning_curve>
<reference>references/infrastructure-as-code.md (section: Terraform Deep Dive)</reference>
</tool>
</tool_comparison>

<recommendation>
<default_choice>Bicep for Azure-only projects (simpler, native)</default_choice>
<alternative_choice>Terraform when multi-cloud or existing Terraform usage</alternative_choice>
</recommendation>

<instruction>
Ask user preference if not already specified. Default to Bicep for Azure-only projects unless team has strong Terraform background or multi-cloud requirements.
</instruction>
</step>

<step number="3" name="design_architecture">
<title>Design Architecture</title>
<description>
Based on application type, traffic requirements, and budget constraints, recommend appropriate Azure compute services, database options, storage solutions, and supporting services. Create a list of resources to provision.
</description>

<architecture_recommendations>
<category name="compute">
<title>Compute Services</title>

<service_option name="app_service">
<name>Azure App Service (Web App)</name>
<type>PaaS</type>
<best_for>
<use_case>Web applications (Node.js, .NET, Python, Java, PHP)</use_case>
<use_case>REST APIs</use_case>
<use_case>Small to medium scale (up to ~1000 requests/sec per instance)</use_case>
</best_for>
<pros>Easiest to manage, built-in scaling, deployment slots, automatic OS patching</pros>
<cons>Less flexibility than containers, limited customization</cons>
<pricing>$13/month (B1) to $292/month (P2v3) per instance</pricing>
<reference>references/compute-services.md</reference>
</service_option>

<service_option name="aks">
<name>Azure Kubernetes Service (AKS)</name>
<type>Container Orchestration</type>
<best_for>
<use_case>Microservices architectures</use_case>
<use_case>Large scale applications (1000+ requests/sec)</use_case>
<use_case>Multi-container applications</use_case>
<use_case>Need full control over runtime environment</use_case>
</best_for>
<pros>Full control, portable, industry standard, scales to thousands of pods</pros>
<cons>Complex to manage, requires Kubernetes expertise, higher operational overhead</cons>
<pricing>Free control plane + node VM costs ($200-1000+/month typical)</pricing>
<reference>references/kubernetes-aks.md</reference>
</service_option>

<service_option name="functions">
<name>Azure Functions</name>
<type>Serverless</type>
<best_for>
<use_case>Event-driven workloads</use_case>
<use_case>Background processing</use_case>
<use_case>Scheduled jobs</use_case>
<use_case>Low/variable traffic patterns</use_case>
</best_for>
<pros>Pay-per-execution, auto-scaling, no server management, consumption plan very cheap</pros>
<cons>Cold start latency, execution time limits (5-10 min), stateless</cons>
<pricing>$0.20 per million executions (Consumption) or $146/month+ (Premium)</pricing>
<reference>references/compute-services.md</reference>
</service_option>

<service_option name="container_apps">
<name>Azure Container Apps</name>
<type>Serverless Containers</type>
<best_for>
<use_case>Microservices without Kubernetes complexity</use_case>
<use_case>Event-driven container workloads</use_case>
<use_case>Need containers but not full Kubernetes</use_case>
</best_for>
<pros>Simpler than AKS, Dapr integration, KEDA-based scaling, consumption billing</pros>
<cons>Newer service, less control than AKS, some limitations vs full Kubernetes</cons>
<pricing>Pay for vCPU/memory consumed ($0.000012/vCPU-second)</pricing>
<reference>references/compute-services.md</reference>
</service_option>
</category>

<category name="databases">
<title>Database Services</title>

<service_option name="azure_sql">
<name>Azure SQL Database</name>
<type>Relational (SQL Server)</type>
<best_for>
<use_case>Relational data with ACID guarantees</use_case>
<use_case>Existing SQL Server applications</use_case>
<use_case>Complex queries and transactions</use_case>
</best_for>
<pricing_models>
<model name="dtu">DTU-based (simpler, fixed resources): $5/month (Basic) to $500+/month</model>
<model name="vcore">vCore-based (flexible, reserved capacity): Better for predictable workloads</model>
</pricing_models>
<features>Automatic backups, point-in-time restore, geo-replication, built-in intelligence</features>
<reference>references/storage-data.md</reference>
</service_option>

<service_option name="cosmos_db">
<name>Cosmos DB</name>
<type>NoSQL (multi-model)</type>
<best_for>
<use_case>Globally distributed applications</use_case>
<use_case>Document, key-value, graph, or columnar data</use_case>
<use_case>Need 99.999% availability SLA</use_case>
<use_case>Flexible schema requirements</use_case>
</best_for>
<pricing>$0.008/GB storage + $0.00008 per RU/s consumed (serverless) or reserved throughput</pricing>
<features>Multi-region replication, multiple consistency models, automatic indexing</features>
<reference>references/storage-data.md</reference>
</service_option>

<service_option name="redis">
<name>Azure Cache for Redis</name>
<type>In-memory cache</type>
<best_for>
<use_case>Session state storage</use_case>
<use_case>Caching frequently accessed data</use_case>
<use_case>Message broker (pub/sub)</use_case>
<use_case>Rate limiting</use_case>
</best_for>
<pricing>$16/month (Basic C0) to $1600+/month (Premium)</pricing>
<features>In-memory performance, persistence options, clustering (Premium), geo-replication</features>
<reference>references/storage-data.md</reference>
</service_option>
</category>

<category name="storage">
<title>Storage Services</title>

<service_option name="storage_account">
<name>Azure Storage Account</name>
<type>Object storage</type>
<services_included>
<service>Blob Storage (files, images, videos, backups)</service>
<service>File Storage (SMB file shares)</service>
<service>Queue Storage (message queues)</service>
<service>Table Storage (NoSQL key-value)</service>
</services_included>
<pricing>$0.018-0.15/GB per month depending on tier and redundancy</pricing>
<use_cases>File uploads, static assets, backups, data lakes, message queues</use_cases>
<reference>references/storage-data.md</reference>
</service_option>

<service_option name="service_bus">
<name>Azure Service Bus</name>
<type>Enterprise messaging</type>
<best_for>
<use_case>Reliable message delivery with guarantees</use_case>
<use_case>Complex routing (topics and subscriptions)</use_case>
<use_case>Transaction support</use_case>
<use_case>Dead letter queues</use_case>
</best_for>
<pricing>$10/month (Basic) to $670/month (Premium) + message charges</pricing>
<reference>references/architecture-patterns.md (section: Messaging Patterns)</reference>
</service_option>
</category>
</architecture_recommendations>

<architecture_output>
<instruction>
Based on user requirements from Step 1, create a list of resources to provision. Present this list to the user for approval before creating IaC templates.
</instruction>

<example_output>
<scenario>Web app with database and caching</scenario>
<resource_list>
- Resource Group: myapp-dev-eastus-rg
- App Service Plan: myapp-dev-eastus-asp-001 (B1 for dev)
- App Service: myapp-dev-eastus-app-001 (with Managed Identity)
- Azure SQL Database: myapp-dev-eastus-sql-001 (Basic tier for dev)
- Azure Cache for Redis: myapp-dev-eastus-redis-001 (C0 for dev)
- Storage Account: myappdeveustst001 (for file uploads)
- Application Insights: myapp-dev-eastus-insights-001
</resource_list>
<estimated_monthly_cost>~$45/month for dev environment</estimated_monthly_cost>
</example_output>

<instruction>Wait for user approval before proceeding to IaC template creation.</instruction>
</architecture_output>
</step>

<step number="4" name="setup_resource_organization">
<title>Setup Resource Organization</title>
<description>
Define naming convention following Azure best practices and create the resource group with proper tags for cost allocation, governance, and organization.
</description>

<naming_convention>
<pattern>&lt;product&gt;-&lt;env&gt;-&lt;region&gt;-&lt;resource-type&gt;-&lt;instance&gt;</pattern>
<description>Consistent naming enables easy identification, automation, and cost tracking</description>

<components>
<component name="product">Product or application name (lowercase, no special chars)</component>
<component name="env">Environment: dev, staging, prod</component>
<component name="region">Azure region short code: eastus, westus, westeurope</component>
<component name="resource_type">Service abbreviation: rg, app, sql, kv, st, etc.</component>
<component name="instance">Sequential number: 001, 002, etc.</component>
</components>

<examples>
<example>myapp-prod-eastus-app-001 (App Service in production)</example>
<example>myapp-prod-eastus-sql-001 (SQL Database in production)</example>
<example>myapp-dev-eastus-rg (Resource group for dev)</example>
<example>shared-prod-eastus-kv-001 (Shared Key Vault for multiple products)</example>
</examples>

<reference>references/resource-organization.md (comprehensive naming guide)</reference>
</naming_convention>

<tagging_strategy>
<description>
Tags enable cost allocation, automation, governance, and resource management. Apply consistently across all resources.
</description>

<required_tags>
<tag name="Environment">
<values>dev, staging, prod</values>
<purpose>Identify environment for cost reports and automation</purpose>
</tag>

<tag name="Product">
<example>myapp, shared-services, analytics</example>
<purpose>Allocate costs to specific products or projects</purpose>
</tag>

<tag name="CostCenter">
<example>engineering, marketing, operations</example>
<purpose>Charge costs to appropriate department or team</purpose>
</tag>

<tag name="Owner">
<example>team@company.com or username</example>
<purpose>Identify who to contact for questions or issues</purpose>
</tag>
</required_tags>

<optional_tags>
<tag name="ManagedBy">Bicep, Terraform, Manual (track how resources are managed)</tag>
<tag name="Criticality">Low, Medium, High, Mission-Critical</tag>
<tag name="DataClassification">Public, Internal, Confidential, Restricted</tag>
<tag name="MaintenanceWindow">Sat-0200-0600 (for patching schedules)</tag>
</optional_tags>

<reference>references/resource-organization.md (tagging best practices)</reference>
</tagging_strategy>

<resource_group_creation>
<title>Create Resource Group</title>

<command type="azure-cli">
<description>Create resource group with proper naming and tagging</description>
<syntax><![CDATA[
az group create \
  --name <product>-<env>-<region>-rg \
  --location <region> \
  --tags \
    Environment=<env> \
    Product=<product> \
    CostCenter=<cost-center> \
    Owner=<owner-email>
]]></syntax>

<example><![CDATA[
az group create \
  --name myapp-dev-eastus-rg \
  --location eastus \
  --tags \
    Environment=dev \
    Product=myapp \
    CostCenter=engineering \
    Owner=devteam@company.com
]]></example>
</command>

<note>
Replace placeholders with actual values from user requirements gathered in Step 1.
</note>

<verification>
<command>az group show --name myapp-dev-eastus-rg --query "{Name:name, Location:location, Tags:tags}"</command>
<expected_output>Resource group created with correct name, location, and all tags present</expected_output>
</verification>
</resource_group_creation>
</step>

<step number="5" name="create_iac_templates">
<title>Create Infrastructure as Code Templates</title>
<description>
Write IaC templates (Bicep or Terraform) that define all Azure resources declaratively. Templates should be parameterized for different environments, include security best practices by default, and output necessary values for downstream use.
</description>

<templates>
<bicep>
<title>Bicep Templates</title>
<description>Azure-native declarative syntax that compiles to ARM templates</description>

<file name="main.bicep">
<description>Main template defining all resources with parameters and outputs</description>
<code><![CDATA[
@description('Environment name')
@allowed(['dev', 'staging', 'prod'])
param environment string

@description('Location for all resources')
param location string = resourceGroup().location

@description('Product name for resource naming')
@minLength(3)
@maxLength(10)
param productName string

// Configuration based on environment
var config = {
  dev: {
    appServiceSku: 'B1'
    appServiceTier: 'Basic'
    appServiceInstances: 1
    sqlTier: 'Basic'
    sqlMaxSize: 2  // GB
  }
  staging: {
    appServiceSku: 'P1v3'
    appServiceTier: 'PremiumV3'
    appServiceInstances: 2
    sqlTier: 'S3'
    sqlMaxSize: 250  // GB
  }
  prod: {
    appServiceSku: 'P2v3'
    appServiceTier: 'PremiumV3'
    appServiceInstances: 3
    sqlTier: 'S3'
    sqlMaxSize: 250  // GB
  }
}

// Common tags for all resources
var commonTags = {
  Environment: environment
  Product: productName
  ManagedBy: 'Bicep'
  DeployedAt: utcNow('yyyy-MM-dd')
}

// App Service Plan (compute)
resource appServicePlan 'Microsoft.Web/serverfarms@2023-12-01' = {
  name: '${productName}-${environment}-${location}-asp-001'
  location: location
  sku: {
    name: config[environment].appServiceSku
    tier: config[environment].appServiceTier
    capacity: config[environment].appServiceInstances
  }
  kind: 'linux'
  properties: {
    reserved: true  // Required for Linux
    perSiteScaling: false
    zoneRedundant: environment == 'prod'  // High availability for prod
  }
  tags: commonTags
}

// App Service with security best practices
resource appService 'Microsoft.Web/sites@2023-12-01' = {
  name: '${productName}-${environment}-${location}-app-001'
  location: location
  identity: {
    type: 'SystemAssigned'  // Managed Identity for zero-credentials access
  }
  properties: {
    serverFarmId: appServicePlan.id
    httpsOnly: true  // Force HTTPS
    clientAffinityEnabled: false  // Better for stateless apps
    siteConfig: {
      linuxFxVersion: 'NODE|20-lts'  // Node.js 20 LTS runtime
      alwaysOn: environment != 'dev'  // Keep warm in staging/prod
      ftpsState: 'Disabled'  // Disable insecure FTP
      minTlsVersion: '1.2'  // Minimum TLS 1.2
      http20Enabled: true  // Enable HTTP/2
      healthCheckPath: '/health'  // Health endpoint for load balancer
      appSettings: [
        {
          name: 'ENVIRONMENT'
          value: environment
        }
        {
          name: 'WEBSITE_NODE_DEFAULT_VERSION'
          value: '~20'
        }
      ]
    }
  }
  tags: commonTags
}

// Application Insights for monitoring
resource appInsights 'Microsoft.Insights/components@2020-02-02' = {
  name: '${productName}-${environment}-${location}-insights-001'
  location: location
  kind: 'web'
  properties: {
    Application_Type: 'web'
    RetentionInDays: environment == 'prod' ? 90 : 30
    publicNetworkAccessForIngestion: 'Enabled'
    publicNetworkAccessForQuery: 'Enabled'
  }
  tags: commonTags
}

// Configure App Service to use Application Insights
resource appInsightsConnection 'Microsoft.Web/sites/config@2023-12-01' = {
  parent: appService
  name: 'appsettings'
  properties: {
    APPINSIGHTS_INSTRUMENTATIONKEY: appInsights.properties.InstrumentationKey
    APPLICATIONINSIGHTS_CONNECTION_STRING: appInsights.properties.ConnectionString
    ApplicationInsightsAgent_EXTENSION_VERSION: '~3'
  }
}

// Azure SQL Server (logical server)
resource sqlServer 'Microsoft.Sql/servers@2023-05-01-preview' = {
  name: '${productName}-${environment}-${location}-sqlsrv-001'
  location: location
  properties: {
    administratorLogin: 'sqladmin'
    administratorLoginPassword: 'REPLACE_IN_KEYVAULT'  // NOTE: Use Key Vault reference in production!
    version: '12.0'
    minimalTlsVersion: '1.2'
    publicNetworkAccess: 'Enabled'  // Use 'Disabled' with Private Endpoint in prod
  }
  tags: commonTags
}

// Azure SQL Database
resource sqlDatabase 'Microsoft.Sql/servers/databases@2023-05-01-preview' = {
  parent: sqlServer
  name: '${productName}-${environment}-db'
  location: location
  sku: {
    name: config[environment].sqlTier
  }
  properties: {
    maxSizeBytes: config[environment].sqlMaxSize * 1073741824  // Convert GB to bytes
    collation: 'SQL_Latin1_General_CP1_CI_AS'
    catalogCollation: 'SQL_Latin1_General_CP1_CI_AS'
    zoneRedundant: environment == 'prod'  // High availability for prod
    readScale: environment == 'prod' ? 'Enabled' : 'Disabled'
    requestedBackupStorageRedundancy: environment == 'prod' ? 'Geo' : 'Local'
  }
  tags: commonTags
}

// Grant App Service Managed Identity access to SQL Database
// NOTE: This creates RBAC assignment, still need to run SQL to create database user
resource sqlRoleAssignment 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  name: guid(sqlServer.id, appService.id, 'sql-contributor')
  scope: sqlServer
  properties: {
    roleDefinitionId: subscriptionResourceId('Microsoft.Authorization/roleDefinitions', '6d8ee4ec-f05a-4a1d-8b00-a9b17e38b437')  // SQL DB Contributor
    principalId: appService.identity.principalId
    principalType: 'ServicePrincipal'
  }
}

// Storage Account for file uploads, logs, etc.
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: '${toLower(replace(productName, '-', ''))}${environment}${toLower(replace(location, '-', ''))}st001'  // Storage names can't have hyphens
  location: location
  kind: 'StorageV2'
  sku: {
    name: environment == 'prod' ? 'Standard_GRS' : 'Standard_LRS'  // Geo-redundant for prod
  }
  properties: {
    accessTier: 'Hot'
    supportsHttpsTrafficOnly: true  // HTTPS only
    minimumTlsVersion: 'TLS1_2'
    allowBlobPublicAccess: false  // Security: no anonymous access
    networkAcls: {
      defaultAction: 'Allow'  // Use 'Deny' with Private Endpoint in prod
      bypass: 'AzureServices'
    }
  }
  tags: commonTags
}

// Outputs for use in CI/CD pipelines and dependent resources
output appServiceId string = appService.id
output appServiceName string = appService.name
output appServiceUrl string = 'https://${appService.properties.defaultHostName}'
output appServiceIdentityPrincipalId string = appService.identity.principalId
output appInsightsInstrumentationKey string = appInsights.properties.InstrumentationKey
output appInsightsConnectionString string = appInsights.properties.ConnectionString
output sqlServerFqdn string = sqlServer.properties.fullyQualifiedDomainName
output sqlDatabaseName string = sqlDatabase.name
output storageAccountName string = storageAccount.name
output storageAccountPrimaryEndpoint string = storageAccount.properties.primaryEndpoints.blob
]]></code>
</file>

<file name="main.parameters.dev.json">
<description>Development environment parameters</description>
<code><![CDATA[
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "environment": {
      "value": "dev"
    },
    "productName": {
      "value": "myapp"
    },
    "location": {
      "value": "eastus"
    }
  }
}
]]></code>
</file>

<file name="main.parameters.prod.json">
<description>Production environment parameters</description>
<code><![CDATA[
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "environment": {
      "value": "prod"
    },
    "productName": {
      "value": "myapp"
    },
    "location": {
      "value": "eastus"
    }
  }
}
]]></code>
</file>

<deployment_instructions>
<title>Deploy with Bicep</title>

<step name="validate">
<description>Validate template syntax before deployment</description>
<command>az bicep build --file main.bicep</command>
<expected_output>No errors, template compiles successfully</expected_output>
</step>

<step name="what_if">
<description>Preview what changes will be made (optional but recommended)</description>
<command><![CDATA[
az deployment group what-if \
  --resource-group myapp-dev-eastus-rg \
  --template-file main.bicep \
  --parameters main.parameters.dev.json
]]></command>
<expected_output>Shows resources that will be created</expected_output>
</step>

<step name="deploy">
<description>Deploy infrastructure to Azure</description>
<command><![CDATA[
az deployment group create \
  --name "deployment-$(date +%Y%m%d-%H%M%S)" \
  --resource-group myapp-dev-eastus-rg \
  --template-file main.bicep \
  --parameters main.parameters.dev.json
]]></command>
<expected_output>Deployment succeeds, outputs are displayed</expected_output>
</step>
</deployment_instructions>
</bicep>

<terraform>
<title>Terraform Configuration</title>
<description>HashiCorp Configuration Language (HCL) for multi-cloud infrastructure</description>

<file name="main.tf">
<description>Main Terraform configuration with provider, resources, and outputs</description>
<code><![CDATA[
terraform {
  required_version = ">= 1.0"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }

  # Backend for remote state (configure separately)
  backend "azurerm" {
    # Configure via backend-config file or environment variables
  }
}

provider "azurerm" {
  features {
    resource_group {
      prevent_deletion_if_contains_resources = true
    }
  }
}

# Variables
variable "environment" {
  type        = string
  description = "Environment name (dev, staging, prod)"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod"
  }
}

variable "product_name" {
  type        = string
  description = "Product name for resource naming (lowercase, no special chars)"

  validation {
    condition     = can(regex("^[a-z0-9-]{3,10}$", var.product_name))
    error_message = "Product name must be 3-10 chars, lowercase alphanumeric and hyphens only"
  }
}

variable "location" {
  type        = string
  default     = "eastus"
  description = "Azure region for resources"
}

# Local values for environment-specific configuration
locals {
  config = {
    dev = {
      app_service_sku      = "B1"
      app_service_instances = 1
      sql_tier             = "Basic"
      sql_max_size_gb      = 2
      storage_replication  = "LRS"
    }
    staging = {
      app_service_sku      = "P1v3"
      app_service_instances = 2
      sql_tier             = "S3"
      sql_max_size_gb      = 250
      storage_replication  = "LRS"
    }
    prod = {
      app_service_sku      = "P2v3"
      app_service_instances = 3
      sql_tier             = "S3"
      sql_max_size_gb      = 250
      storage_replication  = "GRS"
    }
  }

  # Common tags
  tags = {
    Environment = var.environment
    Product     = var.product_name
    ManagedBy   = "Terraform"
  }
}

# Resource Group (already exists from Step 4, but importing)
data "azurerm_resource_group" "main" {
  name = "${var.product_name}-${var.environment}-${var.location}-rg"
}

# App Service Plan
resource "azurerm_service_plan" "main" {
  name                = "${var.product_name}-${var.environment}-${var.location}-asp-001"
  location            = data.azurerm_resource_group.main.location
  resource_group_name = data.azurerm_resource_group.main.name
  os_type             = "Linux"
  sku_name            = local.config[var.environment].app_service_sku
  worker_count        = local.config[var.environment].app_service_instances
  zone_balancing_enabled = var.environment == "prod"

  tags = local.tags
}

# App Service (Linux Web App)
resource "azurerm_linux_web_app" "main" {
  name                = "${var.product_name}-${var.environment}-${var.location}-app-001"
  location            = data.azurerm_resource_group.main.location
  resource_group_name = data.azurerm_resource_group.main.name
  service_plan_id     = azurerm_service_plan.main.id
  https_only          = true

  identity {
    type = "SystemAssigned"
  }

  site_config {
    always_on         = var.environment != "dev"
    health_check_path = "/health"
    ftps_state        = "Disabled"
    minimum_tls_version = "1.2"
    http2_enabled       = true

    application_stack {
      node_version = "20-lts"
    }
  }

  app_settings = {
    "ENVIRONMENT"                      = var.environment
    "WEBSITE_NODE_DEFAULT_VERSION"     = "~20"
  }

  tags = local.tags
}

# Application Insights
resource "azurerm_application_insights" "main" {
  name                = "${var.product_name}-${var.environment}-${var.location}-insights-001"
  location            = data.azurerm_resource_group.main.location
  resource_group_name = data.azurerm_resource_group.main.name
  application_type    = "web"
  retention_in_days   = var.environment == "prod" ? 90 : 30

  tags = local.tags
}

# Connect App Service to Application Insights
resource "azurerm_linux_web_app_slot" "appinsights_config" {
  name           = "production"
  app_service_id = azurerm_linux_web_app.main.id

  site_config {
    application_insights_key               = azurerm_application_insights.main.instrumentation_key
    application_insights_connection_string = azurerm_application_insights.main.connection_string
  }
}

# SQL Server
resource "azurerm_mssql_server" "main" {
  name                         = "${var.product_name}-${var.environment}-${var.location}-sqlsrv-001"
  location                     = data.azurerm_resource_group.main.location
  resource_group_name          = data.azurerm_resource_group.main.name
  version                      = "12.0"
  administrator_login          = "sqladmin"
  administrator_login_password = "REPLACE_WITH_KEYVAULT_REFERENCE"  # Use Key Vault in production!
  minimum_tls_version          = "1.2"
  public_network_access_enabled = true  # Use false with Private Endpoint in prod

  azuread_administrator {
    login_username = "AzureAD Admin"
    object_id      = azurerm_linux_web_app.main.identity[0].principal_id
  }

  tags = local.tags
}

# SQL Database
resource "azurerm_mssql_database" "main" {
  name      = "${var.product_name}-${var.environment}-db"
  server_id = azurerm_mssql_server.main.id
  sku_name  = local.config[var.environment].sql_tier
  max_size_gb = local.config[var.environment].sql_max_size_gb
  zone_redundant = var.environment == "prod"

  tags = local.tags
}

# Storage Account
resource "azurerm_storage_account" "main" {
  name                     = "${replace(var.product_name, "-", "")}${var.environment}${replace(var.location, "-", "")}st001"
  location                 = data.azurerm_resource_group.main.location
  resource_group_name      = data.azurerm_resource_group.main.name
  account_tier             = "Standard"
  account_replication_type = local.config[var.environment].storage_replication
  min_tls_version          = "TLS1_2"
  enable_https_traffic_only = true
  allow_nested_items_to_be_public = false

  tags = local.tags
}

# Outputs
output "app_service_id" {
  value       = azurerm_linux_web_app.main.id
  description = "App Service resource ID"
}

output "app_service_name" {
  value       = azurerm_linux_web_app.main.name
  description = "App Service name"
}

output "app_service_url" {
  value       = "https://${azurerm_linux_web_app.main.default_hostname}"
  description = "App Service default URL"
}

output "app_service_identity_principal_id" {
  value       = azurerm_linux_web_app.main.identity[0].principal_id
  description = "Managed Identity principal ID for RBAC assignments"
}

output "app_insights_instrumentation_key" {
  value       = azurerm_application_insights.main.instrumentation_key
  description = "Application Insights instrumentation key"
  sensitive   = true
}

output "app_insights_connection_string" {
  value       = azurerm_application_insights.main.connection_string
  description = "Application Insights connection string"
  sensitive   = true
}

output "sql_server_fqdn" {
  value       = azurerm_mssql_server.main.fully_qualified_domain_name
  description = "SQL Server fully qualified domain name"
}

output "sql_database_name" {
  value       = azurerm_mssql_database.main.name
  description = "SQL Database name"
}

output "storage_account_name" {
  value       = azurerm_storage_account.main.name
  description = "Storage Account name"
}

output "storage_account_primary_blob_endpoint" {
  value       = azurerm_storage_account.main.primary_blob_endpoint
  description = "Storage Account primary blob endpoint"
}
]]></code>
</file>

<file name="terraform.tfvars">
<description>Variable values for development environment</description>
<code><![CDATA[
environment  = "dev"
product_name = "myapp"
location     = "eastus"
]]></code>
</file>

<file name="terraform.prod.tfvars">
<description>Variable values for production environment</description>
<code><![CDATA[
environment  = "prod"
product_name = "myapp"
location     = "eastus"
]]></code>
</file>

<deployment_instructions>
<title>Deploy with Terraform</title>

<step name="init">
<description>Initialize Terraform working directory and download providers</description>
<command>terraform init</command>
<expected_output>Terraform initializes, downloads azurerm provider</expected_output>
</step>

<step name="format">
<description>Format code to canonical style (optional)</description>
<command>terraform fmt</command>
<expected_output>Files formatted</expected_output>
</step>

<step name="validate">
<description>Validate configuration syntax</description>
<command>terraform validate</command>
<expected_output>Success! The configuration is valid</expected_output>
</step>

<step name="plan">
<description>Preview what changes will be made</description>
<command>terraform plan -var-file="terraform.tfvars" -out=tfplan</command>
<expected_output>Plan shows resources to be created, saved to tfplan file</expected_output>
</step>

<step name="apply">
<description>Apply the planned changes to create infrastructure</description>
<command>terraform apply tfplan</command>
<expected_output>Resources created successfully, outputs displayed</expected_output>
</step>
</deployment_instructions>
</terraform>
</templates>

<security_note>
<warning>SQL Server Administrator Password</warning>
<description>
The templates above use placeholder passwords. In production, NEVER hardcode passwords. Use Azure Key Vault references instead:
</description>

<bicep_solution>
<code><![CDATA[
resource keyVault 'Microsoft.KeyVault/vaults@2023-02-01' existing = {
  name: 'shared-prod-eastus-kv-001'
}

resource sqlServer 'Microsoft.Sql/servers@2023-05-01-preview' = {
  properties: {
    administratorLogin: 'sqladmin'
    administratorLoginPassword: keyVault.getSecret('sql-admin-password')
  }
}
]]></code>
</bicep_solution>

<terraform_solution>
<code><![CDATA[
data "azurerm_key_vault_secret" "sql_password" {
  name         = "sql-admin-password"
  key_vault_id = data.azurerm_key_vault.shared.id
}

resource "azurerm_mssql_server" "main" {
  administrator_login_password = data.azurerm_key_vault_secret.sql_password.value
}
]]></code>
</terraform_solution>

<reference>references/security-identity.md (Key Vault integration patterns)</reference>
</security_note>
</step>

<step number="6" name="validate_templates">
<title>Validate Templates</title>
<description>
Before deploying to Azure, validate that IaC templates are syntactically correct and will deploy successfully. This catches errors early before making changes to production infrastructure.
</description>

<validation>
<bicep_validation>
<title>Bicep Validation</title>

<command name="build">
<description>Compile Bicep to ARM JSON and check for syntax errors</description>
<syntax>az bicep build --file main.bicep</syntax>
<expected_output>Compiles without errors, creates main.json</expected_output>
<common_errors>
<error>Missing required parameter</error>
<error>Invalid resource type or API version</error>
<error>Circular dependency between resources</error>
</common_errors>
</command>

<command name="lint">
<description>Check for code quality issues and best practices</description>
<syntax>az bicep lint --file main.bicep</syntax>
<expected_output>No warnings or errors (or acceptable warnings only)</expected_output>
</command>

<command name="what_if">
<description>Preview what changes will be made without deploying</description>
<syntax><![CDATA[
az deployment group what-if \
  --resource-group myapp-dev-eastus-rg \
  --template-file main.bicep \
  --parameters main.parameters.dev.json
]]></syntax>
<expected_output>Shows resource changes (Create, Modify, Delete, No Change)</expected_output>
<purpose>Catch unexpected changes before deployment</purpose>
</command>
</bicep_validation>

<terraform_validation>
<title>Terraform Validation</title>

<command name="init">
<description>Initialize and download required providers</description>
<syntax>terraform init</syntax>
<expected_output>Terraform initialized successfully</expected_output>
</command>

<command name="fmt_check">
<description>Check if files are properly formatted</description>
<syntax>terraform fmt -check</syntax>
<expected_output>No output means files are formatted correctly</expected_output>
</command>

<command name="validate">
<description>Validate configuration syntax and internal consistency</description>
<syntax>terraform validate</syntax>
<expected_output>Success! The configuration is valid.</expected_output>
<common_errors>
<error>Missing required argument</error>
<error>Invalid resource reference</error>
<error>Type mismatch in variables</error>
</common_errors>
</command>

<command name="plan">
<description>Create execution plan showing what will be created/modified/destroyed</description>
<syntax>terraform plan -var-file="terraform.tfvars"</syntax>
<expected_output>Plan shows resource changes with details</expected_output>
<review_checklist>
<item>Verify correct number of resources will be created</item>
<item>Check SKU sizes match expected environment (dev/staging/prod)</item>
<item>Confirm no unexpected deletions or modifications</item>
<item>Review security settings (HTTPS, TLS versions, public access)</item>
</review_checklist>
</command>
</terraform_validation>
</validation>

<instruction>
Review validation output with user before proceeding to deployment. Explain any warnings or unexpected changes. Get user approval to proceed.
</instruction>
</step>

<step number="7" name="deploy_infrastructure">
<title>Deploy Infrastructure</title>
<description>
Execute the deployment to create all resources in Azure. Use timestamped deployment names for tracking and rollback capabilities.
</description>

<deployment>
<bicep_deployment>
<title>Deploy with Bicep</title>

<command>
<syntax><![CDATA[
az deployment group create \
  --name "deployment-$(date +%Y%m%d-%H%M%S)" \
  --resource-group myapp-dev-eastus-rg \
  --template-file main.bicep \
  --parameters main.parameters.dev.json \
  --verbose
]]></syntax>

<explanation>
- --name with timestamp allows tracking multiple deployments
- --verbose shows detailed progress during deployment
- Deployment typically takes 3-10 minutes depending on resources
</explanation>

<expected_output>
Deployment succeeds with "provisioningState": "Succeeded"
All outputs are displayed (URLs, resource IDs, etc.)
</expected_output>
</command>

<monitoring_deployment>
<description>Monitor deployment progress in real-time (optional, open in another terminal)</description>
<command><![CDATA[
watch -n 5 'az deployment group list \
  --resource-group myapp-dev-eastus-rg \
  --query "[0].{Name:name, State:properties.provisioningState, Time:properties.timestamp}" \
  --output table'
]]></command>
</monitoring_deployment>
</bicep_deployment>

<terraform_deployment>
<title>Deploy with Terraform</title>

<command name="apply">
<syntax>terraform apply -var-file="terraform.tfvars"</syntax>
<interactive>Terraform will prompt "Do you want to perform these actions?" - type "yes" to proceed</interactive>
<expected_output>Apply complete! Resources: X added, 0 changed, 0 destroyed.</expected_output>
</command>

<command name="apply_auto_approve">
<description>Deploy without interactive approval (use in CI/CD pipelines)</description>
<syntax>terraform apply -var-file="terraform.tfvars" -auto-approve</syntax>
<warning>Use with caution - no confirmation prompt before making changes</warning>
</command>

<command name="show_state">
<description>View current Terraform state after deployment</description>
<syntax>terraform show</syntax>
<expected_output>Shows all created resources with their current configuration</expected_output>
</command>
</terraform_deployment>
</deployment>

<troubleshooting>
<common_issues>
<issue name="quota_exceeded">
<symptom>Deployment fails with "Quota exceeded" error</symptom>
<cause>Subscription has reached vCPU or resource quota limits</cause>
<solution>Request quota increase via Azure Portal → Quotas, or use different region/smaller SKU</solution>
<reference>workflows/debug-deployment.md</reference>
</issue>

<issue name="name_conflict">
<symptom>Deployment fails with "Name already exists" error</symptom>
<cause>Resource name conflicts with existing resource (especially globally unique names like storage accounts)</cause>
<solution>Change product name parameter or add instance number suffix</solution>
</issue>

<issue name="rbac_permission_denied">
<symptom>Deployment fails with "Authorization failed" or "Forbidden"</symptom>
<cause>Your Azure account lacks necessary permissions (need Contributor role)</cause>
<solution>Have subscription admin grant you Contributor role on resource group or subscription</solution>
</issue>
</common_issues>
</troubleshooting>
</step>

<step number="8" name="verify_deployment">
<title>Verify Deployment</title>
<description>
Run verification commands to ensure all resources were created successfully, are in the expected state, and are accessible. Catch any deployment issues before declaring success.
</description>

<verification_commands>
<check name="deployment_status">
<description>Verify deployment completed successfully</description>
<command><![CDATA[
az deployment group show \
  --name <deployment-name> \
  --resource-group myapp-dev-eastus-rg \
  --query "{Name:name, State:properties.provisioningState, Duration:properties.duration, Time:properties.timestamp}"
]]></command>
<expected_result>
<field name="State">Succeeded</field>
<field name="Duration">Should show completion time (typically 3-10 minutes)</field>
</expected_result>
</check>

<check name="resource_list">
<description>List all created resources in resource group</description>
<command>az resource list --resource-group myapp-dev-eastus-rg --output table</command>
<expected_result>
All expected resources present:
- App Service Plan
- App Service (Web App)
- SQL Server
- SQL Database
- Storage Account
- Application Insights
</expected_result>
</check>

<check name="app_service_status">
<description>Verify App Service is running</description>
<command><![CDATA[
az webapp show \
  --name myapp-dev-eastus-app-001 \
  --resource-group myapp-dev-eastus-rg \
  --query "{Name:name, State:state, DefaultHostName:defaultHostName, HttpsOnly:httpsOnly, ManagedIdentity:identity.type}"
]]></command>
<expected_result>
<field name="State">"Running"</field>
<field name="HttpsOnly">true</field>
<field name="ManagedIdentity">"SystemAssigned"</field>
</expected_result>
</check>

<check name="test_endpoint">
<description>Test App Service endpoint accessibility (may return 404 if no code deployed yet)</description>
<command>curl -I https://myapp-dev-eastus-app-001.azurewebsites.net</command>
<expected_result>
HTTP/2 200 or 404 (not 502/503 which indicate service issues)
HTTPS is enforced (not HTTP)
</expected_result>
<note>404 is acceptable at this stage - means infrastructure is working but no application code deployed yet</note>
</check>

<check name="sql_connectivity">
<description>Verify SQL Server is accessible (requires firewall rule for your IP)</description>
<command><![CDATA[
# Add your IP to SQL Server firewall
MY_IP=$(curl -s https://api.ipify.org)
az sql server firewall-rule create \
  --resource-group myapp-dev-eastus-rg \
  --server myapp-dev-eastus-sqlsrv-001 \
  --name AllowMyIP \
  --start-ip-address $MY_IP \
  --end-ip-address $MY_IP

# Test connection
az sql db show-connection-string \
  --client ado.net \
  --name myapp-dev-db \
  --server myapp-dev-eastus-sqlsrv-001
]]></command>
<expected_result>Connection string displayed, no authentication errors</expected_result>
</check>

<check name="storage_access">
<description>Verify Storage Account accessibility</description>
<command><![CDATA[
az storage container list \
  --account-name myappdeveustst001 \
  --auth-mode login
]]></command>
<expected_result>Empty list [] or existing containers, no access denied errors</expected_result>
</check>

<check name="app_insights_configured">
<description>Verify Application Insights is collecting data</description>
<command><![CDATA[
az monitor app-insights component show \
  --app myapp-dev-eastus-insights-001 \
  --resource-group myapp-dev-eastus-rg \
  --query "{Name:name, InstrumentationKey:instrumentationKey, ConnectionString:connectionString}"
]]></command>
<expected_result>Instrumentation key and connection string populated</expected_result>
<note>Actual telemetry data will appear after application code is deployed and generates traffic</note>
</check>
</verification_commands>

<outputs_review>
<description>Review deployment outputs for use in subsequent steps</description>
<bicep_outputs>
<command>az deployment group show --name &lt;deployment-name&gt; --resource-group myapp-dev-eastus-rg --query "properties.outputs"</command>
<expected_outputs>
<output name="appServiceUrl">https://myapp-dev-eastus-app-001.azurewebsites.net</output>
<output name="appServiceIdentityPrincipalId">GUID for Managed Identity</output>
<output name="sqlServerFqdn">myapp-dev-eastus-sqlsrv-001.database.windows.net</output>
<output name="storageAccountName">myappdeveustst001</output>
</expected_outputs>
</bicep_outputs>

<terraform_outputs>
<command>terraform output</command>
<expected_outputs>Same as Bicep outputs above</expected_outputs>
</terraform_outputs>

<usage>Save these outputs - you'll need them for deploying application code and configuring CI/CD pipelines</usage>
</outputs_review>
</step>

<step number="9" name="configure_post_deployment">
<title>Configure Post-Deployment Settings</title>
<description>
Configure additional settings that can't be fully managed in IaC templates, such as connection strings using Managed Identity, firewall rules for development access, and integration between services.
</description>

<configurations>
<sql_managed_identity_connection>
<title>Configure SQL Database Connection with Managed Identity</title>
<description>
Enable App Service to connect to SQL Database using its Managed Identity instead of connection strings with passwords (zero-credentials architecture).
</description>

<steps>
<step>
<action>Allow Azure services through SQL Server firewall</action>
<command><![CDATA[
az sql server firewall-rule create \
  --resource-group myapp-dev-eastus-rg \
  --server myapp-dev-eastus-sqlsrv-001 \
  --name AllowAzureServices \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0
]]></command>
</step>

<step>
<action>Get App Service Managed Identity principal ID</action>
<command><![CDATA[
PRINCIPAL_ID=$(az webapp identity show \
  --name myapp-dev-eastus-app-001 \
  --resource-group myapp-dev-eastus-rg \
  --query principalId --output tsv)

echo "Managed Identity Principal ID: $PRINCIPAL_ID"
]]></command>
</step>

<step>
<action>Create SQL database user for Managed Identity (run in SQL Management Studio or Azure Data Studio)</action>
<sql_script><![CDATA[
-- Connect to your database (not master)
CREATE USER [myapp-dev-eastus-app-001] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [myapp-dev-eastus-app-001];
ALTER ROLE db_datawriter ADD MEMBER [myapp-dev-eastus-app-001];
ALTER ROLE db_ddladmin ADD MEMBER [myapp-dev-eastus-app-001];  -- For Entity Framework migrations
]]></sql_script>
<note>Use your App Service name as the username</note>
</step>

<step>
<action>Configure App Service connection string to use Managed Identity</action>
<command><![CDATA[
az webapp config connection-string set \
  --name myapp-dev-eastus-app-001 \
  --resource-group myapp-dev-eastus-rg \
  --connection-string-type SQLAzure \
  --settings DefaultConnection="Server=myapp-dev-eastus-sqlsrv-001.database.windows.net;Database=myapp-dev-db;Authentication=Active Directory Default;"
]]></command>
<note>No password in connection string - authentication uses Managed Identity automatically</note>
</step>
</steps>

<reference>references/security-identity.md (Managed Identity patterns)</reference>
</sql_managed_identity_connection>

<storage_managed_identity_access>
<title>Grant App Service Access to Storage Account</title>
<description>Allow App Service to read/write blobs using its Managed Identity</description>

<command><![CDATA[
# Grant Storage Blob Data Contributor role
az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role "Storage Blob Data Contributor" \
  --scope /subscriptions/<subscription-id>/resourceGroups/myapp-dev-eastus-rg/providers/Microsoft.Storage/storageAccounts/myappdeveustst001
]]></command>

<verification>
<command>az role assignment list --assignee $PRINCIPAL_ID --output table</command>
<expected_result>Shows "Storage Blob Data Contributor" role on storage account</expected_result>
</verification>
</storage_managed_identity_access>

<local_development_access>
<title>Enable Local Development Access (Development Only)</title>
<description>
For development environment, add firewall rules to allow developers to access SQL Server and Storage from their local machines.
</description>

<warning>Only do this for dev environment. Production should use Private Endpoints and VNet integration.</warning>

<command><![CDATA[
# Add your current IP to SQL Server firewall
MY_IP=$(curl -s https://api.ipify.org)
az sql server firewall-rule create \
  --resource-group myapp-dev-eastus-rg \
  --server myapp-dev-eastus-sqlsrv-001 \
  --name AllowMyIP \
  --start-ip-address $MY_IP \
  --end-ip-address $MY_IP

# For Storage, public access is already enabled in dev (networkAcls.defaultAction: Allow)
]]></command>
</local_development_access>
</configurations>
</step>

<step number="10" name="document_and_save">
<title>Document and Save Infrastructure</title>
<description>
Create comprehensive documentation of the deployed infrastructure and commit all IaC templates to version control. This ensures infrastructure can be replicated, teammates can understand the setup, and changes are tracked over time.
</description>

<documentation>
<readme_file>
<title>Create README.md in IaC Repository</title>
<file name="README.md">
<content><![CDATA[
# MyApp Infrastructure

Infrastructure as Code for MyApp using [Bicep/Terraform].

## Resources Deployed

### Compute
- **Resource Group**: myapp-dev-eastus-rg
- **App Service Plan**: myapp-dev-eastus-asp-001 (B1 - Basic tier)
- **App Service**: myapp-dev-eastus-app-001
  - Runtime: Node.js 20 LTS
  - HTTPS only: Enabled
  - Managed Identity: Enabled

### Data
- **SQL Server**: myapp-dev-eastus-sqlsrv-001
- **SQL Database**: myapp-dev-db (Basic tier, 2 GB)
- **Storage Account**: myappdeveustst001 (LRS replication)

### Monitoring
- **Application Insights**: myapp-dev-eastus-insights-001

## Deployment

### Prerequisites
- Azure CLI authenticated (`az login`)
- [For Bicep: Azure CLI includes Bicep]
- [For Terraform: Terraform CLI installed]
- Contributor role on subscription

### Deploy to Development

#### Using Bicep
```bash
az group create \
  --name myapp-dev-eastus-rg \
  --location eastus

az deployment group create \
  --name "deployment-$(date +%Y%m%d-%H%M%S)" \
  --resource-group myapp-dev-eastus-rg \
  --template-file main.bicep \
  --parameters main.parameters.dev.json
```

#### Using Terraform
```bash
terraform init
terraform plan -var-file="terraform.tfvars"
terraform apply -var-file="terraform.tfvars"
```

## Outputs

After deployment, these values are available:

- **App URL**: https://myapp-dev-eastus-app-001.azurewebsites.net
- **SQL Server**: myapp-dev-eastus-sqlsrv-001.database.windows.net
- **Managed Identity Principal ID**: [output from deployment]
- **Application Insights Key**: [output from deployment]

## Post-Deployment Configuration

### SQL Database Access with Managed Identity

```sql
-- Run in SQL Database (not master):
CREATE USER [myapp-dev-eastus-app-001] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [myapp-dev-eastus-app-001];
ALTER ROLE db_datawriter ADD MEMBER [myapp-dev-eastus-app-001];
```

### Storage Access with Managed Identity

```bash
az role assignment create \
  --assignee <principal-id> \
  --role "Storage Blob Data Contributor" \
  --scope <storage-account-resource-id>
```

## Environment-Specific Configuration

| Environment | App Service SKU | SQL Tier | Storage Replication | Monthly Cost (est.) |
|-------------|----------------|----------|---------------------|---------------------|
| dev         | B1             | Basic    | LRS                 | ~$45                |
| staging     | P1v3           | S3       | LRS                 | ~$300               |
| prod        | P2v3           | S3       | GRS                 | ~$650               |

## Troubleshooting

### Deployment Failed
```bash
# Check deployment errors
az deployment group show \
  --name <deployment-name> \
  --resource-group myapp-dev-eastus-rg \
  --query "properties.error"
```

### App Service Not Responding
```bash
# Check app status
az webapp show \
  --name myapp-dev-eastus-app-001 \
  --resource-group myapp-dev-eastus-rg \
  --query "state"

# View logs
az webapp log tail \
  --name myapp-dev-eastus-app-001 \
  --resource-group myapp-dev-eastus-rg
```

## Next Steps

1. **Deploy Application Code**: See workflows/deploy-application.md
2. **Setup CI/CD Pipeline**: See workflows/setup-cicd-pipeline.md
3. **Configure Monitoring**: See workflows/setup-monitoring.md
4. **Implement Security**: See workflows/secure-infrastructure.md

## References

- [Infrastructure as Code Guide](../references/infrastructure-as-code.md)
- [Security & Identity Patterns](../references/security-identity.md)
- [Cost Optimization](../references/cost-optimization.md)

## Support

For issues or questions:
- Email: devteam@company.com
- Slack: #infrastructure
]]></content>
</file>
</readme_file>

<architecture_diagram>
<title>Create Architecture Diagram (Optional)</title>
<description>
Use tools like draw.io, Lucidchart, or Azure Architecture Diagrams to visualize resource relationships.
</description>
<components_to_show>
- Resource Group boundary
- Compute resources (App Service Plan, App Service)
- Data resources (SQL Server/Database, Storage Account)
- Monitoring (Application Insights)
- Network flow (Internet → App Service → SQL Database)
- Managed Identity connections (dotted lines showing MI authentication)
</components_to_show>
</architecture_diagram>
</documentation>

<version_control>
<title>Commit to Git</title>
<description>
Commit all IaC templates, parameter files, and documentation to version control. This creates an audit trail and enables team collaboration.
</description>

<files_to_commit>
<file>main.bicep (or main.tf for Terraform)</file>
<file>main.parameters.dev.json</file>
<file>main.parameters.staging.json</file>
<file>main.parameters.prod.json</file>
<file>terraform.tfvars (for Terraform)</file>
<file>README.md</file>
<file>.gitignore (exclude secrets, tfstate files)</file>
</files_to_commit>

<gitignore_content>
<description>Create .gitignore to exclude sensitive and generated files</description>
<content><![CDATA[
# Terraform
*.tfstate
*.tfstate.backup
.terraform/
*.tfvars.secrets
terraform.tfplan

# Bicep
*.json (compiled ARM templates)

# Secrets
*.key
*.pem
*.secrets
secrets/

# IDE
.vscode/
.idea/
*.swp

# OS
.DS_Store
Thumbs.db
]]></content>
</gitignore_content>

<git_commands>
<command><![CDATA[
# Initialize repo if needed
git init

# Add files
git add main.bicep main.parameters.*.json README.md .gitignore
# OR for Terraform
git add main.tf *.tfvars README.md .gitignore

# Commit
git commit -m "Initial infrastructure provisioning

- Added Bicep/Terraform templates for dev environment
- Resources: App Service, SQL Database, Storage Account, Application Insights
- Configured Managed Identity, HTTPS only, proper SKU sizing
- Documented deployment process and post-deployment config

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"

# Push to remote
git push origin main
]]></command>
</git_commands>
</version_control>

<cost_tracking_setup>
<title>Setup Cost Tracking (Optional)</title>
<description>
Configure Azure Cost Management budget alerts to track spending for this resource group.
</description>

<command><![CDATA[
# Create budget alert for resource group
az consumption budget create \
  --budget-name myapp-dev-budget \
  --category Cost \
  --amount 100 \
  --time-grain Monthly \
  --start-date $(date +%Y-%m-01) \
  --resource-group myapp-dev-eastus-rg \
  --meter-filter "Microsoft.Compute" "Microsoft.Sql" "Microsoft.Web" "Microsoft.Storage"
]]></command>

<notification_setup>
Configure email alerts when spending reaches 50%, 80%, 100% of budget via Azure Portal → Cost Management → Budgets.
</notification_setup>
</cost_tracking_setup>
</step>
</process>

<anti_patterns>
<title>Common Anti-Patterns to Avoid</title>
<description>
Learn from common mistakes. These anti-patterns lead to unmaintainable infrastructure, security vulnerabilities, or cost overruns.
</description>

<anti_pattern name="manual_portal_creation">
<mistake>Creating resources manually in Azure Portal instead of using IaC</mistake>
<why_bad>
- Cannot replicate to other environments
- No version control or audit trail
- Configuration drift over time
- Can't automate deployments
- Team members don't know what exists
</why_bad>
<correct_approach>Always define resources in Bicep or Terraform templates. Portal is only for viewing/troubleshooting.</correct_approach>
<reference>references/anti-patterns.md</reference>
</anti_pattern>

<anti_pattern name="hardcoded_values">
<mistake>Hardcoding environment-specific values in templates instead of using parameters</mistake>
<examples>
- Hardcoded resource names: "myapp-prod-app" instead of "${productName}-${environment}-app"
- Hardcoded SKUs: Always using "P2v3" instead of environment-based selection
- Hardcoded regions: "eastus" with no parameter
</examples>
<why_bad>Can't reuse template for multiple environments or products. Leads to template duplication.</why_bad>
<correct_approach>Use parameters/variables for all environment-specific and product-specific values.</correct_approach>
</anti_pattern>

<anti_pattern name="missing_tags">
<mistake>Not tagging resources or inconsistent tagging</mistake>
<why_bad>
- Can't allocate costs to products or teams
- Automation scripts can't find resources
- Unclear ownership when issues arise
- Compliance violations (many orgs require tagging)
</why_bad>
<correct_approach>Define common tags in variables and apply consistently to all resources. Enforce with Azure Policy.</correct_approach>
<reference>references/resource-organization.md</reference>
</anti_pattern>

<anti_pattern name="wrong_skus_for_environment">
<mistake>Using Free/Basic SKUs in production or expensive SKUs in dev</mistake>
<examples>
- Production app on Free tier (no SLA, auto-sleep)
- Development SQL Database on Premium tier ($1000+/month wasted)
- Basic tier SQL Database in production (no high availability)
</examples>
<why_bad>Either wastes money (over-provisioned dev) or creates outages (under-provisioned prod).</why_bad>
<correct_approach>Use environment-based SKU selection: Basic/B1 for dev, Standard/S for staging, Premium/P for prod.</correct_approach>
</anti_pattern>

<anti_pattern name="ignoring_security">
<mistake>Not enabling security features by default</mistake>
<examples>
- HTTPS not enforced (httpsOnly: false)
- FTP enabled (security risk)
- TLS 1.0/1.1 allowed (vulnerable protocols)
- Public access to storage accounts
- No Managed Identity (credentials in code)
</examples>
<why_bad>Creates security vulnerabilities exploitable by attackers. Fails compliance audits.</why_bad>
<correct_approach>Enable all security best practices in templates: HTTPS only, TLS 1.2+, Managed Identity, disable FTP, Private Endpoints.</correct_approach>
<reference>references/security-identity.md</reference>
</anti_pattern>

<anti_pattern name="credentials_in_code">
<mistake>Storing connection strings with passwords in app settings or hardcoded in application</mistake>
<examples>
- SQL connection string: "Server=...;User=sa;Password=mypassword123"
- Storage connection string with access keys
- Redis password in appsettings.json
</examples>
<why_bad>Credentials leak in logs, code repositories, CI/CD pipelines. Credentials rotation is difficult.</why_bad>
<correct_approach>Always use Managed Identity. For legacy systems that require credentials, use Key Vault references.</correct_approach>
<reference>references/security-identity.md (zero-credentials architecture)</reference>
</anti_pattern>

<anti_pattern name="no_validation">
<mistake>Deploying templates without running validation first</mistake>
<why_bad>
- Wastes time on failed deployments
- Can create partial deployments that are hard to clean up
- Discovers errors after making production changes
</why_bad>
<correct_approach>Always run: az bicep build + what-if (Bicep) or terraform validate + plan (Terraform)</correct_approach>
</anti_pattern>

<anti_pattern name="no_naming_convention">
<mistake>Inconsistent or unclear resource names</mistake>
<examples>
- "app1", "app2", "testapp", "prod-app-final-v2"
- Mixing naming styles across resources
- Names don't indicate environment or purpose
</examples>
<why_bad>Impossible to understand what resources are for, which environment, who owns them. Chaos at scale.</why_bad>
<correct_approach>Follow consistent pattern: product-env-region-type-instance</correct_approach>
<reference>references/resource-organization.md</reference>
</anti_pattern>

<anti_pattern name="missing_outputs">
<mistake>Not outputting important values from templates</mistake>
<examples>
- No output for app URL
- No output for Managed Identity principal ID (needed for RBAC)
- No output for connection strings
</examples>
<why_bad>Downstream automation can't get needed values. Manual lookup is error-prone and slow.</why_bad>
<correct_approach>Output all values that will be needed by CI/CD, app configuration, or dependent infrastructure.</correct_approach>
</anti_pattern>

<anti_pattern name="no_version_control">
<mistake>Not committing IaC templates to Git</mistake>
<why_bad>
- No history of changes
- Can't rollback to previous configuration
- Team members don't have access to templates
- No code review process
</why_bad>
<correct_approach>Commit all IaC files to Git immediately. Never deploy from local files only.</correct_approach>
</anti_pattern>
</anti_patterns>

<success_criteria>
<title>Definition of Success</title>
<description>
A well-provisioned infrastructure deployment meets all these criteria. Use this checklist to verify completion.
</description>

<criteria>
<criterion name="iac_defined">
<check>All resources defined in IaC templates (Bicep or Terraform)</check>
<verification>Open template files and confirm all resources from architecture design are present</verification>
</criterion>

<criterion name="naming_convention">
<check>Consistent naming convention followed across all resources</check>
<verification>Resource names match pattern: product-env-region-type-instance</verification>
</criterion>

<criterion name="proper_tagging">
<check>All resources tagged with Environment, Product, CostCenter, Owner</check>
<verification>az resource list --resource-group &lt;rg-name&gt; --query "[].{Name:name, Tags:tags}"</verification>
</criterion>

<criterion name="security_enabled">
<check>Security best practices enabled</check>
<verification_checks>
- HTTPS only enforced on App Service (httpsOnly: true)
- TLS 1.2 minimum (minTlsVersion: '1.2')
- Managed Identity enabled (identity.type: 'SystemAssigned')
- FTP disabled (ftpsState: 'Disabled')
- Storage: no public blob access (allowBlobPublicAccess: false)
</verification_checks>
</criterion>

<criterion name="appropriate_skus">
<check>Appropriate SKUs for environment</check>
<verification_checks>
- Dev: Basic/B1 tier (low cost)
- Staging: Standard/P1v3 tier (production-like)
- Prod: Premium/P2v3+ tier (high availability, performance)
</verification_checks>
</criterion>

<criterion name="deployment_validated">
<check>Deployment validated before executing</check>
<verification>az bicep build or terraform validate ran without errors</verification>
</criterion>

<criterion name="deployment_succeeded">
<check>All resources deployed successfully</check>
<verification>az deployment group show shows "provisioningState": "Succeeded"</verification>
</criterion>

<criterion name="app_accessible">
<check>Application accessible at expected URL</check>
<verification>curl https://&lt;app-name&gt;.azurewebsites.net returns HTTP 200 or 404 (not 502/503)</verification>
<note>404 is acceptable if no application code deployed yet</note>
</criterion>

<criterion name="monitoring_configured">
<check>Application Insights configured and linked to App Service</check>
<verification>App Service has APPINSIGHTS_INSTRUMENTATIONKEY app setting populated</verification>
</criterion>

<criterion name="outputs_captured">
<check>Deployment outputs captured for downstream use</check>
<verification>appServiceUrl, appServiceIdentityPrincipalId, sqlServerFqdn, etc. all populated</verification>
</criterion>

<criterion name="documentation_created">
<check>README.md created with deployment instructions and architecture details</check>
<verification>README.md file exists with deployment commands, resource list, outputs</verification>
</criterion>

<criterion name="version_controlled">
<check>All IaC files committed to Git</check>
<verification>git status shows no uncommitted IaC templates; git log shows commit</verification>
</criterion>

<criterion name="no_manual_resources">
<check>No resources created manually in portal</check>
<verification>All resources in resource group match template definitions</verification>
</criterion>

<criterion name="managed_identity_configured">
<check>Managed Identity configured for service-to-service authentication</check>
<verification>App Service can connect to SQL and Storage using MI (no credentials in app settings)</verification>
</criterion>
</criteria>

<final_check>
All criteria above must be true. If any criterion fails, address it before proceeding to application deployment.
</final_check>
</success_criteria>

<next_steps>
<title>Recommended Next Steps</title>
<description>
After successfully provisioning infrastructure, proceed with these workflows to build a complete, production-ready system.
</description>

<next_workflow priority="1">
<name>Deploy Application Code</name>
<file>workflows/deploy-application.md</file>
<description>Deploy your application code to the newly provisioned App Service using CI/CD or manual deployment</description>
</next_workflow>

<next_workflow priority="2">
<name>Setup CI/CD Pipeline</name>
<file>workflows/setup-cicd-pipeline.md</file>
<description>Automate deployments with GitHub Actions or Azure DevOps pipelines for continuous delivery</description>
</next_workflow>

<next_workflow priority="3">
<name>Configure Monitoring</name>
<file>workflows/setup-monitoring.md</file>
<description>Setup comprehensive monitoring, alerts, and dashboards in Application Insights for observability</description>
</next_workflow>

<next_workflow priority="4">
<name>Implement Security Best Practices</name>
<file>workflows/secure-infrastructure.md</file>
<description>Additional security hardening: Key Vault integration, Private Endpoints, network isolation, security scanning</description>
</next_workflow>

<next_workflow priority="5">
<name>Optimize Costs</name>
<file>workflows/optimize-costs.md</file>
<description>Review spending, implement cost optimization strategies, setup budget alerts</description>
</next_workflow>

<next_workflow priority="6">
<name>Setup Additional Environments</name>
<file>workflows/setup-environments.md</file>
<description>Provision staging and production environments using the same IaC templates with different parameters</description>
</next_workflow>
</next_steps>
