# Terraform Modules

**Docs:** [Terraform Modules](https://developer.hashicorp.com/terraform/language/modules) | [Azure Provider Modules](https://registry.terraform.io/namespaces/Azure)

---

## What is a Module?

A reusable block of Terraform configuration. Instead of writing the same resource definitions for dev and prod, you write it once as a module and call it with different variables.

```
modules/
  container-apps/    ← defines the Container App resources
  postgresql/        ← defines the PostgreSQL resources
  keyvault/          ← defines the Key Vault resources
  monitoring/        ← defines App Insights + Log Analytics

main.tf calls each module:
  module "container_apps" {
    source = "./modules/container-apps"
    environment = "dev"
    ...
  }
```

---

## Standard Module Structure

Every module should have three files:

```
modules/<name>/
  main.tf        ← the actual resources
  variables.tf   ← input variables the module accepts
  outputs.tf     ← values the module exposes to the caller
```

---

## variables.tf

Defines what inputs the module accepts.

```hcl
variable "environment" {
  description = "Deployment environment (dev or prod)"
  type        = string
}

variable "location" {
  description = "Azure region"
  type        = string
  default     = "southeastasia"
}

variable "resource_group_name" {
  description = "Resource group to deploy into"
  type        = string
}
```

---

## outputs.tf

Exposes values from inside the module so other modules or main.tf can use them.

```hcl
output "connection_string" {
  description = "App Insights connection string"
  value       = azurerm_application_insights.main.connection_string
  sensitive   = true
}

output "database_url" {
  description = "PostgreSQL connection URL"
  value       = "postgresql://${var.db_admin_username}:${var.db_admin_password}@${azurerm_postgresql_flexible_server.main.fqdn}:5432/${var.db_name}?sslmode=require"
  sensitive   = true
}
```

**sensitive = true:** Hides the value in terraform plan and apply output. Still accessible by other modules.

---

## Calling a Module in main.tf

```hcl
module "monitoring" {
  source              = "./modules/monitoring"
  environment         = var.environment
  location            = var.location
  resource_group_name = var.resource_group_name
}

# Use an output from another module
module "container_apps" {
  source                        = "./modules/container-apps"
  environment                   = var.environment
  location                      = var.location
  resource_group_name           = var.resource_group_name
  appinsights_connection_string = module.monitoring.connection_string
}
```

---

## lifecycle — ignore_changes

Tells Terraform to ignore changes to specific attributes after initial creation. Used when another system (e.g. a CI/CD pipeline) owns that attribute.

```hcl
resource "azurerm_container_app" "backend" {
  name = "zen-backend-${var.environment}"
  ...

  lifecycle {
    ignore_changes = [template]
  }
}
```

**Why this matters for Container Apps:**
The pipeline deploys new image versions by updating the container template directly. Without `ignore_changes = [template]`, running `terraform apply` would revert the container back to the image tag hardcoded in Terraform — overwriting whatever the pipeline deployed.

```
Without lifecycle:
  Pipeline deploys sha-NEW → Container Apps updated
  terraform apply → reverts back to sha-OLD in tfvars ✗

With lifecycle:
  Pipeline deploys sha-NEW → Container Apps updated
  terraform apply → ignores template, only touches infra ✓
```

**Rule:** Pipeline owns images. Terraform owns infrastructure.

---

## tfvars Files

Variable values for each environment. Always gitignored — never committed.

```hcl
# environments/dev.tfvars
environment         = "dev"
location            = "southeastasia"
resource_group_name = "zen-rg-dev"
```

Pass to Terraform commands with:
```bash
terraform plan -var-file=environments/dev.tfvars
terraform apply -var-file=environments/dev.tfvars
```

**Never paste tfvars content into chat or logs** — they contain passwords and credentials.
