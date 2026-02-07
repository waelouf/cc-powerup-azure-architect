<required_reading>
<title>Required Reading Before Starting</title>
<description>
Read these reference files to understand deployment options, strategies, and monitoring setup before deploying applications to Azure.
</description>

<reference file="references/compute-services.md">
Decision matrix for choosing the right Azure compute service (App Service, Functions, AKS, Container Apps, Static Web Apps) based on application characteristics
</reference>

<reference file="references/deployment-strategies.md">
Blue-green deployments, canary releases, rolling updates, deployment slots, zero-downtime deployment patterns
</reference>

<reference file="references/monitoring-observability.md">
Application Insights integration, logging best practices, metrics collection, alerting configuration
</reference>
</required_reading>

<objective>
Deploy application code to Azure compute services (App Service, Azure Functions, AKS, Container Apps, or Static Web Apps) using appropriate deployment methods. Implements zero-downtime deployment strategies using staging slots, ensures health checks are configured, sets up application configuration via environment variables and Key Vault references, and verifies deployment success through health endpoints and Application Insights telemetry. Follows 2024-2025 deployment best practices including staged deployments, automated testing, and rollback procedures.
</objective>

<prerequisites>
<prerequisite>Infrastructure already provisioned (resource group, compute service, supporting services)</prerequisite>
<prerequisite>Application code ready to deploy with build artifacts (ZIP file, container image, or K8s manifests)</prerequisite>
<prerequisite>Azure CLI authenticated with access to target resource group</prerequisite>
<prerequisite>Application has /health endpoint implemented for health checks</prerequisite>
<prerequisite>Configuration externalized via environment variables (no hardcoded values)</prerequisite>
<prerequisite>Application Insights SDK integrated in application code</prerequisite>
<prerequisite>For AKS: kubectl installed and configured</prerequisite>
<prerequisite>For Functions: Azure Functions Core Tools installed (if deploying locally)</prerequisite>
</prerequisites>

<process>
<step number="1" name="determine_target_service">
<title>Determine Target Compute Service</title>
<description>
Identify which Azure compute service the application should be deployed to based on application type, architecture, and requirements. Different services have different deployment methods and capabilities.
</description>

<application_types>
<application_type name="web_app_api">
<description>Web applications, REST APIs, GraphQL APIs</description>
<recommended_services>
<service priority="1">
<name>Azure App Service</name>
<best_for>Most web apps and APIs, simplest deployment, built-in scaling and deployment slots</best_for>
<deployment_method>ZIP deploy, Docker container, or continuous deployment</deployment_method>
</service>
<service priority="2">
<name>Azure Kubernetes Service (AKS)</name>
<best_for>Microservices, complex architectures, need full Kubernetes control</best_for>
<deployment_method>kubectl apply with K8s manifests or Helm charts</deployment_method>
</service>
</recommended_services>
</application_type>

<application_type name="serverless_functions">
<description>Event-driven functions, background processing, scheduled jobs</description>
<recommended_services>
<service priority="1">
<name>Azure Functions</name>
<best_for>Serverless workloads, pay-per-execution, event triggers</best_for>
<deployment_method>func azure functionapp publish or ZIP deploy</deployment_method>
</service>
</recommended_services>
</application_type>

<application_type name="container_app">
<description>Containerized applications without Kubernetes complexity</description>
<recommended_services>
<service priority="1">
<name>Azure Container Apps</name>
<best_for>Containers with simpler management than AKS, KEDA-based scaling</best_for>
<deployment_method>az containerapp update with container image</deployment_method>
</service>
<service priority="2">
<name>Azure Kubernetes Service (AKS)</name>
<best_for>Need full Kubernetes features, complex orchestration</best_for>
<deployment_method>kubectl apply with K8s manifests</deployment_method>
</service>
</recommended_services>
</application_type>

<application_type name="static_site">
<description>Static HTML/CSS/JS, JAMstack applications, SPAs</description>
<recommended_services>
<service priority="1">
<name>Azure Static Web Apps</name>
<best_for>Static sites with optional serverless APIs, built-in CI/CD from GitHub</best_for>
<deployment_method>GitHub Actions integration or SWA CLI</deployment_method>
</service>
<service priority="2">
<name>Azure App Service (Static)</name>
<best_for>Simple static hosting as part of larger App Service deployment</best_for>
<deployment_method>ZIP deploy of static files</deployment_method>
</service>
</recommended_services>
</application_type>
</application_types>

<service_selection_guide>
<decision_question>What type of application are you deploying?</decision_question>
<wait_for_response>Get user response before proceeding to deployment steps</wait_for_response>
</service_selection_guide>

<reference>references/compute-services.md (detailed service comparison)</reference>
</step>

<step number="2" name="prepare_application">
<title>Prepare Application for Deployment</title>
<description>
Ensure the application meets Azure deployment requirements including health endpoints, externalized configuration, proper logging, and monitoring integration. These requirements enable zero-downtime deployments and effective troubleshooting.
</description>

<application_requirements>
<requirement name="health_endpoint" priority="critical">
<title>Health Check Endpoint</title>
<description>
Application must expose a health endpoint that returns HTTP 200 when healthy. Azure uses this for health probes, load balancing decisions, and deployment verification.
</description>

<endpoint_patterns>
<pattern>/health</pattern>
<pattern>/healthz</pattern>
<pattern>/api/health</pattern>
<pattern>/.well-known/health</pattern>
</endpoint_patterns>

<implementation_guidance>
<framework name="express_nodejs">
<code><![CDATA[
app.get('/health', (req, res) => {
  // Check database connectivity, dependencies, etc.
  const healthy = checkDatabaseConnection() && checkDependencies();

  if (healthy) {
    res.status(200).json({ status: 'healthy', timestamp: new Date() });
  } else {
    res.status(503).json({ status: 'unhealthy', timestamp: new Date() });
  }
});
]]></code>
</framework>

<framework name="aspnet_core">
<code><![CDATA[
app.MapHealthChecks("/health", new HealthCheckOptions
{
    ResponseWriter = async (context, report) =>
    {
        context.Response.ContentType = "application/json";
        var result = JsonSerializer.Serialize(new
        {
            status = report.Status.ToString(),
            checks = report.Entries.Select(e => new { name = e.Key, status = e.Value.Status.ToString() })
        });
        await context.Response.WriteAsync(result);
    }
});
]]></code>
</framework>
</implementation_guidance>

<health_check_criteria>
<check>Endpoint returns HTTP 200 when healthy, 503 when unhealthy</check>
<check>Response time under 2 seconds</check>
<check>Checks database connectivity (if applicable)</check>
<check>Checks external dependency availability (if critical)</check>
<check>Does NOT perform expensive operations (avoid performance impact)</check>
</health_check_criteria>
</requirement>

<requirement name="externalized_configuration" priority="critical">
<title>Externalized Configuration</title>
<description>
All environment-specific configuration must come from environment variables or Azure Key Vault, never hardcoded in application code or committed to source control.
</description>

<configuration_sources>
<source name="environment_variables">
<description>Standard configuration values (non-sensitive)</description>
<examples>
- NODE_ENV=production
- LOG_LEVEL=info
- PORT=8080
- API_BASE_URL=https://api.example.com
</examples>
</source>

