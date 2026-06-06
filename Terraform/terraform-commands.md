# Terraform Commands

**Docs:** [Terraform CLI](https://developer.hashicorp.com/terraform/cli/commands) | [Azure Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)

---

## What is Terraform?

Infrastructure as Code (IaC) — define Azure resources in `.tf` files instead of clicking in the portal. Every resource is reproducible, version-controlled, and auditable.

```
You write .tf files describing what you want
→ terraform plan shows what it will create/change/destroy
→ terraform apply makes it happen
→ Azure resources now match your code
```

**Key principle:** Terraform owns infrastructure. Pipelines own application deployments (image versions). Keep these separate.

---

## Core Workflow

```bash
terraform init      # Download providers, set up backend
terraform plan      # Preview changes — always run before apply
terraform apply     # Apply changes to Azure
terraform destroy   # Destroy all managed resources
```

---

## init

**What:** Initialises Terraform — downloads providers and connects to remote state backend.
**When:** First time in a new directory, after adding new modules, or after changing backend config.

```bash
terraform init
```

After adding a new module:
```bash
terraform init -upgrade
```

---

## plan

**What:** Shows exactly what Terraform will create, change, or destroy — without doing anything.
**When:** Always run before apply. Review the output carefully.

```bash
terraform plan -var-file=environments/dev.tfvars
```

**Reading the output:**
```
+ create    → new resource will be created
~ update    → existing resource will be modified in-place
- destroy   → resource will be deleted
-/+ replace → resource will be destroyed and recreated
```

**Note:** `-/+ replace` on a Container App means downtime. Check this carefully.

---

## apply

**What:** Applies the planned changes to Azure.
**When:** After reviewing the plan output and confirming it looks correct.

```bash
terraform apply -var-file=environments/dev.tfvars
```

Auto-approve (skip confirmation prompt — use carefully):
```bash
terraform apply -var-file=environments/dev.tfvars -auto-approve
```

---

## destroy

**What:** Destroys all resources managed by this Terraform configuration.
**When:** Cleaning up an environment completely. Irreversible.

```bash
terraform destroy -var-file=environments/dev.tfvars
```

---

## import

**What:** Brings an existing Azure resource under Terraform management.
**When:** A resource was created manually or outside Terraform and you want Terraform to manage it.

```bash
terraform import <resource_address> <azure_resource_id>
```

**Example — import existing Log Analytics workspace:**
```bash
terraform import \
  module.monitoring.azurerm_log_analytics_workspace.main \
  /subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.OperationalInsights/workspaces/<name>
```

---

## state list

**What:** Lists all resources currently tracked in Terraform state.
**When:** To see what Terraform is managing.

```bash
terraform state list
```

---

## output

**What:** Shows the output values defined in outputs.tf.
**When:** To get URLs, IPs, or other values after apply.

```bash
terraform output
```

Get a specific output:
```bash
terraform output backend_url
```

---

## fmt

**What:** Formats all .tf files to the standard Terraform style.
**When:** Before committing — keeps code consistent across the team.

```bash
terraform fmt -recursive
```

---

## validate

**What:** Checks .tf files for syntax errors without connecting to Azure.
**When:** Quick check before running plan.

```bash
terraform validate
```
