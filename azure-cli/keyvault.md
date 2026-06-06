# Key Vault

**Docs:** [az keyvault](https://learn.microsoft.com/en-us/cli/azure/keyvault?view=azure-cli-latest)

---

## What is Key Vault?

A secure store for secrets, keys, and certificates. Applications read secrets at runtime via Managed Identity — no passwords ever touch the codebase or environment files.

```
Application starts on Azure
→ Authenticates to Key Vault using Managed Identity
→ Reads DATABASE_URL, API_KEY, etc.
→ Uses them in memory — never written to disk
```

**Zero-secrets model:** Nothing sensitive in code, git history, or environment files. Everything lives in Key Vault.

---

## Create

**What:** Creates a new Key Vault.
**When:** Once per environment.

```bash
az keyvault create \
  --name <vault-name> \
  --resource-group <rg> \
  --location <region>
```

| Flag | Required | Description |
|------|----------|-------------|
| --name | Yes | Vault name — globally unique, 3-24 chars, alphanumeric and hyphens |
| --resource-group | Yes | Resource group |
| --location | Yes | Azure region |

**Example:**
```bash
az keyvault create \
  --name zen-kv-dev \
  --resource-group zen-rg-dev \
  --location southeastasia
```

---

## Set a Secret

**What:** Stores a secret in the vault.
**When:** Adding a new credential or rotating an existing one.

```bash
az keyvault secret set \
  --vault-name <vault-name> \
  --name <secret-name> \
  --value <secret-value>
```

**Example:**
```bash
az keyvault secret set \
  --vault-name zen-kv-dev \
  --name DATABASE-URL \
  --value "postgresql://user:password@server.postgres.database.azure.com:5432/db?sslmode=require"
```

**Note:** Secret names can only contain alphanumeric characters and hyphens.

---

## Get a Secret

**What:** Retrieves a secret value from the vault.
**When:** Verifying a secret was stored correctly.

```bash
az keyvault secret show \
  --vault-name <vault-name> \
  --name <secret-name> \
  --query "value" \
  -o tsv
```

---

## List Secrets

**What:** Lists all secret names in the vault (not their values).
**When:** Auditing what is stored in the vault.

```bash
az keyvault secret list \
  --vault-name <vault-name> \
  --query "[].name" \
  -o table
```

---

## Grant Access via RBAC

**What:** Gives a Managed Identity permission to read secrets.
**When:** Wiring a Container App to read from Key Vault.

```bash
az role assignment create \
  --assignee <managed-identity-client-id> \
  --role "Key Vault Secrets User" \
  --scope $(az keyvault show --name <vault-name> --query id -o tsv)
```

**Roles:**
| Role | Permission |
|------|-----------|
| Key Vault Secrets User | Read secrets only |
| Key Vault Secrets Officer | Read and write secrets |
| Key Vault Administrator | Full access |

**Principle of least privilege:** Give each container only the role it needs. A frontend that doesn't need secrets should have no Key Vault access at all.

---

## Delete a Secret

**What:** Soft-deletes a secret (recoverable for 90 days by default).
**When:** Rotating credentials — delete old, set new.

```bash
az keyvault secret delete \
  --vault-name <vault-name> \
  --name <secret-name>
```
