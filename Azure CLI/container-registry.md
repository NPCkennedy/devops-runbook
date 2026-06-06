# Container Registry (ACR)

**Docs:** [az acr](https://learn.microsoft.com/en-us/cli/azure/acr?view=azure-cli-latest)

---

## What is ACR?

Azure Container Registry stores Docker images privately. Every image pushed to ACR is tagged and versioned. The CI/CD pipeline builds images and pushes them here. Container Apps pull images from here to deploy.

```
Developer pushes code
→ Pipeline builds Docker image
→ Tags it with git SHA (e.g. sha-08643f...)
→ Pushes to ACR
→ Container Apps pulls from ACR and deploys
```

**Always use SHA tags — never :latest.**
SHA tags are permanent and traceable. :latest gets overwritten on every push — you lose the ability to roll back.

---

## Create

**What:** Creates a new container registry.
**When:** Once per project — stores all Docker images.

```bash
az acr create \
  --name <name> \
  --resource-group <rg> \
  --sku Basic
```

| Flag | Required | Description |
|------|----------|-------------|
| --name | Yes | Registry name — must be globally unique, alphanumeric only |
| --resource-group | Yes | Resource group to place it in |
| --sku | Yes | Pricing tier: Basic, Standard, Premium |

**SKU comparison:**
| SKU | Use case |
|-----|----------|
| Basic | Dev/demo — low storage, no geo-replication |
| Standard | Production — more storage, webhooks |
| Premium | Enterprise — geo-replication, private endpoints |

**Example:**
```bash
az acr create --name zenecr2026 --resource-group zen-rg-dev --sku Basic
```

**Note:** ACR names are globally unique across all Azure customers. If your name is taken, add a suffix (e.g. zenecr2026b).

---

## Login

**What:** Authenticates Docker to push/pull from the registry.
**When:** Before running docker push or docker pull locally.

```bash
az acr login --name <name>
```

---

## List Repositories

**What:** Lists all image repositories in the registry.
**When:** To see what images exist in ACR.

```bash
az acr repository list --name <name> -o table
```

---

## List Tags (Images)

**What:** Lists all tagged images for a specific repository.
**When:** To find the latest SHA tag before a rollback or terraform apply.

```bash
az acr repository show-tags \
  --name <name> \
  --repository <repo> \
  --orderby time_desc \
  --top 5
```

**Example:**
```bash
az acr repository show-tags \
  --name zenecr2026 \
  --repository backend \
  --orderby time_desc \
  --top 3
```

---

## Get Credentials

**What:** Returns the admin username and password for the registry.
**When:** When configuring Terraform or updating secrets after credential rotation.

```bash
az acr credential show --name <name>
```

Get just the password:
```bash
az acr credential show --name <name> --query "passwords[0].value" -o tsv
```

---

## Rotate Credentials

**What:** Generates a new password for the registry, invalidating the old one.
**When:** After accidental credential exposure.

```bash
az acr credential renew --name <name> --password-name password
```

**Note:** After rotating, update the new password in: Terraform tfvars, GitHub Secrets (if used), and any local .env files.

---

## Grant Pull Access to Service Principal

**What:** Gives a service principal permission to pull images from ACR.
**When:** When setting up a pipeline or Container App to pull images.

```bash
az role assignment create \
  --assignee <service-principal-id> \
  --role AcrPull \
  --scope $(az acr show --name <name> --query id -o tsv)
```