<source name="keyvault_references">
<description>Sensitive values (connection strings, API keys, secrets)</description>
<syntax>@Microsoft.KeyVault(SecretUri=https://&lt;vault&gt;.vault.azure.net/secrets/&lt;secret-name&gt;/)</syntax>
<examples>
- DATABASE_URL=@Microsoft.KeyVault(SecretUri=https://myapp-kv.vault.azure.net/secrets/db-connection/)
- API_KEY=@Microsoft.KeyVault(SecretUri=https://myapp-kv.vault.azure.net/secrets/api-key/)
</examples>
</source>

<source name="managed_identity">
<description>Zero-credentials access to Azure services</description>
<note>Use DefaultAzureCredential in application code - no connection strings needed</note>
</source>
</configuration_sources>

<anti_patterns>
<anti_pattern>Hardcoding connection strings in appsettings.json</anti_pattern>
<anti_pattern>Different configuration files per environment (appsettings.dev.json, appsettings.prod.json)</anti_pattern>
<anti_pattern>Storing secrets in environment variables in source control</anti_pattern>
</anti_patterns>
</requirement>

<requirement name="logging" priority="high">
<title>Proper Logging Configuration</title>
<description>
Application must log to stdout/stderr (not files) so Azure can capture logs. Use structured logging for better querying in Application Insights.
</description>

<logging_best_practices>
<practice>Log to stdout (info) and stderr (errors), never to local files</practice>
<practice>Use structured/JSON logging for better parsing and querying</practice>
<practice>Include correlation IDs for request tracing</practice>
<practice>Log at appropriate levels (DEBUG, INFO, WARN, ERROR)</practice>
<practice>Avoid logging sensitive data (passwords, tokens, PII)</practice>
<practice>Include context: timestamp, user ID, request ID, operation name</practice>
</logging_best_practices>

<example_structured_logging>
<code><![CDATA[
// Good: Structured logging
logger.info({
  event: 'user_login',
  userId: user.id,
  timestamp: new Date(),
  requestId: req.id,
  duration: 145
});

// Bad: String concatenation
logger.info('User ' + user.id + ' logged in at ' + new Date());
]]></code>
</example_structured_logging>
</requirement>

<requirement name="app_insights_integration" priority="high">
<title>Application Insights Integration</title>
<description>
Integrate Application Insights SDK for automatic telemetry collection (requests, dependencies, exceptions, custom metrics).
</description>

<integration_methods>
<method name="sdk_integration">
<description>Recommended: Use Application Insights SDK in application code</description>
<nodejs>npm install applicationinsights</nodejs>
<dotnet>Install-Package Microsoft.ApplicationInsights.AspNetCore</dotnet>
</method>

<method name="auto_instrumentation">
<description>Alternative: Use agent-based auto-instrumentation (App Service only)</description>
<note>Set APPINSIGHTS_INSTRUMENTATIONKEY or APPLICATIONINSIGHTS_CONNECTION_STRING app setting</note>
</method>
</integration_methods>

<verification>Application Insights dashboard shows incoming requests after deployment</verification>
</requirement>

<requirement name="build_artifacts" priority="critical">
<title>Build Artifacts Ready</title>
<description>
Application must be built and packaged in format appropriate for target service.
</description>

<artifact_formats>
<format service="app_service">
<type>ZIP file</type>
<contents>Application code, dependencies, package.json/requirements.txt, static assets</contents>
<exclude>node_modules (install via package.json), .git, .env files, local config</exclude>
</format>

<format service="functions">
<type>ZIP file with host.json</type>
<contents>Function code, host.json, local.settings.json (template only), requirements.txt</contents>
</format>

<format service="aks">
<type>Container image + K8s manifests</type>
<contents>Docker image pushed to ACR, deployment.yaml, service.yaml, ingress.yaml</contents>
</format>

<format service="container_apps">
<type>Container image</type>
<contents>Docker image pushed to ACR or Docker Hub</contents>
</format>
</artifact_formats>
</requirement>
</application_requirements>

<validation_checklist>
<check>Health endpoint returns HTTP 200 when tested locally</check>
<check>Application starts successfully with environment variables</check>
<check>No hardcoded configuration values in code</check>
<check>Application logs to stdout/stderr</check>
<check>Application Insights SDK initialized</check>
<check>Build artifacts created successfully</check>
</validation_checklist>
</step>

<step number="3" name="deploy_by_service_type">
<title>Deploy Application to Target Service</title>
<description>
Execute deployment using the appropriate method for the target Azure service. Each service has different deployment mechanisms, capabilities, and best practices.
</description>

<deployment_methods>
<app_service>
<title>Azure App Service Deployment</title>
<description>
Deploy web applications and APIs to Azure App Service using ZIP deployment with staging slots for zero-downtime deployments.
</description>

<deployment_strategy name="staging_slot_deployment">
<title>Zero-Downtime Deployment with Staging Slots (Recommended)</title>
<description>
Deploy to staging slot, test thoroughly, then swap to production. This provides instant rollback capability and zero downtime.
</description>

<steps>
<step>
<title>Deploy to Staging Slot</title>
<command type="azure-cli"><![CDATA[
# Deploy application to staging slot
az webapp deployment source config-zip \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-app-001 \
  --slot staging \
  --src dist/app.zip

# Wait for deployment to complete (typically 1-3 minutes)
]]></command>

<note>ZIP file should contain application code, dependencies, and static assets. Excludes node_modules (installed via package.json).</note>
</step>

<step>
<title>Warm Up Staging Slot</title>
<description>Send requests to staging slot to warm up the application before swapping</description>
<command type="bash"><![CDATA[
# Get staging URL
STAGING_URL="https://myapp-prod-eastus-app-001-staging.azurewebsites.net"

# Test health endpoint
curl -f $STAGING_URL/health || { echo "Health check failed"; exit 1; }

# Warm up by sending test requests
for i in {1..10}; do
  curl -s $STAGING_URL/api/test > /dev/null
  sleep 1
done

echo "Staging slot warmed up and healthy"
]]></command>
</step>

<step>
<title>Run Smoke Tests on Staging</title>
<description>Execute automated tests against staging slot to verify deployment</description>
<test_scenarios>
<test>Health endpoint returns 200</test>
<test>API endpoints return expected responses</test>
<test>Database connectivity working</test>
<test>Application Insights receiving telemetry</test>
<test>No error logs in recent 5 minutes</test>
</test_scenarios>
</step>

<step>
<title>Swap Staging to Production</title>
<description>Perform zero-downtime swap of staging slot to production</description>
<command type="azure-cli"><![CDATA[
# Swap staging slot to production
az webapp deployment slot swap \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-app-001 \
  --slot staging \
  --target-slot production

# Swap completes instantly (traffic now goes to new version)
echo "Swap completed - production is now running new version"
]]></command>

<note>Swap is instant (under 1 second). Previous production version is now in staging slot for quick rollback if needed.</note>
</step>

<step>
<title>Monitor Production After Swap</title>
<description>Watch production metrics for 15-30 minutes to catch issues</description>
<monitoring_commands><![CDATA[
# Check production health
curl -f https://myapp-prod-eastus-app-001.azurewebsites.net/health

# Watch logs in real-time
az webapp log tail \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-app-001

# Check Application Insights for errors (in Azure Portal or CLI)
]]></monitoring_commands>

<red_flags>
<flag>Error rate spike in Application Insights</flag>
<flag>Response time degradation (>2x normal)</flag>
<flag>Health check failures</flag>
<flag>Increased CPU/memory usage</flag>
<flag>Database connection errors</flag>
</red_flags>

<rollback_procedure>
If issues detected, swap back immediately (previous version still in staging):
<command><![CDATA[
az webapp deployment slot swap \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-app-001 \
  --slot staging \
  --target-slot production
]]></command>
</rollback_procedure>
</step>
</steps>

<benefits>
- Zero downtime (swap is instant)
- Instant rollback (just swap back)
- Test in production-like environment before going live
- Can compare staging and production side-by-side
</benefits>
</deployment_strategy>

<deployment_strategy name="direct_deployment">
<title>Direct Production Deployment (Not Recommended for Production)</title>
<description>
Deploy directly to production slot. Only use for dev/staging environments or when deployment slots are not available (Basic tier).
</description>

<warning>This causes downtime during deployment and has no instant rollback. Use staging slots for production deployments.</warning>

<command><![CDATA[
# Direct deployment to production (causes ~30-60 seconds downtime)
az webapp deployment source config-zip \
  --resource-group myapp-dev-eastus-rg \
  --name myapp-dev-eastus-app-001 \
  --src dist/app.zip
]]></command>
</deployment_strategy>

<alternative_methods>
<method name="docker_container">
<title>Deploy Docker Container to App Service</title>
<command><![CDATA[
# Configure App Service to use container from ACR
az webapp config container set \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-app-001 \
  --docker-custom-image-name myappacr.azurecr.io/myapp:v1.2.3 \
  --docker-registry-server-url https://myappacr.azurecr.io

# Restart to pull new image
az webapp restart \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-app-001
]]></command>
</method>

<method name="github_actions">
<title>Continuous Deployment from GitHub Actions</title>
<description>Preferred for automated CI/CD - see workflows/setup-cicd-pipeline.md</description>
</method>
</alternative_methods>
</app_service>

<azure_functions>
<title>Azure Functions Deployment</title>
<description>
Deploy serverless functions using Azure Functions Core Tools or ZIP deployment.
</description>

<deployment_methods>
<method name="func_cli" recommended="true">
<title>Using Azure Functions Core Tools</title>
<description>Easiest method for deploying functions from local machine</description>

<steps>
<step>
<title>Deploy Function App</title>
<command><![CDATA[
# Deploy using func CLI (automatically creates ZIP and deploys)
func azure functionapp publish myapp-prod-eastus-func-001

# Deployment includes:
# - Building function app
# - Creating deployment package
# - Uploading to Azure
# - Restarting function app
]]></command>

<expected_output>
Deployment successful. Function URLs displayed for HTTP-triggered functions.
</expected_output>
</step>

<step>
<title>Verify Function Endpoints</title>
<command><![CDATA[
# Test HTTP-triggered function
curl -f https://myapp-prod-eastus-func-001.azurewebsites.net/api/myfunction

# View function logs
func azure functionapp logstream myapp-prod-eastus-func-001
]]></command>
</step>
</steps>
</method>

<method name="zip_deployment">
<title>Using ZIP Deployment (CI/CD Pipelines)</title>
<command><![CDATA[
# Create ZIP of function app (include host.json, function.json, etc.)
zip -r function-app.zip . -x "*.git*" -x "local.settings.json"

# Deploy ZIP to Function App
az functionapp deployment source config-zip \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-func-001 \
  --src function-app.zip

# Sync triggers (ensure function host is aware of all triggers)
az functionapp function sync \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-func-001
]]></command>
</method>
</deployment_methods>

<deployment_slots_for_functions>
<description>Functions also support deployment slots for zero-downtime deployments</description>
<command><![CDATA[
# Deploy to staging slot
func azure functionapp publish myapp-prod-eastus-func-001 --slot staging

# Test staging
curl https://myapp-prod-eastus-func-001-staging.azurewebsites.net/api/myfunction

# Swap to production
az functionapp deployment slot swap \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-func-001 \
  --slot staging \
  --target-slot production
]]></command>
</deployment_slots_for_functions>
</azure_functions>

<aks_kubernetes>
<title>Azure Kubernetes Service (AKS) Deployment</title>
<description>
Deploy containerized applications to AKS using kubectl and Kubernetes manifests.
</description>

<prerequisites>
<prerequisite>Container image built and pushed to Azure Container Registry (ACR)</prerequisite>
<prerequisite>Kubernetes manifests created (deployment.yaml, service.yaml, ingress.yaml)</prerequisite>
<prerequisite>kubectl configured with AKS credentials</prerequisite>
</prerequisites>

<steps>
<step>
<title>Get AKS Credentials</title>
<command><![CDATA[
# Configure kubectl to use AKS cluster
az aks get-credentials \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-aks-001 \
  --overwrite-existing

# Verify connection
kubectl cluster-info
kubectl get nodes
]]></command>
</step>

<step>
<title>Create or Update Kubernetes Resources</title>
<description>Apply Kubernetes manifests to deploy application</description>

<manifest_example name="deployment.yaml">
<code><![CDATA[
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
  labels:
    app: myapp
    version: v1.2.3
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0  # Zero-downtime deployment
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        version: v1.2.3
    spec:
      containers:
      - name: myapp
        image: myappacr.azurecr.io/myapp:v1.2.3
        ports:
        - containerPort: 8080
        env:
        - name: NODE_ENV
          value: "production"
        - name: PORT
          value: "8080"
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
]]></code>
</manifest_example>

<deployment_command><![CDATA[
# Apply all manifests in k8s/ directory
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml

# Alternative: Apply entire directory
kubectl apply -f k8s/
]]></deployment_command>
</step>

<step>
<title>Monitor Rollout Status</title>
<description>Watch deployment progress and ensure pods are healthy</description>
<command><![CDATA[
# Watch rollout progress (waits until deployment is complete)
kubectl rollout status deployment/myapp -n production

# Check pod status
kubectl get pods -n production -l app=myapp

# View recent events
kubectl get events -n production --sort-by='.lastTimestamp'

# Check logs from new pods
kubectl logs -n production -l app=myapp --tail=50 -f
]]></command>

<expected_output>
<output>deployment "myapp" successfully rolled out</output>
<output>All pods in Running state with READY 1/1</output>
<output>No error events in recent events list</output>
</expected_output>
</step>

<step>
<title>Verify Service Accessibility</title>
<command><![CDATA[
# Get service external IP (if LoadBalancer type)
kubectl get service myapp -n production

# Test service endpoint
EXTERNAL_IP=$(kubectl get service myapp -n production -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
curl -f http://$EXTERNAL_IP/health

# Or test via ingress (if configured)
curl -f https://myapp.example.com/health
]]></command>
</step>
</steps>

<rollback_procedure>
<title>Rollback AKS Deployment</title>
<description>If deployment fails or causes issues, rollback to previous version</description>
<command><![CDATA[
# View rollout history
kubectl rollout history deployment/myapp -n production

# Rollback to previous version
kubectl rollout undo deployment/myapp -n production

# Rollback to specific revision
kubectl rollout undo deployment/myapp -n production --to-revision=2

# Verify rollback completed
kubectl rollout status deployment/myapp -n production
]]></command>
</rollback_procedure>

<deployment_strategies>
<strategy name="rolling_update">
<description>Default: Gradually replace pods with new version (zero downtime)</description>
<configuration>maxSurge: 1, maxUnavailable: 0</configuration>
</strategy>

<strategy name="blue_green">
<description>Deploy new version alongside old, then switch traffic</description>
<implementation>Create two deployments (blue, green) and update service selector</implementation>
</strategy>

<strategy name="canary">
<description>Route small percentage of traffic to new version, gradually increase</description>
<implementation>Use service mesh (Istio, Linkerd) or ingress controller with traffic splitting</implementation>
<reference>references/deployment-strategies.md</reference>
</strategy>
</deployment_strategies>
</aks_kubernetes>

<container_apps>
<title>Azure Container Apps Deployment</title>
<description>
Deploy containerized applications to Azure Container Apps - simpler than AKS, with built-in scaling and ingress.
</description>

<deployment_command><![CDATA[
# Update Container App with new image
az containerapp update \
  --name myapp-prod-eastus-ca-001 \
  --resource-group myapp-prod-eastus-rg \
  --image myappacr.azurecr.io/myapp:v1.2.3

# Container Apps automatically does rolling update with zero downtime

# Check deployment status
az containerapp show \
  --name myapp-prod-eastus-ca-001 \
  --resource-group myapp-prod-eastus-rg \
  --query "properties.latestRevisionName"

# View application logs
az containerapp logs show \
  --name myapp-prod-eastus-ca-001 \
  --resource-group myapp-prod-eastus-rg \
  --follow
]]></command>

<revision_management>
<description>Container Apps uses revisions - each deployment creates a new revision</description>
<command><![CDATA[
# List all revisions
az containerapp revision list \
  --name myapp-prod-eastus-ca-001 \
  --resource-group myapp-prod-eastus-rg \
  --output table

# Activate specific revision (rollback)
az containerapp revision activate \
  --name myapp-prod-eastus-ca-001 \
  --resource-group myapp-prod-eastus-rg \
  --revision <revision-name>
]]></command>
</revision_management>

<traffic_splitting>
<description>Split traffic between revisions for canary deployments</description>
<command><![CDATA[
# Send 90% to old version, 10% to new version (canary)
az containerapp ingress traffic set \
  --name myapp-prod-eastus-ca-001 \
  --resource-group myapp-prod-eastus-rg \
  --revision-weight <old-revision>=90 <new-revision>=10

# After validation, shift 100% to new version
az containerapp ingress traffic set \
  --name myapp-prod-eastus-ca-001 \
  --resource-group myapp-prod-eastus-rg \
  --revision-weight <new-revision>=100
]]></command>
</traffic_splitting>
</container_apps>

<static_web_apps>
<title>Azure Static Web Apps Deployment</title>
<description>
Deploy static websites and SPAs to Azure Static Web Apps with integrated API support.
</description>

<deployment_methods>
<method name="github_actions" recommended="true">
<description>Automatic deployment via GitHub Actions (created when setting up SWA)</description>
<note>Push to GitHub triggers automatic build and deployment - no manual steps needed</note>
</method>

<method name="swa_cli">
<description>Manual deployment using SWA CLI</description>
<command><![CDATA[
# Install SWA CLI
npm install -g @azure/static-web-apps-cli

# Deploy to Azure
swa deploy \
  --app-location ./dist \
  --api-location ./api \
  --deployment-token <token-from-portal>
]]></command>
</method>
</deployment_methods>
</static_web_apps>
</deployment_methods>
</step>

<step number="4" name="configure_application_settings">
<title>Configure Application Settings and Secrets</title>
<description>
Set environment variables and configure Key Vault references for application configuration. All sensitive values should be stored in Azure Key Vault and referenced via app settings.
</description>

<configuration_management>
<app_service_configuration>
<title>App Service / Functions Configuration</title>
<description>
Configure app settings (environment variables) and connection strings using Azure CLI.
</description>

<set_app_settings>
<command><![CDATA[
# Set multiple application settings at once
az webapp config appsettings set \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-app-001 \
  --settings \
    NODE_ENV=production \
    LOG_LEVEL=info \
    API_TIMEOUT=30000 \
    FEATURE_FLAG_NEW_UI=true

# Verify settings applied
az webapp config appsettings list \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-app-001 \
  --output table
]]></command>
</set_app_settings>

<keyvault_references>
<title>Configure Key Vault References (Recommended for Secrets)</title>
<description>
Reference secrets from Key Vault instead of storing them directly in app settings. App Service uses Managed Identity to retrieve secrets automatically.
</description>

<command><![CDATA[
# Set app settings with Key Vault references
az webapp config appsettings set \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-app-001 \
  --settings \
    DATABASE_CONNECTION_STRING="@Microsoft.KeyVault(SecretUri=https://myapp-prod-kv.vault.azure.net/secrets/db-connection-string/)" \
    API_KEY="@Microsoft.KeyVault(SecretUri=https://myapp-prod-kv.vault.azure.net/secrets/api-key/)" \
    STORAGE_CONNECTION_STRING="@Microsoft.KeyVault(SecretUri=https://myapp-prod-kv.vault.azure.net/secrets/storage-connection/)"

# App Service automatically resolves these references using Managed Identity
# Application code receives the actual secret values as environment variables
]]></command>

<note>Ensure App Service Managed Identity has "Get" permission on Key Vault secrets</note>
<reference>references/security-identity.md (Key Vault integration)</reference>
</keyvault_references>

<connection_strings>
<description>For compatibility with some frameworks, use connection strings setting type</description>
<command><![CDATA[
az webapp config connection-string set \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-app-001 \
  --connection-string-type SQLAzure \
  --settings \
    DefaultConnection="Server=myapp-prod-sqlsrv.database.windows.net;Database=myapp-prod-db;Authentication=Active Directory Default;"
]]></command>
<note>Connection string uses Managed Identity authentication (no password)</note>
</connection_strings>
</app_service_configuration>

<aks_configuration>
<title>AKS Configuration (ConfigMaps and Secrets)</title>
<description>
Use Kubernetes ConfigMaps for non-sensitive config and Secrets for sensitive data. Can also integrate with Azure Key Vault via CSI driver.
</description>

<configmap_example>
<code><![CDATA[
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
  namespace: production
data:
  NODE_ENV: "production"
  LOG_LEVEL: "info"
  API_TIMEOUT: "30000"
  FEATURE_FLAG_NEW_UI: "true"
]]></code>

<usage_in_deployment>
<code><![CDATA[
spec:
  containers:
  - name: myapp
    envFrom:
    - configMapRef:
        name: myapp-config
]]></code>
</usage_in_deployment>
</configmap_example>

<secrets_with_keyvault>
<description>Use Azure Key Vault CSI driver to mount secrets from Key Vault</description>
<reference>references/kubernetes-aks.md (Key Vault integration)</reference>
</secrets_with_keyvault>
</aks_configuration>
</configuration_management>

<application_insights_configuration>
<title>Configure Application Insights</title>
<description>
Ensure Application Insights is properly configured for telemetry collection.
</description>

<command><![CDATA[
# Get Application Insights instrumentation key and connection string
INSTRUMENTATION_KEY=$(az monitor app-insights component show \
  --app myapp-prod-eastus-insights-001 \
  --resource-group myapp-prod-eastus-rg \
  --query instrumentationKey -o tsv)

CONNECTION_STRING=$(az monitor app-insights component show \
  --app myapp-prod-eastus-insights-001 \
  --resource-group myapp-prod-eastus-rg \
  --query connectionString -o tsv)

# Configure App Service to use Application Insights
az webapp config appsettings set \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-app-001 \
  --settings \
    APPINSIGHTS_INSTRUMENTATIONKEY=$INSTRUMENTATION_KEY \
    APPLICATIONINSIGHTS_CONNECTION_STRING=$CONNECTION_STRING \
    ApplicationInsightsAgent_EXTENSION_VERSION="~3"

# For Node.js, also set:
# APPINSIGHTS_PROFILERFEATURE_VERSION="1.0.0"
# APPINSIGHTS_SNAPSHOTFEATURE_VERSION="1.0.0"
]]></command>

<verification>
Application Insights should start receiving telemetry within 1-2 minutes of application startup
</verification>
</application_insights_configuration>
</step>

<step number="5" name="verify_deployment">
<title>Verify Deployment Success</title>
<description>
Run comprehensive verification checks to ensure application is deployed correctly, accessible, and functioning as expected. This includes health checks, endpoint testing, log analysis, and telemetry verification.
</description>

<verification_checks>
<check name="app_status">
<title>Verify Application Status</title>
<description>Confirm application service is running and healthy</description>

<app_service_check>
<command><![CDATA[
# Check App Service state
az webapp show \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-app-001 \
  --query "{Name:name, State:state, AvailabilityState:availabilityState, DefaultHostName:defaultHostName}"
]]></command>
<expected_result>
<field name="State">"Running"</field>
<field name="AvailabilityState">"Normal"</field>
</expected_result>
</app_service_check>

<aks_check>
<command><![CDATA[
# Check pod status
kubectl get pods -n production -l app=myapp

# All pods should be Running with READY 1/1
]]></command>
<expected_result>All pods in Running state with readiness probe passing</expected_result>
</aks_check>
</check>

<check name="health_endpoint">
<title>Test Health Endpoint</title>
<description>Verify health check endpoint returns successful response</description>
<command><![CDATA[
# Test health endpoint (should return HTTP 200)
curl -f https://myapp-prod-eastus-app-001.azurewebsites.net/health || {
  echo "ERROR: Health check failed";
  exit 1;
}

# With verbose output to see response details
curl -v https://myapp-prod-eastus-app-001.azurewebsites.net/health
]]></command>
<expected_result>HTTP/2 200 response with healthy status</expected_result>
<failure_action>If health check fails, check application logs immediately and consider rollback</failure_action>
</check>

<check name="api_endpoints">
<title>Test Key API Endpoints</title>
<description>Verify critical API endpoints are accessible and return correct responses</description>
<test_scenarios>
<scenario>
<endpoint>/api/version</endpoint>
<expected>Returns current application version</expected>
</scenario>
<scenario>
<endpoint>/api/status</endpoint>
<expected>Returns service status and uptime</expected>
</scenario>
<scenario>
<endpoint>/api/test</endpoint>
<expected>Returns test data confirming API is functional</expected>
</scenario>
</test_scenarios>

<example_test><![CDATA[
# Test version endpoint
VERSION=$(curl -s https://myapp-prod-eastus-app-001.azurewebsites.net/api/version)
echo "Deployed version: $VERSION"

# Test with authentication if required
curl -H "Authorization: Bearer $TOKEN" \
  https://myapp-prod-eastus-app-001.azurewebsites.net/api/protected
]]></example_test>
</check>

<check name="application_logs">
<title>Check Application Logs</title>
<description>Review recent logs for errors, warnings, or startup issues</description>

<app_service_logs>
<command><![CDATA[
# Stream logs in real-time
az webapp log tail \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-app-001

# Download logs for offline analysis
az webapp log download \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-app-001 \
  --log-file app-logs.zip
]]></command>
</app_service_logs>

<aks_logs>
<command><![CDATA[
# View logs from all pods
kubectl logs -n production -l app=myapp --tail=100 -f

# View logs from specific pod
kubectl logs -n production <pod-name> --tail=100
]]></command>
</aks_logs>

<log_analysis>
<red_flags>
<flag>Exceptions or stack traces</flag>
<flag>Database connection errors</flag>
<flag>Authentication failures</flag>
<flag>Timeout errors</flag>
<flag>Memory or resource errors (OutOfMemoryError, ENOMEM)</flag>
<flag>Dependency failures (third-party API errors)</flag>
</red_flags>

<healthy_indicators>
<indicator>Application started successfully messages</indicator>
<indicator>Successful database connections</indicator>
<indicator>Incoming HTTP requests logged</indicator>
<indicator>No error-level logs in first 5 minutes</indicator>
</healthy_indicators>
</log_analysis>
</check>

<check name="app_insights_telemetry">
<title>Verify Application Insights Telemetry</title>
<description>Confirm Application Insights is receiving telemetry data</description>

<command><![CDATA[
# Check request count in last 5 minutes
az monitor app-insights metrics show \
  --app myapp-prod-eastus-insights-001 \
  --resource-group myapp-prod-eastus-rg \
  --metric "requests/count" \
  --start-time $(date -u -d '5 minutes ago' '+%Y-%m-%dT%H:%M:%SZ') \
  --end-time $(date -u '+%Y-%m-%dT%H:%M:%SZ') \
  --aggregation count

# Check for exceptions
az monitor app-insights metrics show \
  --app myapp-prod-eastus-insights-001 \
  --resource-group myapp-prod-eastus-rg \
  --metric "exceptions/count" \
  --start-time $(date -u -d '5 minutes ago' '+%Y-%m-%dT%H:%M:%SZ') \
  --aggregation count
]]></command>

<portal_verification>
<description>In Azure Portal, navigate to Application Insights and verify:</description>
<checks>
<check>Live Metrics shows active connections and incoming requests</check>
<check>Application Map displays service topology</check>
<check>Failures blade shows zero or minimal failures</check>
<check>Performance blade shows acceptable response times</check>
</checks>
</portal_verification>

<expected_telemetry>
<telemetry>Requests: Incoming HTTP requests logged</telemetry>
<telemetry>Dependencies: Database queries, external API calls tracked</telemetry>
<telemetry>Exceptions: Should be zero or minimal</telemetry>
<telemetry>Custom Events: Application-specific events if instrumented</telemetry>
</expected_telemetry>
</check>

<check name="database_connectivity">
<title>Verify Database Connectivity</title>
<description>Ensure application can connect to and query database</description>

<test_methods>
<method>Check application logs for successful database connection messages</method>
<method>Test API endpoint that requires database access</method>
<method>Review Application Insights dependencies for SQL queries</method>
<method>Check for database connection errors in logs</method>
</test_methods>

<example_test><![CDATA[
# Test endpoint that queries database
curl -s https://myapp-prod-eastus-app-001.azurewebsites.net/api/users | jq .

# Expected: Returns user data from database
# If returns error or empty, check database connectivity
]]></example_test>
</check>

<check name="performance_metrics">
<title>Monitor Performance Metrics</title>
<description>Check CPU, memory, and response times are within acceptable ranges</description>

<metrics_to_monitor>
<metric name="cpu_usage">
<threshold>Should be under 70% average</threshold>
<concern>Above 80% indicates possible under-provisioning</concern>
</metric>

<metric name="memory_usage">
<threshold>Should be under 80% of available memory</threshold>
<concern>Above 90% risks out-of-memory errors</concern>
</metric>

<metric name="response_time">
<threshold>P95 should be under 1000ms (1 second)</threshold>
<concern>Above 2000ms indicates performance degradation</concern>
</metric>

<metric name="error_rate">
<threshold>Should be under 1% of requests</threshold>
<concern>Above 5% indicates serious issues</concern>
</metric>
</metrics_to_monitor>

<monitoring_commands>
<app_service><![CDATA[
# View metrics in Azure Portal or via CLI
az monitor metrics list \
  --resource /subscriptions/<subscription-id>/resourceGroups/myapp-prod-eastus-rg/providers/Microsoft.Web/sites/myapp-prod-eastus-app-001 \
  --metric "CpuPercentage" "MemoryPercentage" "HttpResponseTime" \
  --start-time $(date -u -d '10 minutes ago' '+%Y-%m-%dT%H:%M:%SZ')
]]></app_service>

<aks><![CDATA[
# Check pod resource usage
kubectl top pods -n production -l app=myapp

# View metrics in Azure Monitor or Prometheus (if configured)
]]></aks>
</monitoring_commands>
</check>
</verification_checks>

<automated_verification_script>
<title>Comprehensive Verification Script</title>
<description>Script to run all verification checks automatically</description>
<script><![CDATA[
#!/bin/bash
set -e

APP_URL="https://myapp-prod-eastus-app-001.azurewebsites.net"
RG_NAME="myapp-prod-eastus-rg"
APP_NAME="myapp-prod-eastus-app-001"

echo "=== Deployment Verification Started ==="

# 1. Check app status
echo "[1/6] Checking application status..."
STATE=$(az webapp show --resource-group $RG_NAME --name $APP_NAME --query "state" -o tsv)
if [ "$STATE" != "Running" ]; then
  echo "ERROR: App is not running (state: $STATE)"
  exit 1
fi
echo "✓ Application is running"

# 2. Test health endpoint
echo "[2/6] Testing health endpoint..."
HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" $APP_URL/health)
if [ "$HTTP_CODE" != "200" ]; then
  echo "ERROR: Health check failed (HTTP $HTTP_CODE)"
  exit 1
fi
echo "✓ Health endpoint returns 200"

# 3. Check logs for errors
echo "[3/6] Checking recent logs..."
ERROR_COUNT=$(az webapp log tail --resource-group $RG_NAME --name $APP_NAME --logs 50 2>&1 | grep -i error | wc -l)
if [ "$ERROR_COUNT" -gt 5 ]; then
  echo "WARNING: Found $ERROR_COUNT errors in recent logs"
fi
echo "✓ Log check complete"

# 4. Test API endpoint
echo "[4/6] Testing API endpoint..."
curl -f -s $APP_URL/api/version > /dev/null || { echo "ERROR: API test failed"; exit 1; }
echo "✓ API endpoint accessible"

# 5. Check Application Insights
echo "[5/6] Verifying Application Insights..."
REQUEST_COUNT=$(az monitor app-insights metrics show \
  --app myapp-prod-eastus-insights-001 \
  --resource-group $RG_NAME \
  --metric "requests/count" \
  --start-time $(date -u -d '5 minutes ago' '+%Y-%m-%dT%H:%M:%SZ') \
  --aggregation count --query "value.count" -o tsv)
if [ "$REQUEST_COUNT" -gt 0 ]; then
  echo "✓ Application Insights receiving telemetry ($REQUEST_COUNT requests)"
else
  echo "WARNING: No requests logged in Application Insights yet"
fi

# 6. Performance check
echo "[6/6] Checking performance metrics..."
CPU=$(az monitor metrics list \
  --resource /subscriptions/<sub-id>/resourceGroups/$RG_NAME/providers/Microsoft.Web/sites/$APP_NAME \
  --metric "CpuPercentage" \
  --start-time $(date -u -d '5 minutes ago' '+%Y-%m-%dT%H:%M:%SZ') \
  --aggregation average --query "value[0].timeseries[0].data[-1].average" -o tsv)
echo "✓ Current CPU: ${CPU}%"

echo ""
echo "=== Deployment Verification Complete ==="
echo "✓ All checks passed - deployment successful!"
]]></script>
</automated_verification_script>
</step>

<step number="6" name="monitor_post_deployment">
<title>Monitor Post-Deployment Health</title>
<description>
Actively monitor the application for 15-30 minutes after deployment to catch any issues that may not be immediately apparent. Watch for error rate spikes, performance degradation, or resource constraints.
</description>

<monitoring_duration>
<recommendation>Monitor for 15-30 minutes after production deployment</recommendation>
<extended_monitoring>For critical deployments, extend monitoring to 1-2 hours</extended_monitoring>
</monitoring_duration>

<monitoring_checklist>
<metric name="error_rate">
<description>Monitor error rate in Application Insights</description>
<healthy_threshold>Less than 1% of requests fail</healthy_threshold>
<warning_threshold>1-5% error rate</warning_threshold>
<critical_threshold>Above 5% error rate - consider rollback</critical_threshold>
<monitoring_location>Application Insights → Failures blade</monitoring_location>
</metric>

<metric name="response_time">
<description>Monitor average and P95 response times</description>
<healthy_threshold>P95 under 1000ms</healthy_threshold>
<warning_threshold>P95 1000-2000ms</warning_threshold>
<critical_threshold>P95 above 2000ms or 2x normal baseline</critical_threshold>
<monitoring_location>Application Insights → Performance blade</monitoring_location>
</metric>

<metric name="cpu_memory_usage">
<description>Monitor resource utilization</description>
<healthy_threshold>CPU under 70%, Memory under 80%</healthy_threshold>
<warning_threshold>CPU 70-85%, Memory 80-90%</warning_threshold>
<critical_threshold>CPU above 85%, Memory above 90% (risk of OOM)</critical_threshold>
<monitoring_location>Azure Portal → App Service → Metrics</monitoring_location>
</metric>

<metric name="failed_requests">
<description>Monitor for HTTP 5xx errors</description>
<healthy_threshold>Zero or minimal 5xx errors</healthy_threshold>
<warning_threshold>Sporadic 5xx errors (under 10/hour)</warning_threshold>
<critical_threshold>Sustained 5xx errors or increasing trend</critical_threshold>
<monitoring_location>Application Insights → Failures</monitoring_location>
</metric>

<metric name="dependency_failures">
<description>Monitor calls to databases, external APIs, storage</description>
<healthy_threshold>All dependencies healthy, no failures</healthy_threshold>
<warning_threshold>Intermittent dependency failures</warning_threshold>
<critical_threshold>Sustained dependency failures (database unreachable, API timeouts)</critical_threshold>
<monitoring_location>Application Insights → Dependencies</monitoring_location>
</metric>

<metric name="exceptions">
<description>Monitor unhandled exceptions in application</description>
<healthy_threshold>Zero or very few exceptions</healthy_threshold>
<warning_threshold>Occasional exceptions (under 10/hour)</warning_threshold>
<critical_threshold>Exception storm or recurring exception pattern</critical_threshold>
<monitoring_location>Application Insights → Failures → Exceptions</monitoring_location>
</metric>
</monitoring_checklist>

<red_flags>
<title>Critical Issues Requiring Immediate Action</title>
<description>If any of these occur, consider immediate rollback</description>

<flag severity="critical">
<issue>Error rate spike above 5%</issue>
<action>Rollback immediately - application is failing for significant percentage of users</action>
</flag>

<flag severity="critical">
<issue>Application not responding (502/503 errors)</issue>
<action>Check if application crashed; rollback if issue persists beyond 2 minutes</action>
</flag>

<flag severity="critical">
<issue>Database connection failures</issue>
<action>Verify database is accessible; check Managed Identity permissions; rollback if unresolvable</action>
</flag>

<flag severity="critical">
<issue>Memory usage spiking above 95%</issue>
<action>Risk of out-of-memory crash; check for memory leak; rollback if issue persists</action>
</flag>

<flag severity="high">
<issue>Response time degradation (2x normal)</issue>
<action>Investigate performance issue; prepare rollback if not resolvable quickly</action>
</flag>

<flag severity="high">
<issue>Sustained exceptions or error patterns</issue>
<action>Review exception details; rollback if exceptions are widespread and not handled</action>
</flag>
</red_flags>

<rollback_decision_criteria>
<title>When to Rollback</title>
<description>Decision framework for determining if rollback is necessary</description>

<immediate_rollback>
<situation>Application completely down (all requests failing)</situation>
<situation>Data corruption or data loss risk</situation>
<situation>Security vulnerability exposed</situation>
<situation>Critical business function broken (e.g., payments failing)</situation>
</immediate_rollback>

<rollback_within_5_minutes>
<situation>Error rate sustained above 5% for 2+ minutes</situation>
<situation>Performance degradation 3x worse than baseline</situation>
<situation>Critical dependency failures (database, authentication, payment gateway)</situation>
</rollback_within_5_minutes>

<rollback_within_15_minutes>
<situation>Error rate 2-5% that's not decreasing</situation>
<situation>Intermittent failures affecting user experience</situation>
<situation>Performance issues not explained or improving</situation>
</rollback_within_15_minutes>

<fix_forward>
<situation>Minor cosmetic issues (UI glitches, non-critical features)</situation>
<situation>Issues that can be quickly fixed with configuration change</situation>
<situation>Issues affecting only small percentage of users or specific edge cases</situation>
</fix_forward>
</rollback_decision_criteria>

<rollback_procedures>
<app_service_rollback>
<title>App Service Rollback (Swap Back)</title>
<description>If deployed via staging slot, swap back instantly</description>
<command><![CDATA[
# Swap staging and production (previous version now in staging)
az webapp deployment slot swap \
  --resource-group myapp-prod-eastus-rg \
  --name myapp-prod-eastus-app-001 \
  --slot staging \
  --target-slot production

# Rollback completes in under 1 second
echo "Rollback complete - production restored to previous version"
]]></command>
</app_service_rollback>

<aks_rollback>
<title>AKS Rollback</title>
<command><![CDATA[
# Rollback to previous revision
kubectl rollout undo deployment/myapp -n production

# Monitor rollback progress
kubectl rollout status deployment/myapp -n production
]]></command>
</aks_rollback>

<container_apps_rollback>
<title>Container Apps Rollback</title>
<command><![CDATA[
# Activate previous revision
az containerapp revision activate \
  --name myapp-prod-eastus-ca-001 \
  --resource-group myapp-prod-eastus-rg \
  --revision <previous-revision-name>
]]></command>
</container_apps_rollback>
</rollback_procedures>

<monitoring_tools>
<tool name="application_insights_live_metrics">
<description>Real-time metrics and request tracking</description>
<access>Azure Portal → Application Insights → Live Metrics</access>
<useful_for>Seeing immediate impact of deployment, tracking live traffic</useful_for>
</tool>

<tool name="log_streaming">
<description>Real-time log output from application</description>
<command>az webapp log tail --resource-group &lt;rg&gt; --name &lt;app&gt;</command>
<useful_for>Catching errors, warnings, and application behavior in real-time</useful_for>
</tool>

<tool name="azure_monitor_metrics">
<description>Infrastructure metrics (CPU, memory, network)</description>
<access>Azure Portal → App Service → Metrics</access>
<useful_for>Detecting resource constraints, performance bottlenecks</useful_for>
</tool>
</monitoring_tools>
</step>
</process>

<anti_patterns>
<title>Deployment Anti-Patterns to Avoid</title>
<description>
Common mistakes that lead to failed deployments, downtime, or operational issues. Learn from these anti-patterns to ensure successful, safe deployments.
</description>

<anti_pattern name="direct_production_deployment">
<mistake>Deploying directly to production without staging slot or testing</mistake>
<why_bad>
- Causes downtime during deployment
- No way to test before going live
- Difficult to rollback (must redeploy previous version)
- Users experience any deployment issues immediately
- No safety net
</why_bad>
<correct_approach>
Always use staging slots (App Service/Functions) or blue-green deployment (AKS). Deploy to staging, test thoroughly, then swap/promote to production.
</correct_approach>
<reference>references/deployment-strategies.md</reference>
</anti_pattern>

<anti_pattern name="no_health_endpoint">
<mistake>Application has no health check endpoint</mistake>
<why_bad>
- Cannot verify deployment succeeded
- Load balancers cannot determine pod/instance health
- Azure cannot detect when application is ready to serve traffic
- Deployments may route traffic to unhealthy instances
- Difficult to debug startup issues
</why_bad>
<correct_approach>
Always implement /health endpoint that returns HTTP 200 when healthy, 503 when unhealthy. Check dependencies (database, critical services).
</correct_approach>
</anti_pattern>

<anti_pattern name="hardcoded_configuration">
<mistake>Hardcoding configuration values in code or using appsettings.json for environment-specific values</mistake>
<why_bad>
- Different configuration per environment requires different builds
- Secrets committed to source control (security risk)
- Cannot change configuration without redeployment
- Configuration drift between environments
</why_bad>
<correct_approach>
Use environment variables for all configuration. Store secrets in Azure Key Vault and reference via app settings. Configuration should be external to code.
</correct_approach>
<reference>references/security-identity.md</reference>
</anti_pattern>

<anti_pattern name="no_testing_after_deployment">
<mistake>Not testing application after deployment</mistake>
<why_bad>
- Issues go undetected until users report them
- No verification that deployment succeeded
- Cannot confidently promote to production
- Reactive instead of proactive problem detection
</why_bad>
<correct_approach>
Always run automated tests after deployment: health checks, API tests, smoke tests. Monitor Application Insights for errors. Verify core functionality.
</correct_approach>
</anti_pattern>

<anti_pattern name="no_rollback_plan">
<mistake>No documented or tested rollback procedure</mistake>
<why_bad>
- When issues occur, team scrambles to figure out how to rollback
- Rollback takes too long, extending user impact
- May make situation worse with incorrect rollback attempt
- Panic leads to mistakes
</why_bad>
<correct_approach>
Document and test rollback procedure before deploying. Know exactly how to rollback (slot swap, kubectl rollback, revision activation). Practice during staging deployments.
</correct_approach>
</anti_pattern>

<anti_pattern name="deploying_during_peak_hours">
<mistake>Deploying to production during peak traffic times</mistake>
<why_bad>
- If issues occur, maximum number of users affected
- Increased load makes troubleshooting harder
- Higher risk of performance issues under load
- Customer-facing problems at worst possible time
</why_bad>
<correct_approach>
Deploy during low-traffic windows (early morning, weekends, planned maintenance windows). For 24/7 applications, use canary deployments or blue-green with traffic shifting.
</correct_approach>
</anti_pattern>

<anti_pattern name="no_monitoring_post_deployment">
<mistake>Deploying and walking away without monitoring</mistake>
<why_bad>
- Issues develop unnoticed
- Small problems escalate into major incidents
- Missed opportunity to catch and fix issues early
- Users discover problems before you do
</why_bad>
<correct_approach>
Actively monitor for 15-30 minutes post-deployment. Watch error rates, response times, logs. Set up alerts for anomalies. Have engineer on-call monitoring dashboards.
</correct_approach>
</anti_pattern>

<anti_pattern name="incomplete_deployment">
<mistake>Deploying application code without updating configuration or dependencies</mistake>
<why_bad>
- Application expects configuration that doesn't exist
- Missing environment variables cause startup failures
- Database migrations not run
- Dependencies out of sync
</why_bad>
<correct_approach>
Deployment checklist: Update configuration, run database migrations, verify dependencies, update secrets, configure monitoring. Deploy as complete package.
</correct_approach>
</anti_pattern>

<anti_pattern name="ignoring_warnings">
<mistake>Proceeding with deployment despite warnings in build or validation</mistake>
<why_bad>
- Warnings often indicate real problems
- "It worked locally" leads to production failures
- Technical debt accumulates
- Debugging is harder in production
</why_bad>
<correct_approach>
Address all warnings before deploying. Run linters, validators, security scans. Fix issues in dev/staging, not production.
</correct_approach>
</anti_pattern>
</anti_patterns>

<success_criteria>
<title>Definition of Successful Deployment</title>
<description>
A successful deployment meets all these criteria. Use this checklist to verify deployment completion before considering it done.
</description>

<criteria>
<criterion name="zero_downtime">
<check>Deployment completed with zero downtime (no user-facing impact)</check>
<verification>No 502/503 errors during deployment window; users experienced no service interruption</verification>
<applies_to>Production deployments with staging slots or blue-green strategy</applies_to>
</criterion>

<criterion name="staging_tested">
<check>Application deployed to staging environment first and tested</check>
<verification>Staging health checks passed, smoke tests executed successfully, no errors in staging logs</verification>
<applies_to>All production deployments</applies_to>
</criterion>

<criterion name="health_checks_passing">
<check>Health endpoint returns HTTP 200 consistently</check>
<verification>curl -f https://app-url/health returns 200; Azure health probes show healthy</verification>
<applies_to>All deployments</applies_to>
</criterion>

<criterion name="swapped_to_production">
<check>Application successfully swapped/promoted to production</check>
<verification>Production URL serves new version; version endpoint returns expected version number</verification>
<applies_to>Production deployments</applies_to>
</criterion>

<criterion name="telemetry_flowing">
<check>Application Insights receiving telemetry from new deployment</check>
<verification>Live Metrics shows active connections; requests logged in last 5 minutes; no telemetry gaps</verification>
<applies_to>All environments with Application Insights</applies_to>
</criterion>

<criterion name="no_error_spikes">
<check>Error rate remains stable (no spike above baseline)</check>
<verification>Application Insights Failures blade shows error rate under 1%; no exception storms</verification>
<applies_to>All deployments</applies_to>
</criterion>

<criterion name="performance_acceptable">
<check>Response times within acceptable range (no degradation)</check>
<verification>P95 response time under 1000ms or within 10% of pre-deployment baseline</verification>
<applies_to>All deployments</applies_to>
</criterion>

<criterion name="logs_clean">
<check>No errors or warnings in application logs post-deployment</check>
<verification>Recent logs show successful startup, no exceptions, no configuration errors</verification>
<applies_to>All deployments</applies_to>
</criterion>

<criterion name="dependencies_healthy">
<check>All dependencies accessible and functioning</check>
<verification>Application Insights Dependencies shows successful calls to database, storage, external APIs; no dependency failures</verification>
<applies_to>All deployments</applies_to>
</criterion>

<criterion name="configuration_correct">
<check>Application configuration loaded correctly</check>
<verification>Environment variables accessible to application; Key Vault references resolved; no configuration-related errors</verification>
<applies_to>All deployments</applies_to>
</criterion>

<criterion name="rollback_available">
<check>Rollback procedure available and tested</check>
<verification>Previous version still in staging slot or previous K8s revision available; rollback command tested and ready</verification>
<applies_to>Production deployments</applies_to>
</criterion>

<criterion name="monitoring_active">
<check>Post-deployment monitoring completed for 15-30 minutes</check>
<verification>Metrics stable, no anomalies detected, error rates normal, performance acceptable</verification>
<applies_to>Production deployments</applies_to>
</criterion>

<criterion name="documentation_updated">
<check>Deployment documented (version deployed, who deployed, when)</check>
<verification>Git tag created, release notes written, deployment log updated</verification>
<applies_to>Production deployments</applies_to>
</criterion>
</criteria>

<final_validation>
All criteria above must be true. If any criterion fails, investigate immediately and consider rollback if issue cannot be quickly resolved.
</final_validation>
</success_criteria>

<next_steps>
<title>Recommended Next Steps After Deployment</title>
<description>
After successfully deploying application code, proceed with these workflows to ensure ongoing operational excellence.
</description>

<next_workflow priority="1">
<name>Setup CI/CD Pipeline</name>
<file>workflows/setup-cicd-pipeline.md</file>
<description>Automate deployments with GitHub Actions or Azure DevOps to eliminate manual deployment steps and ensure consistency</description>
<when>After first successful manual deployment</when>
</next_workflow>

<next_workflow priority="2">
<name>Configure Comprehensive Monitoring</name>
<file>workflows/setup-monitoring.md</file>
<description>Setup alerts, dashboards, and proactive monitoring beyond basic Application Insights to catch issues before users report them</description>
<when>Immediately after production deployment</when>
</next_workflow>

<next_workflow priority="3">
<name>Implement Additional Security</name>
<file>workflows/secure-infrastructure.md</file>
<description>Add network isolation, private endpoints, enhanced authentication, security scanning</description>
<when>Before handling production traffic</when>
</next_workflow>

<next_workflow priority="4">
<name>Optimize Costs</name>
<file>workflows/optimize-costs.md</file>
<description>Review resource utilization, right-size services, implement auto-scaling to optimize spending</description>
<when>After 1-2 weeks of production traffic data</when>
</next_workflow>

<next_workflow priority="5">
<name>Implement Disaster Recovery</name>
<file>workflows/implement-disaster-recovery.md</file>
<description>Setup backup strategy, multi-region deployment, disaster recovery procedures</description>
<when>Before business-critical production launch</when>
</next_workflow>
</next_steps>
