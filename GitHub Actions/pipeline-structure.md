# GitHub Actions — Pipeline Structure

**Docs:** [GitHub Actions](https://docs.github.com/en/actions) | [Azure Login Action](https://github.com/Azure/login)

---

## What is GitHub Actions?

An automation platform built into GitHub. Every push or merge triggers a workflow — a set of steps that run automatically in the cloud.

```
Developer merges PR to dev
→ GitHub Actions workflow triggers
→ Lints code
→ Builds Docker image
→ Tags with git SHA
→ Pushes to ACR
→ Deploys to Container Apps
→ No manual steps
```

---

## Workflow File Structure

Workflows live in `.github/workflows/`. Each `.yml` file is one workflow.

```yaml
name: Workflow Name

on:                          # What triggers this workflow
  push:
    branches: [dev]          # Runs on push to dev branch

jobs:
  job-name:                  # One or more jobs
    runs-on: ubuntu-latest   # Runner OS

    steps:
      - name: Step name
        uses: action/name@v4 # Use a pre-built action
        with:
          key: value

      - name: Run command
        run: echo "hello"    # Run a shell command
```

---

## Common Triggers

```yaml
on:
  push:
    branches: [main, dev]           # Push to specific branches

  pull_request:
    branches: [main, dev]           # PR targeting these branches

  workflow_dispatch:                 # Manual trigger from GitHub UI
```

---

## Lint Workflow Pattern

Runs on every push to any feature branch. Blocks merges if code quality fails.

```yaml
name: Lint

on:
  push:
    branches-ignore: [main, dev]    # Runs on feat/* branches

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Lint Python (Ruff)
        run: pip install ruff && ruff check src/backend

      - name: Lint TypeScript (ESLint)
        run: |
          cd src/frontend
          npm install
          npx eslint .
```

---

## Deploy Workflow Pattern

Runs on merge to dev. Builds, tags, pushes image to ACR, deploys to Container Apps.

```yaml
name: Deploy

on:
  push:
    branches: [dev]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Login to Azure
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Login to ACR
        run: az acr login --name ${{ secrets.ACR_NAME }}

      - name: Build and push image
        run: |
          SHA=sha-${{ github.sha }}
          docker build -t ${{ secrets.ACR_LOGIN_SERVER }}/backend:$SHA ./src/backend
          docker push ${{ secrets.ACR_LOGIN_SERVER }}/backend:$SHA

      - name: Deploy to Container Apps
        run: |
          SHA=sha-${{ github.sha }}
          az containerapp update \
            --name zen-backend-dev \
            --resource-group zen-rg-dev \
            --image ${{ secrets.ACR_LOGIN_SERVER }}/backend:$SHA
```

---

## SHA Tagging

**Why SHA tags and never :latest:**

```
:latest gets overwritten on every push
→ no image history
→ rollback is impossible

sha-08643f... is permanent and unique
→ every image is traceable to an exact git commit
→ rollback = deploy the previous SHA
→ takes under 2 minutes
```

In a workflow, the git SHA is available as:
```yaml
${{ github.sha }}
```

Tag format used: `sha-${{ github.sha }}`

---

## Rollback

To roll back a Container App to a previous image version:

```bash
az containerapp update \
  --name <container-app-name> \
  --resource-group <rg> \
  --image <acr-login-server>/<repo>:<previous-sha-tag>
```

Find previous SHA tags:
```bash
az acr repository show-tags \
  --name <acr-name> \
  --repository <repo> \
  --orderby time_desc \
  --top 5
```
