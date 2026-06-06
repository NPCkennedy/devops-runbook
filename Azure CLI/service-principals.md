# Service Principals & RBAC

**Docs:** [az ad sp](https://learn.microsoft.com/en-us/cli/azure/ad/sp?view=azure-cli-latest) | [az role assignment](https://learn.microsoft.com/en-us/cli/azure/role/assignment?view=azure-cli-latest)

---

## What is a Service Principal?

A non-human identity that automated systems use to authenticate to Azure. Think of it as a robot employee account with specific permissions.

```
GitHub Actions (the worker)
→ uses Service Principal (the ID card)
→ to authenticate to Azure
→ and perform specific actions (push to ACR, deploy containers)
```

**Service Principal vs Managed Identity:**
| | Service Principal | Managed Identity |
|--|------------------|-----------------|
| Used by | External systems (GitHub Actions, Terraform) | Azure resources (Container Apps, VMs) |
| Credentials | Client ID + Secret (must be managed) | No credentials — Azure handles auth automatically |
| Use case | CI/CD pipelines, local dev | App-to-Azure service communication |

---

## Check if Service Principal Exists

**What:** Lists service principals matching a display name.
**When:** Verifying a service principal exists after infrastructure deletion.

```bash
az ad sp list \
  --display-name "<name>" \
  --query "[].{name:displayName, id:appId}" \
  -o table
```

---

## Create Service Principal

**What:** Creates a new service principal with Contributor role on a subscription.
**When:** Setting up a new CI/CD pipeline that needs to deploy to Azure.

```bash
az ad sp create-for-rbac \
  --name "<name>" \
  --role Contributor \
  --scopes /subscriptions/<subscription-id> \
  --json-auth
```

| Flag | Required | Description |
|------|----------|-------------|
| --name | Yes | Display name for the service principal |
| --role | Yes | Azure role to assign |
| --scopes | Yes | What the SP has access to (subscription or resource group) |
| --json-auth | No | Output in JSON format for use in GitHub Secrets |

**Save the output immediately** — the client secret is only shown once.

**Store the output as AZURE_CREDENTIALS in GitHub Secrets.**

---

## List Role Assignments

**What:** Shows what permissions a service principal or identity has.
**When:** Auditing access or troubleshooting permission errors.

```bash
az role assignment list \
  --assignee <service-principal-id-or-email> \
  --query "[].{role:roleDefinitionName, scope:scope}" \
  -o table
```

---

## Assign Role

**What:** Grants a role to a service principal or Managed Identity on a specific resource.
**When:** Giving a container app access to Key Vault, or a pipeline access to ACR.

```bash
az role assignment create \
  --assignee <principal-id> \
  --role "<role-name>" \
  --scope <resource-id>
```

**Common roles:**
| Role | What it allows |
|------|---------------|
| Contributor | Create, update, delete resources |
| Reader | Read only |
| AcrPull | Pull images from ACR |
| AcrPush | Push images to ACR |
| Key Vault Secrets User | Read secrets from Key Vault |
| Key Vault Secrets Officer | Read and write secrets |

**Example — give pipeline AcrPush:**
```bash
az role assignment create \
  --assignee <service-principal-id> \
  --role AcrPush \
  --scope $(az acr show --name <acr-name> --query id -o tsv)
```

---

## Create Managed Identity

**What:** Creates a user-assigned Managed Identity for a container or resource.
**When:** When a Container App needs to access Key Vault or ACR without hardcoded credentials.

```bash
az identity create \
  --name <identity-name> \
  --resource-group <rg>
```

Get the client ID (needed for role assignments):
```bash
az identity show \
  --name <identity-name> \
  --resource-group <rg> \
  --query "clientId" \
  -o tsv
```
