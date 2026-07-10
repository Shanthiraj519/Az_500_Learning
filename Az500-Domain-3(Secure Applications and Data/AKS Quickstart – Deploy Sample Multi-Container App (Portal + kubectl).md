## AKS Quickstart – Deploy Sample Multi-Container App (Portal + kubectl)

**Date:** July 10, 2026
**Reference:** [Microsoft Learn – Deploy an AKS cluster using Azure portal](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-portal)
**Purpose:** Hands-on kubectl/AKS proficiency (not AZ-500 exam-mapped; general Kubernetes skill-building)

### What was deployed
- AKS cluster `MyAKSCluster` created via Azure Portal, Dev/Test preset, Free tier
- Node pools: default `agentpool` (2 nodes, system) + `nplinux` (1 node, user pool) — 3 nodes total
- Sample microservices app (`aks-store-quickstart.yaml`) deployed via kubectl:
  - RabbitMQ (StatefulSet)
  - order-service (Deployment)
  - product-service (Deployment)
  - store-front (Deployment, LoadBalancer service)

### Issues encountered and resolved

1. **`kubectl` not recognized in PowerShell**
   Binary existed at `$HOME\.azure-kubectl\kubectl.exe` but wasn't on PATH (common after `az aks install-cli`).
   Fix: added the folder to PATH for the session/permanently via `$env:Path` / `SetEnvironmentVariable`.

2. **`kubectl apply` failed with "invalid object to validate"**
   Cause: YAML file created via `notepad` was empty/incomplete on first save.
   Fix: re-pasted full manifest content into Notepad.

3. **`store-front` Deployment invalid: missing `containers`/`labels`**
   Cause: partial content loss during Notepad copy-paste (rest of manifest applied fine).
   Fix: wrote the `store-front` block directly from PowerShell using a here-string (`@'...'@ | Set-Content -Encoding utf8`) to avoid Notepad's UTF-16 encoding/paste issues, then applied it as a separate file.

### Unresolved issue

- **`order-service` pod stuck in `CrashLoopBackOff`**
  - Init container `wait-for-rabbitmq` completed successfully (RabbitMQ TCP port reachable).
  - Main container exits with `Exit Code: 1` before binding to port 3000 — startup probe reports `connection refused`.
  - Likely an application-level issue (AMQP connection/auth to RabbitMQ, or startup race condition) rather than a cluster/networking config issue.
  - Root cause not pursued further — deprioritized in favor of AZ-500 exam prep (exam July 25, 2026). Flagged here for future revisit.

### Cleanup
- Resource group deleted post-lab to avoid ongoing cost, given 3-node cluster vs. single-node budget assumption (₹800 alert threshold).

### Takeaway
Reinforced kubectl basics (`get nodes`, `apply`, `get pods`, `describe pod`, `logs`), practiced diagnosing a CrashLoopBackOff via pod events, and hit real-world YAML/PowerShell encoding gotchas — useful troubleshooting reps even though
