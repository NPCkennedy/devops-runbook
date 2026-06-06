# Container Apps

**Docs:** [az containerapp](https://learn.microsoft.com/en-us/cli/azure/containerapp?view=azure-cli-latest)

---

## What is Azure Container Apps?

A fully managed platform for running containerised applications. No Kubernetes cluster to manage. Auto-scales based on traffic — scales to zero when idle (no cost).

```
Your Docker image in ACR
→ Container Apps pulls it
→ Runs it at a public HTTPS URL
→ Scales automatically based on traffic
→ Scales to zero when nobody is hitting it
```

**Container Apps vs AKS:**
| | Container Apps | AKS |
|--|----------------|-----|
| Management | Fully managed | You manage the cluster |
| Cost | Pay per use, scales to zero | Fixed cluster cost |
| Use case | Demo, simple microservices | Production, complex workloads |
| Complexity | Low | High |

---

## Install Extension

**What:** Installs the Container Apps CLI extension.
**When:** First time using Container Apps commands on a new machine.

```bash
az extension add --name containerapp --upgrade
```

---

## Create Environment

**What:** Creates a Container Apps environment — a shared boundary for multiple containers.
**When:** Once per deployment environment (dev, prod). All containers in the same environment share networking.

```bash
az containerapp env create \
  --name <env-name> \
  --resource-group <rg> \
  --location <region>
```

**Example:**
```bash
az containerapp env create \
  --name zen-container-env-dev \
  --resource-group zen-rg-dev \
  --location southeastasia
```

---

## Create Container App

**What:** Deploys a container from ACR into the environment.
**When:** First deployment of a new container.

```bash
az containerapp create \
  --name <name> \
  --resource-group <rg> \
  --environment <env-name> \
  --image <acr-login-server>/<repo>:<tag> \
  --registry-server <acr-login-server> \
  --registry-username <acr-username> \
  --registry-password <acr-password> \
  --target-port <port> \
  --ingress external
```

---

## Update Image (Rollback / Deploy New Version)

**What:** Updates the container to run a different image tag.
**When:** Rolling back to a previous version or deploying a hotfix manually.

```bash
az containerapp update \
  --name <name> \
  --resource-group <rg> \
  --image <acr-login-server>/<repo>:<sha-tag>
```

**Example — rollback frontend:**
```bash
az containerapp update \
  --name zen-frontend-dev \
  --resource-group zen-rg-dev \
  --image zenecr2026.azurecr.io/frontend:sha-08643f958d4493fa6ef630b3c87d2384ed65ccb2
```

**Note:** This is how you roll back in under 2 minutes. Always keep the last known good SHA noted before any deployment.

---

## Show (Get FQDN / URL)

**What:** Shows details of a container app including its public URL.
**When:** To get the live URL of a deployed container.

```bash
az containerapp show \
  --name <name> \
  --resource-group <rg> \
  --query "properties.configuration.ingress.fqdn" \
  -o tsv
```

---

## List

**What:** Lists all container apps in a resource group.
**When:** To see what is deployed.

```bash
az containerapp list \
  --resource-group <rg> \
  --query "[].{name:name, fqdn:properties.configuration.ingress.fqdn}" \
  -o table
```

---

## View Logs

**What:** Streams live logs from a running container.
**When:** Debugging a container that is failing or behaving unexpectedly.

```bash
az containerapp logs show \
  --name <name> \
  --resource-group <rg> \
  --follow
```

---

## Set Environment Variable

**What:** Adds or updates an environment variable on a running container.
**When:** Changing config without redeploying the image.

```bash
az containerapp update \
  --name <name> \
  --resource-group <rg> \
  --set-env-vars KEY=VALUE
```
