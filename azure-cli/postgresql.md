# PostgreSQL Flexible Server

**Docs:** [az postgres flexible-server](https://learn.microsoft.com/en-us/cli/azure/postgres/flexible-server?view=azure-cli-latest)

---

## What is PostgreSQL Flexible Server?

A fully managed PostgreSQL database on Azure. You don't manage the VM, OS, or patches — Azure handles it. You just connect and use the database.

**Billing:** Charged by the hour regardless of traffic. The meter runs 24/7 as long as the server exists.

**Tiers:**
| Tier | Use case | Cost |
|------|----------|------|
| Burstable (B1ms) | Dev/test | ~$15-25/month |
| General Purpose | Production | ~$150-400/month |
| Memory Optimized | High performance | ~$300-800/month |

---

## Install Extension

```bash
az extension add --name rdbms-connect --upgrade
```

---

## Create

**What:** Provisions a new PostgreSQL Flexible Server.
**When:** Once per environment.

```bash
az postgres flexible-server create \
  --name <server-name> \
  --resource-group <rg> \
  --location <region> \
  --admin-user <username> \
  --admin-password <password> \
  --sku-name Standard_B1ms \
  --tier Burstable \
  --version 15 \
  --storage-size 32
```

| Flag | Required | Description |
|------|----------|-------------|
| --name | Yes | Server name — must be globally unique |
| --resource-group | Yes | Resource group |
| --location | Yes | Azure region |
| --admin-user | Yes | Admin username |
| --admin-password | Yes | Admin password (min 8 chars, mixed case, number, special) |
| --sku-name | Yes | VM size e.g. Standard_B1ms |
| --tier | Yes | Burstable, GeneralPurpose, MemoryOptimized |
| --version | No | PostgreSQL version (15 recommended) |
| --storage-size | No | Storage in GB (minimum 32) |

---

## Create Database

**What:** Creates a database inside the server.
**When:** After creating the server — one server can host multiple databases.

```bash
az postgres flexible-server db create \
  --server-name <server-name> \
  --resource-group <rg> \
  --database-name <db-name>
```

---

## Add Firewall Rule

**What:** Allows connections from a specific IP or from all Azure services.
**When:** To allow your application or local machine to connect.

Allow all Azure services:
```bash
az postgres flexible-server firewall-rule create \
  --name <server-name> \
  --resource-group <rg> \
  --rule-name allow-azure-services \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0
```

Allow your local IP:
```bash
MY_IP=$(curl -s ifconfig.me)
az postgres flexible-server firewall-rule create \
  --name <server-name> \
  --resource-group <rg> \
  --rule-name allow-my-ip \
  --start-ip-address $MY_IP \
  --end-ip-address $MY_IP
```

---

## Update Admin Password

**What:** Rotates the admin password.
**When:** After accidental credential exposure.

```bash
az postgres flexible-server update \
  --name <server-name> \
  --resource-group <rg> \
  --admin-password <new-password>
```

**Note:** After rotating, update the password in: Key Vault secret, Terraform tfvars, and any local .env files.

---

## Show Connection String

**What:** Constructs the database connection string.
**When:** When configuring application environment variables.

```
postgresql://<admin-user>:<password>@<server-name>.postgres.database.azure.com:5432/<db-name>?sslmode=require
```

**Note:** Always use `sslmode=require` for Azure PostgreSQL connections.

---

## Delete

**What:** Deletes the server and all databases inside it.
**When:** Cleaning up environments. Irreversible.

```bash
az postgres flexible-server delete \
  --name <server-name> \
  --resource-group <rg> \
  --yes
```
