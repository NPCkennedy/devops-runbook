# Terraform Remote State (Azure Backend)

**Docs:** [Azure Backend](https://developer.hashicorp.com/terraform/language/backend/azurerm) | [Store state in Azure Storage](https://learn.microsoft.com/en-us/azure/developer/terraform/store-state-in-azure-storage)

---

## What is Remote State?

By default Terraform stores state in a local `terraform.tfstate` file. Remote state stores it in Azure Blob Storage instead.

**Why remote state matters for teams:**
```
Local state    → only you can apply, no locking, gets lost
Remote state   → team shares one state, locked during apply,
                 survives machine loss, auditable history
```

**State locking:** When one person runs `terraform apply`, the state file is locked. Anyone else who tries to apply at the same time gets blocked until the first apply finishes. Prevents state corruption.

---

## Step 1 — Create the Backend Infrastructure

Run these commands once before any Terraform init. These resources are created manually — not by Terraform — because Terraform needs them to exist before it can store state.

```bash
# Create dedicated resource group for state
az group create \
  --name zen-tfstate-rg \
  --location southeastasia

# Create storage account (name must be globally unique)
az storage account create \
  --name zentfstate2026 \
  --resource-group zen-tfstate-rg \
  --location southeastasia \
  --sku Standard_LRS \
  --kind StorageV2 \
  --min-tls-version TLS1_2 \
  --allow-blob-public-access false

# Create container inside the storage account
az storage container create \
  --name tfstate \
  --account-name zentfstate2026 \
  --auth-mode login
```

**Why a separate resource group?**
If dev or prod resource groups are deleted, the Terraform state survives. Always keep state in its own isolated resource group.

---

## Step 2 — Add Delete Lock (Recommended)

Prevents the Terraform state resource group from being accidentally deleted.

```bash
az group lock create \
  --name "protect-tfstate" \
  --lock-type CanNotDelete \
  --resource-group zen-tfstate-rg
```

---

## Step 3 — Configure Terraform Backend

Add this to your `terraform/backend.tf` or inside the `terraform {}` block in `main.tf`:

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "zen-tfstate-rg"
    storage_account_name = "zentfstate2026"
    container_name       = "tfstate"
    key                  = "terraform.tfstate"
  }
}
```

| Field | Description |
|-------|-------------|
| resource_group_name | Resource group containing the storage account |
| storage_account_name | Name of the storage account |
| container_name | Blob container name |
| key | Filename for the state file inside the container |

---

## Step 4 — Initialise

```bash
terraform init
```

Terraform connects to the Azure Storage backend and downloads the existing state (or creates a new one).

---

## Verify State File Exists

```bash
az storage blob list \
  --account-name zentfstate2026 \
  --container-name tfstate \
  --auth-mode login \
  -o table
```

You should see `terraform.tfstate` listed.

---

## Recover from Lost State

If the state storage account was deleted:

1. Recreate the storage account and container (Step 1 above)
2. Update backend config if the account name changed
3. Run `terraform init`
4. Terraform will create a fresh empty state
5. Re-import existing resources with `terraform import` or run `terraform apply` to recreate everything

**Prevention:** Always add a delete lock to the tfstate resource group.
