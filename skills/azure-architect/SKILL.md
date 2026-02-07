---
name: azure-architect
description: Builds Azure cloud infrastructure and DevOps pipelines from scratch through production. Use when provisioning new Azure resources, setting up CI/CD pipelines, deploying applications to Azure services (App Service, AKS, Functions), configuring monitoring and security, or managing Azure infrastructure lifecycle. CLI-based with Azure CLI, Bicep/Terraform, and DevOps best practices.
---

<objective>
Build production-grade Azure cloud infrastructure and DevOps pipelines from initial provisioning through production operations. Provides comprehensive guidance for infrastructure as code (Bicep/Terraform), CI/CD automation, application deployment, monitoring setup, security implementation, cost optimization, and troubleshooting. All operations use Azure CLI and follow 2024-2025 best practices for security, cost-consciousness, and operational excellence.
</objective>

<quick_start>
1. Review the 13-option intake menu to select your task (provision, deploy, monitor, etc.)
2. Respond with your choice - Claude will route to the appropriate workflow
3. Follow the step-by-step workflow with provided Azure CLI commands
4. Verify deployment using the validation commands
5. Check success criteria: infrastructure deployed, monitoring active, security enabled, costs tracked
</quick_start>

<success_criteria>
- All Azure resources defined in Infrastructure as Code (Bicep or Terraform, no manual portal changes)
- Infrastructure deployed successfully and resources accessible
- Monitoring configured (Application Insights, Log Analytics) and collecting data
- Security best practices enabled (HTTPS only, Managed Identity, RBAC, no hardcoded secrets)
- Cost estimates provided for current configuration
- All changes version controlled in git
- Verification commands run successfully
</success_criteria>

<essential_principles>
<overview>
Core principles that guide all Azure DevOps architecture workflows in this powerup. These principles must be followed across all tasks - from provisioning through production operations.
</overview>

<principle name="infrastructure_as_code">
<title>Infrastructure as Code is the Source of Truth</title>
<description>
All Azure resources MUST be defined in code (Bicep, Terraform, or ARM templates). Never create resources manually in the portal for production - manual changes create configuration drift and are impossible to replicate or version control.
</description>

<tool_selection>
<option name="bicep">
<use_when>
- Azure-only infrastructure
- You want the newest Azure features immediately
- Team prefers native Azure tooling
- No state file management complexity
</use_when>
</option>

<option name="terraform">
<use_when>
- Multi-cloud or hybrid environments
- Need mature ecosystem and community modules
- Require advanced state management and drift detection
- Team has existing Terraform expertise
</use_when>
</option>
</tool_selection>
</principle>

<principle name="azure_cli_first">
<title>Azure CLI First, Portal Never</title>
<description>
Use Azure CLI (az) for all operations. The portal is for viewing and understanding, not for making changes.
</description>

<benefits>
- Repeatable and scriptable
- Version controlled
- Auditable
- Automatable in CI/CD pipelines
</benefits>
</principle>

<principle name="security_by_default">
<title>Security by Default</title>
<description>
Never hardcode secrets. Security must be baked into infrastructure from the start, not added later.
</description>

<required_practices>
- Azure Key Vault for secrets, keys, and certificates
- Managed Identity for authentication (no credentials in code)
- RBAC for access control (principle of least privilege)
- Azure AD integration for user authentication
- RBAC has replaced legacy access policies - always use RBAC permission model for Key Vault and other resources
</required_practices>
</principle>

<principle name="cost_consciousness">
<title>Cost Consciousness</title>
<description>
Azure costs accumulate quickly. Every architectural decision has cost implications.
</description>

