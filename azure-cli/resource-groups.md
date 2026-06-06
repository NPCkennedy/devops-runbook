# Resource Groups

**Docs:** [az group](https://learn.microsoft.com/en-us/cli/azure/group?view=azure-cli-latest)

---

## What is a Resource Group?

A logical container in Azure. Every resource (database, container app, key vault, etc.) must live inside one. Think of it as a folder — delete the folder and everything inside it is deleted too.

Use separate resource groups to isolate environments:
```
project-rg-dev      → development resources
project-rg-prod     → production resources
project-tfstate-rg  → Terraform state (keep separate — survives if other RGs are deleted)
```

---

## Create

**What:** Creates a new resource group.
**When:** Before provisioning any Azure resources.

```bash
az group create \
  --name <name> \
  --location <region>
```

| Flag | Required | Description |
|------|----------|-------------|
| --name | Yes | Name of the resource group |
| --location | Yes | Azure region e.g. southeastasia |

**Example:**
```bash
az group create --name zen-rg-dev --location southeastasia
```

**Common regions:**
| Region | Location |
|--------|----------|
| southeastasia | Singapore |
| eastasia | Hong Kong |
| eastus | Virginia, USA |
| westeurope | Netherlands |

---

## List

**What:** Lists all resource groups in the current subscription.
**When:** To verify what exists or find a resource group name.

```bash
az group list --query "[].{name:name, location:location}" -o table
```

---

## Show

**What:** Shows details of a specific resource group.
**When:** To confirm a resource group exists and check its properties.

```bash
az group show --name <name>
```

---

## Delete

**What:** Deletes a resource group and ALL resources inside it.
**When:** Cleaning up environments after use. Irreversible.

```bash
az group delete --name <name> --yes
```

| Flag | Required | Description |
|------|----------|-------------|
| --name | Yes | Name of the resource group to delete |
| --yes | No | Skip confirmation prompt |

**Warning:** This deletes everything inside the resource group. There is no undo.

---

## Add Delete Lock

**What:** Prevents a resource group from being accidentally deleted.
**When:** On critical resource groups like Terraform state storage.

```bash
az group lock create \
  --name "no-delete" \
  --lock-type CanNotDelete \
  --resource-group <name>
```

**Note:** A delete lock on the Terraform state resource group prevents the incident where a teammate accidentally deletes everything.
