# Lab– System-Assigned Managed Identity: VM Access to Azure Blob Storage
Ref:  [ App Registration & Service Principal Permissions](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/tutorial-windows-managed-identities-vm-access?pivots=identity-windows-mi-vm-access-data-lake)
## Objective
Configure a VM's system-assigned managed identity to authenticate to Azure Blob Storage and read container contents — without storing any credentials, keys, or connection strings in code.

## Scenario
An application running on a VM needs read/write access to a Blob Storage container. Instead of embedding a storage account key or SAS token, use Microsoft Entra ID–backed managed identity + RBAC.

## Environment
- VM: `myvm` (Ubuntu 24.04 LTS)
- Storage Account: `storage79519`
- Container: `mycont-demo`
- Subscription: `subscription-test`
- Role: `Storage Blob Data Contributor`

## Steps

### 1. Enable System-Assigned Managed Identity on the VM
- VM → **Identity** → **System assigned** tab → Status → **On** → **Save**
- Confirms a service principal is created in Entra ID, tied to the VM's lifecycle

### 2. Create Storage Account + Container
- Created storage account `storage79519`
- Container `mycont-demo` created, test blob `image (3) (1).png` uploaded

### 3. Grant RBAC Role to the VM's Managed Identity
- Storage Account → **Access Control (IAM)** → **Add role assignment**
- Role: `Storage Blob Data Contributor`
- Assign access to: **Managed identity** → **Virtual machine** → `myvm`
- Verified in IAM blade: `myvm` listed under `Storage Blob Data Contributor`, scope = *This resource*

### 4. Request Access Token from IMDS (inside the VM)
```powershell
$response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fstorage.azure.com%2F' -Method GET -Headers @{Metadata="true"}
$content = $response.Content | ConvertFrom-Json
$AccessToken = $content.access_token
```

### 5. Call Blob Storage REST API with the Token
```powershell
Invoke-WebRequest -Uri "https://storage79519.blob.core.windows.net/mycont-demo?restype=container&comp=list" `
  -Headers @{Authorization="Bearer $AccessToken"; "x-ms-version"="2021-08-06"}
```

**Result:** `200 OK` — XML response listing `image (3) (1).png` inside `mycont-demo`.

## Troubleshooting Encountered
- Initial call returned `ContainerNotFound` — used container name `midemo` instead of the actual `mycont-demo`.
- Fix: matched the URI path to the exact container name created in the portal.
- Takeaway: IMDS/token errors ≠ RBAC errors ≠ resource-not-found errors. Read the error body carefully before assuming a permissions issue.

## Exam Takeaway (AZ-500 – Domain 1: Identity)
- Managed identity eliminates stored credentials; the token is scoped and short-lived, requested only from inside the resource via the link-local IMDS endpoint (`169.254.169.254`) — not reachable externally.
- RBAC for blob **data plane** can be scoped at multiple levels:
  - Subscription
  - Resource group
  - Storage account (used in this lab — "This resource")
  - **Container level** (least privilege) — via `az role assignment create --scope .../containers/<name>`
- Exam scenario pattern: *"App needs to read blobs without embedding secrets"* → System-assigned MI + RBAC role assignment, not access keys or SAS tokens.
Screenshots:
<img width="1365" height="477" alt="image" src="https://github.com/user-attachments/assets/7119ac28-8b14-459b-873f-b2971e41273b" />
<img width="1358" height="546" alt="image" src="https://github.com/user-attachments/assets/c100c407-ae96-4195-b810-03211cb6d114" />
<img width="1364" height="576" alt="image" src="https://github.com/user-attachments/assets/8afcf6d5-b63e-4920-9f3e-4f1518c247a5" />
<img width="1365" height="608" alt="image" src="https://github.com/user-attachments/assets/9b6a2aa2-5894-4c73-b5a0-aada7f1d4319" />
<img width="1365" height="611" alt="image" src="https://github.com/user-attachments/assets/d47181b2-8e46-4540-b214-ff4feeb3a46d" />
<img width="1141" height="412" alt="image" src="https://github.com/user-attachments/assets/31741d69-02ec-420c-9dfb-4973f8c6cea5" />

## Status
✅ Complete and validated 