<cost_strategies>
- Right-size resources (don't over-provision)
- Use auto-scaling to match actual demand
- Implement auto-shutdown for non-production resources
- Use Reserved Instances/Savings Plans for predictable workloads
- Tag all resources for cost allocation and tracking
</cost_strategies>
</principle>

<principle name="multi_environment">
<title>Multi-Environment Architecture</title>
<description>
Design for multiple environments from day one. Each environment serves a specific purpose with appropriate resource sizing and cost optimization.
</description>

<environments>
<environment name="dev">
Rapid iteration, minimal cost, auto-shutdown enabled
</environment>
<environment name="staging">
Production-like configuration for integration testing
</environment>
<environment name="production">
High availability, disaster recovery, comprehensive monitoring
</environment>
</environments>

<requirement>
Use consistent naming conventions and resource organization across all environments.
</requirement>
</principle>

<principle name="observability">
<title>Observability is Not Optional</title>
<description>
Production systems without monitoring are blind systems. Observability must be configured before going to production.
</description>

<required_components>
- Application Insights for application telemetry
- Log Analytics for centralized logging
- Azure Monitor for infrastructure metrics
- Alerts for proactive issue detection
</required_components>
</principle>

<principle name="deployment_automation">
<title>Deployment Automation</title>
<description>
Manual deployments create inconsistency and errors. All deployments must flow through CI/CD pipelines.
</description>

<pipeline_options>
- Azure DevOps Pipelines (Azure-native)
- GitHub Actions (code-to-cloud integration)
- YAML-based pipeline definitions (version controlled)
</pipeline_options>

<deployment_strategies>
Use deployment strategies like blue-green or canary to minimize risk.
</deployment_strategies>
</principle>
</essential_principles>

<security_checklist>
<title>Pre-Deployment Security Validation</title>
<description>
Run this checklist before deploying any infrastructure to production. All items must pass.
</description>

<check>No secrets or credentials hardcoded in IaC templates or application code</check>
<check>All services configured to use Managed Identity (no connection strings with credentials)</check>
<check>HTTPS only enabled for all web services (httpsOnly: true)</check>
<check>RBAC roles assigned with least privilege principle</check>
<check>Network security groups (NSGs) configured to restrict traffic</check>
<check>Azure Key Vault configured for secret management</check>
<check>Private Endpoints enabled for data services (no public access)</check>
<check>All resources tagged for governance and cost tracking</check>
</security_checklist>

<intake>
<prompt>What would you like to do?</prompt>

<options>
<option number="1" label="Provision new infrastructure">
Create Azure resources from scratch using Infrastructure as Code
</option>
<option number="2" label="Setup CI/CD pipeline">
Configure automated deployment pipeline with GitHub Actions or Azure DevOps
</option>
<option number="3" label="Deploy application">
Deploy app to Azure (App Service, AKS, Functions, Container Apps, etc.)
</option>
<option number="4" label="Setup monitoring">
Configure Application Insights, Log Analytics, alerts, and dashboards
</option>
<option number="5" label="Debug deployment">
Troubleshoot failed deployments or infrastructure issues
</option>
<option number="6" label="Optimize costs">
Analyze and reduce Azure spending with FinOps strategies
</option>
<option number="7" label="Secure infrastructure">
Implement security best practices (Key Vault, RBAC, Managed Identity)
</option>
<option number="8" label="Setup networking">
Configure VNets, NSGs, Application Gateway, Private Endpoints
</option>
<option number="9" label="Design shared resources">
Architect multi-product shared infrastructure (database servers, storage, etc.)
</option>
<option number="10" label="Scale infrastructure">
Handle growth and traffic increases with auto-scaling
</option>
<option number="11" label="Setup environments">
Create dev/staging/prod environments with appropriate configurations
</option>
<option number="12" label="Implement disaster recovery">
Setup backup, restore, and failover strategies
</option>
<option number="13" label="Something else">
Describe what you need and Claude will provide guidance
</option>
</options>

<instruction>Wait for user response before proceeding.</instruction>
</intake>

<routing>
<route patterns="1|provision|create|infrastructure|new|bicep|terraform"
       workflow="workflows/provision-infrastructure.md">
Create new Azure infrastructure from scratch
</route>

<route patterns="2|cicd|ci/cd|pipeline|devops|github actions|deploy automation"
       workflow="workflows/setup-cicd-pipeline.md">
Configure CI/CD with Azure DevOps or GitHub Actions
</route>

<route patterns="3|deploy|app|application|release|publish"
       workflow="workflows/deploy-application.md">
Deploy applications to Azure services
</route>

<route patterns="4|monitor|monitoring|logs|alerts|insights|observability"
       workflow="workflows/setup-monitoring.md">
Configure monitoring, logging, and alerts
</route>

<route patterns="5|debug|troubleshoot|fix|broken|error|failed|issue"
       workflow="workflows/debug-deployment.md">
Troubleshoot failed deployments and issues
</route>

<route patterns="6|cost|optimize|expensive|finops|spending|budget"
       workflow="workflows/optimize-costs.md">
Analyze and reduce Azure spending
</route>

<route patterns="7|secure|security|rbac|key vault|identity|managed identity"
       workflow="workflows/secure-infrastructure.md">
Implement security best practices
</route>

<route patterns="8|network|vnet|nsg|gateway|subnet|firewall"
       workflow="workflows/setup-networking.md">
Configure VNets, NSGs, Application Gateway
</route>

<route patterns="9|shared|multi-product|multi-tenant|share|database server"
       workflow="workflows/design-shared-resources.md">
Architect shared infrastructure for multiple products
</route>

<route patterns="10|scale|scaling|autoscale|grow|traffic|performance"
       workflow="workflows/scale-infrastructure.md">
Handle growth and traffic scaling
</route>

<route patterns="11|environment|dev|staging|prod|production|test"
       workflow="workflows/setup-environments.md">
Create dev/staging/prod environments
</route>

<route patterns="12|disaster|recovery|backup|restore|failover|dr|bcdr"
       workflow="workflows/implement-disaster-recovery.md">
Setup backup, restore, and failover
</route>

<route patterns="13|migrate|migration|move to azure"
       workflow="workflows/migrate-to-azure.md">
Migrate existing infrastructure to Azure
</route>

<instruction>After reading the routed workflow, follow it exactly step by step.</instruction>
</routing>

<verification>
<title>After Every Change</title>
<description>
Run these verification commands to ensure your changes work correctly. Execute all applicable steps and report results to the user.
</description>

<step number="1" name="validate_iac">
<description>Validate Infrastructure as Code templates</description>
<commands>
<bicep>az bicep build --file main.bicep</bicep>
<terraform>terraform validate &amp;&amp; terraform plan</terraform>
</commands>
</step>

<step number="2" name="check_deployment">
<description>Check resource deployment status</description>
<command>az deployment group show --name &lt;deployment-name&gt; --resource-group &lt;rg-name&gt;</command>
</step>

<step number="3" name="verify_resources">
<description>Verify resources are running</description>
<command>az resource list --resource-group &lt;rg-name&gt; --output table</command>
</step>

<step number="4" name="test_endpoint">
<description>Test application endpoint (if applicable)</description>
<commands>
<option>curl https://&lt;your-app&gt;.azurewebsites.net/health</option>
<option>az webapp show --name &lt;app-name&gt; --resource-group &lt;rg-name&gt; --query "state"</option>
</commands>
</step>

<step number="5" name="check_errors">
<description>Check recent deployments and errors</description>
<command>az monitor activity-log list --resource-group &lt;rg-name&gt; --max-events 10</command>
</step>

<reporting>
<title>Report to User</title>
<status_items>
<item>Infrastructure: ✓ Deployed / ✗ Failed (with error details)</item>
<item>Application: ✓ Running / ✗ Stopped</item>
<item>Monitoring: ✓ Collecting data / ⚠ Needs configuration</item>
<item>Cost estimate: $X/month for current configuration</item>
</status_items>
<requirement>Always provide next steps or recommendations for improvement</requirement>
</reporting>
</verification>

<references>
<title>Domain Knowledge</title>
<description>All reference files located in references/ directory</description>

<category name="Infrastructure">
<file name="infrastructure-as-code.md">Bicep vs Terraform, Azure Verified Modules, state management</file>
<file name="compute-services.md">App Service, Functions, Container Apps, AKS, VMs decision matrix</file>
<file name="storage-data.md">Azure SQL, Cosmos DB, Storage Accounts with pricing</file>
</category>

<category name="Architecture">
<file name="architecture-patterns.md">Microservices, API Gateway, CQRS, Circuit Breaker, BFF</file>
<file name="multi-tenancy-patterns.md">Shared database/schema, database-per-tenant, governance</file>
<file name="anti-patterns.md">What NOT to do across all domains</file>
</category>

<category name="Deployment">
<file name="cicd-pipelines.md">GitHub Actions with OIDC (no secrets!), Azure DevOps</file>
<file name="deployment-strategies.md">Blue-green, canary, rolling updates, zero-downtime</file>
</category>

<category name="Networking">
<file name="networking.md">VNets, NSGs, Private Endpoints, Application Gateway</file>
</category>

<category name="Kubernetes">
<file name="kubernetes-aks.md">Production AKS setup, node pools, auto-scaling, security</file>
</category>

<category name="Security">
<file name="security-identity.md">Managed Identity (zero credentials), RBAC, Key Vault</file>
</category>

<category name="Observability">
<file name="monitoring-observability.md">Application Insights, Log Analytics, KQL queries</file>
</category>

<category name="Operations">
<file name="cost-optimization.md">FinOps fundamentals, real pricing, savings calculations</file>
<file name="disaster-recovery.md">RTO/RPO, Azure Backup, Site Recovery, DR testing</file>
</category>

<category name="Organization">
<file name="resource-organization.md">Naming conventions, tagging strategies, management groups</file>
</category>
</references>

<workflows>
<title>Available Workflows</title>
<description>All workflow files located in workflows/ directory</description>

<workflow name="provision-infrastructure.md">
Create new Azure infrastructure from scratch with IaC
</workflow>

<workflow name="setup-cicd-pipeline.md">
Configure CI/CD with Azure DevOps or GitHub Actions
</workflow>

<workflow name="deploy-application.md">
Deploy applications to Azure services
</workflow>

<workflow name="setup-monitoring.md">
Configure monitoring, logging, and alerts
</workflow>

<workflow name="debug-deployment.md">
Troubleshoot failed deployments and issues
</workflow>

<workflow name="optimize-costs.md">
Analyze and reduce Azure spending
</workflow>

<workflow name="secure-infrastructure.md">
Implement security best practices
</workflow>

<workflow name="setup-networking.md">
Configure VNets, NSGs, Application Gateway
</workflow>

<workflow name="design-shared-resources.md">
Architect shared infrastructure for multiple products
</workflow>

<workflow name="scale-infrastructure.md">
Handle growth and traffic scaling
</workflow>

<workflow name="setup-environments.md">
Create dev/staging/prod environments
</workflow>

<workflow name="implement-disaster-recovery.md">
Setup backup, restore, and failover
</workflow>

<workflow name="migrate-to-azure.md">
Migrate existing infrastructure to Azure
</workflow>
</workflows>
