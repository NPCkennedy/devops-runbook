# Image Tagging

**Docs:** [docker tag](https://docs.docker.com/engine/reference/commandline/tag/) | [az acr repository show-tags](https://learn.microsoft.com/en-us/cli/azure/acr/repository?view=azure-cli-latest#az-acr-repository-show-tags)

---

## What is Image Tagging?

A tag is a label attached to a Docker image that identifies a specific version. When you push an image to ACR, the tag is how you tell Container Apps which exact version to run.

---

## SHA Tagging vs :latest

**Never use :latest in production pipelines.**

```
:latest
  → gets overwritten on every push
  → no version history
  → rollback is impossible
  → "which version is running?" has no answer

sha-08643f958d4493fa6ef630b3c87d2384ed65ccb2
  → permanent — never overwritten
  → every image is traceable to an exact git commit
  → rollback = deploy the previous SHA
  → "which version is running?" always has an answer
```

---

## How SHA Tagging Works in the Pipeline

The git SHA is the unique fingerprint of every commit. The pipeline uses it to tag the Docker image built from that commit.

```yaml
# In GitHub Actions workflow
- name: Build and push image
  run: |
    SHA=sha-${{ github.sha }}
    docker build -t $ACR_LOGIN_SERVER/backend:$SHA ./src/backend
    docker push $ACR_LOGIN_SERVER/backend:$SHA
```

Result:
```
zenecr2026.azurecr.io/backend:sha-08643f958d4493fa6ef630b3c87d2384ed65ccb2
```

This image is permanently linked to commit `08643f9` in GitHub.

---

## View Tags in ACR

**What:** Lists all tagged images for a repository, newest first.
**When:** Before a rollback — find the last known good SHA.

```bash
az acr repository show-tags \
  --name <acr-name> \
  --repository <repo-name> \
  --orderby time_desc \
  --top 5
```

**Example:**
```bash
az acr repository show-tags \
  --name zenecr2026 \
  --repository backend \
  --orderby time_desc \
  --top 5
```

**Example output:**
```
[
  "sha-08643f958d4493fa6ef630b3c87d2384ed65ccb2",  ← latest (currently deployed)
  "sha-572ed1c8bc245ef598d018a93aa0db768dff5ef9",  ← previous
  "sha-050c156eb569730940de4b9466b7c563727767f7"   ← older
]
```

---

## Rollback Using a Previous SHA Tag

**What:** Redeploys a Container App to a previous image version.
**When:** A bad deployment went live and needs to be reverted immediately.

```bash
az containerapp update \
  --name <container-app-name> \
  --resource-group <rg> \
  --image <acr-login-server>/<repo>:<previous-sha-tag>
```

**Example — rollback frontend to previous version:**
```bash
az containerapp update \
  --name zen-frontend-dev \
  --resource-group zen-rg-dev \
  --image zenecr2026.azurecr.io/frontend:sha-572ed1c8bc245ef598d018a93aa0db768dff5ef9
```

**Note:** Note down the current good SHA before any deployment. If something breaks, you need it immediately.

---

## Rollback Both Environments

Each environment is a separate Container App — roll back each one individually.

```bash
# Rollback dev
az containerapp update \
  --name zen-frontend-dev \
  --resource-group zen-rg-dev \
  --image <acr-login-server>/frontend:<previous-sha>

# Rollback prod
az containerapp update \
  --name zen-frontend-prod \
  --resource-group zen-rg-prod \
  --image <acr-login-server>/frontend:<previous-sha>
```

**Advantage of separate environments:** You can roll back prod immediately while keeping dev on the new version to investigate the issue.

---

## Tagging Convention

| Tag format | Example | Use |
|-----------|---------|-----|
| `sha-{git-sha}` | sha-08643f... | All pipeline deployments — permanent |
| `latest` | latest | Never use in pipelines |
| `v1.0` | v1.0 | Release tags only — applied manually at milestone |
