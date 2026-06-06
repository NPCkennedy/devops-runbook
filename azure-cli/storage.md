# Storage Accounts

**Docs:** [az storage account](https://learn.microsoft.com/en-us/cli/azure/storage/account?view=azure-cli-latest) | [az storage container](https://learn.microsoft.com/en-us/cli/azure/storage/container?view=azure-cli-latest)

---

## What is Azure Blob Storage?

Object storage for unstructured data — images, files, backups, Terraform state files. Cheap, durable, and globally accessible.

**Common uses in a cloud project:**
| Use | What it stores |
|-----|---------------|
| Product images | Static files served via CDN (Front Door) |
| Terraform remote state | .tfstate file shared across the team |
| Application logs | Log exports and backups |

---

## Create Storage Account

**What:** Creates a new storage account.
**When:** For Terraform state backend or application file storage.

```bash
az storage account create \
  --name <account-name> \
  --resource-group <rg> \
  --location <region> \
  --sku Standard_LRS \
  --kind StorageV2 \
  --min-tls-version TLS1_2 \
  --allow-blob-public-access false
```

| Flag | Required | Description |
|------|----------|-------------|
| --name | Yes | Account name — globally unique, 3-24 chars, lowercase alphanumeric only |
| --resource-group | Yes | Resource group |
| --location | Yes | Azure region |
| --sku | Yes | Replication: Standard_LRS (local), Standard_GRS (geo-redundant) |
| --kind | No | StorageV2 is current standard |
| --min-tls-version | No | TLS1_2 minimum for security |
| --allow-blob-public-access | No | false = no public access (recommended) |

**SKU options:**
| SKU | Replication | Use case |
|-----|-------------|----------|
| Standard_LRS | Local (3 copies in one datacenter) | Dev, Terraform state |
| Standard_GRS | Geo-redundant (replicates to paired region) | Production |

---

## Create Blob Container

**What:** Creates a container (folder) inside a storage account.
**When:** After creating the storage account — blobs live inside containers.

```bash
az storage container create \
  --name <container-name> \
  --account-name <account-name> \
  --auth-mode login
```

| Flag | Required | Description |
|------|----------|-------------|
| --name | Yes | Container name |
| --account-name | Yes | Storage account to create it in |
| --auth-mode | No | login = use your Azure AD credentials |

**Example — create Terraform state container:**
```bash
az storage container create \
  --name tfstate \
  --account-name zentfstate2026 \
  --auth-mode login
```

---

## List Containers

**What:** Lists all containers in a storage account.
**When:** To verify containers exist.

```bash
az storage container list \
  --account-name <account-name> \
  --auth-mode login \
  -o table
```

---

## List Blobs

**What:** Lists all blobs (files) in a container.
**When:** To verify Terraform state file exists.

```bash
az storage blob list \
  --account-name <account-name> \
  --container-name <container-name> \
  --auth-mode login \
  -o table
```

---

## Get Storage Account Key

**What:** Returns the access key for a storage account.
**When:** When configuring Terraform backend authentication.

```bash
az storage account keys list \
  --resource-group <rg> \
  --account-name <account-name> \
  --query "[0].value" \
  -o tsv
```
