# App Insights & Log Analytics

**Docs:** [az monitor app-insights](https://learn.microsoft.com/en-us/cli/azure/monitor/app-insights?view=azure-cli-latest) | [az monitor log-analytics](https://learn.microsoft.com/en-us/cli/azure/monitor/log-analytics?view=azure-cli-latest)

---

## What is Application Insights?

A monitoring service that records every request, response time, error, and stack trace from your application in real time.

**App Insights vs /health endpoint:**
| | /health endpoint | App Insights |
|--|-----------------|--------------|
| What it tells you | Is the container alive? (yes/no) | How long did each request take? Which user hit a 500 error? What's the error rate? |
| When it runs | On demand | Continuously, 24/7 |
| Use case | Load balancer health check | Observability, debugging, alerting |

**Always pair with Log Analytics:** App Insights sends telemetry to a Log Analytics workspace for storage and querying.

**Cost:** Free up to 5GB of data per month. A typical demo environment generates under 100MB.

---

## Create Log Analytics Workspace

**What:** Creates the storage backend for telemetry data.
**When:** Before creating App Insights — it depends on this.

```bash
az monitor log-analytics workspace create \
  --workspace-name <workspace-name> \
  --resource-group <rg> \
  --location <region> \
  --sku PerGB2018 \
  --retention-time 30
```

| Flag | Required | Description |
|------|----------|-------------|
| --workspace-name | Yes | Name of the workspace |
| --resource-group | Yes | Resource group |
| --location | Yes | Azure region |
| --sku | No | Pricing tier — PerGB2018 is standard |
| --retention-time | No | Days to keep data (default 30) |

---

## Create App Insights

**What:** Creates an Application Insights resource linked to a Log Analytics workspace.
**When:** After creating the Log Analytics workspace.

```bash
# Get workspace ID first
WORKSPACE_ID=$(az monitor log-analytics workspace show \
  --workspace-name <workspace-name> \
  --resource-group <rg> \
  --query id -o tsv)

# Create App Insights
az monitor app-insights component create \
  --app <app-insights-name> \
  --resource-group <rg> \
  --location <region> \
  --kind web \
  --application-type web \
  --workspace $WORKSPACE_ID
```

---

## Get Connection String

**What:** Returns the connection string to wire into your application.
**When:** When configuring the APPLICATIONINSIGHTS_CONNECTION_STRING environment variable on containers.

```bash
az monitor app-insights component show \
  --app <app-insights-name> \
  --resource-group <rg> \
  --query connectionString \
  -o tsv
```

**How to use it:**
Set `APPLICATIONINSIGHTS_CONNECTION_STRING` as an environment variable on your container. The application SDK picks it up automatically and starts sending telemetry — no code changes needed for basic request tracking.

---

## Verify Telemetry

**What:** Confirms App Insights exists and is receiving data.
**When:** After wiring containers — verify telemetry is flowing.

```bash
az monitor app-insights component show \
  --app <app-insights-name> \
  --resource-group <rg> \
  --query "{name:name, appId:appId}" \
  -o table
```

Then open Azure Portal → resource group → App Insights resource → Overview to see live metrics.

---

## Import Existing Workspace into Terraform

**What:** If a Log Analytics workspace already exists, import it into Terraform state so Terraform manages it.
**When:** After manually creating a workspace that Terraform doesn't know about yet.

```bash
terraform import \
  module.monitoring.azurerm_log_analytics_workspace.main \
  /subscriptions/<subscription-id>/resourceGroups/<rg>/providers/Microsoft.OperationalInsights/workspaces/<workspace-name>
```
