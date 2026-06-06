# Account & Subscription

**Docs:** [az account](https://learn.microsoft.com/en-us/cli/azure/account?view=azure-cli-latest)

---

## Login

**What:** Authenticates your terminal session to Azure.
**When:** Before running any az command on a new terminal or after token expiry.

```bash
az login
```

For a specific tenant:
```bash
az login --tenant <tenant-id>
```

With a specific scope:
```bash
az logout
az login --tenant <tenant-id> --scope "https://management.azure.com/.default"
```

---

## Show Current Account

**What:** Shows which subscription and account you are currently logged into.
**When:** Before provisioning anything — always verify you are on the right subscription.

```bash
az account show --query "{name:name, id:id}" -o table
```

| Flag | Required | Description |
|------|----------|-------------|
| --query | No | JMESPath filter — only show fields you need |
| -o | No | Output format: table, json, yaml, tsv |

**Example output:**
```
Name                 Id
-------------------  ------------------------------------
Azure for Students   ce9656f2-9327-40c5-b220-6c45feacfd66
```

---

## List All Subscriptions

**What:** Lists all subscriptions available to your account.
**When:** When you have multiple subscriptions and need to find the right one.

```bash
az account list --query "[].{name:name, id:id, state:state}" -o table
```

---

## Set Active Subscription

**What:** Switches your active subscription context.
**When:** When you have multiple subscriptions and need to work on a specific one.

```bash
az account set --subscription <subscription-id>
```

**Example:**
```bash
az account set --subscription ce9656f2-9327-40c5-b220-6c45feacfd66
```

**Note:** Always run `az account show` after switching to confirm the change took effect.

---

## Logout

**What:** Ends your Azure CLI session.
**When:** On shared machines or when switching accounts.

```bash
az logout
```
