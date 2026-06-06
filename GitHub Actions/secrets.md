# GitHub Actions — Secrets

**Docs:** [GitHub Encrypted Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

---

## What are GitHub Secrets?

Encrypted variables stored in GitHub and injected into workflows at runtime. Never appear in logs or code.

```
AZURE_CREDENTIALS stored in GitHub Secrets
→ workflow references ${{ secrets.AZURE_CREDENTIALS }}
→ GitHub decrypts and injects at runtime
→ never visible in logs
```

---

## Set a Secret (via GitHub UI)

```
Repository → Settings → Secrets and variables → Actions → New repository secret
```

---

## Set a Secret (via GitHub CLI)

```bash
gh secret set SECRET_NAME --body "secret-value"
```

---

## Reference a Secret in a Workflow

```yaml
steps:
  - name: Login to Azure
    uses: azure/login@v2
    with:
      creds: ${{ secrets.AZURE_CREDENTIALS }}
```

---

## Required Secrets for Azure CI/CD

| Secret | Value | Purpose |
|--------|-------|---------|
| AZURE_CREDENTIALS | JSON output from `az ad sp create-for-rbac --json-auth` | Authenticates pipeline to Azure |
| ACR_LOGIN_SERVER | e.g. zenecr2026.azurecr.io | ACR hostname for image tagging |
| ACR_NAME | e.g. zenecr2026 | ACR name for az acr login |
| AZURE_SUBSCRIPTION_ID | Subscription ID | Target subscription for deployments |

---

## Generate AZURE_CREDENTIALS

```bash
az ad sp create-for-rbac \
  --name "github-actions-sp" \
  --role Contributor \
  --scopes /subscriptions/<subscription-id> \
  --json-auth
```

Copy the entire JSON output and paste it as the value of the `AZURE_CREDENTIALS` secret.

**The JSON looks like:**
```json
{
  "clientId": "...",
  "clientSecret": "...",
  "subscriptionId": "...",
  "tenantId": "..."
}
```

**Save this immediately** — the clientSecret is only shown once.

---

## Update a Secret

Go to: Repository → Settings → Secrets and variables → Actions → Click the secret → Update

Or via CLI:
```bash
gh secret set SECRET_NAME --body "new-value"
```

---

## Security Rules

- Never log secrets with `echo ${{ secrets.X }}` — GitHub will mask them but it is bad practice
- Never hardcode secrets in workflow files
- Never commit `.env` files or `*.tfvars` files containing secrets
- Rotate secrets immediately after accidental exposure
- Use per-resource permissions — give the pipeline only the access it needs (e.g. AcrPush, not Contributor on the whole subscription)
