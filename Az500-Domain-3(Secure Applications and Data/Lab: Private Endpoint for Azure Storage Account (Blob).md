# Lab: Private Endpoint for Azure Storage Account (Blob)

## 📌 Objective
Deploy a Private Endpoint for an Azure Storage Account (Blob service) to restrict access to the storage account over the private VNet, validate DNS resolution via Azure Private DNS Zone, and troubleshoot public/portal access behavior once public network access is denied.

Reference: [Microsoft Learn - Tutorial: Connect to a storage account using an Azure Private Endpoint](https://learn.microsoft.com/en-us/azure/private-link/tutorial-private-endpoint-storage-portal)

---

## 🏗️ Environment Details

| Resource | Value |
|---|---|
| Resource Group | RG-Domain3-Lab |
| Storage Account | storage79519 |
| Storage Kind | StorageV2 (general purpose v2) |
| Replication | LRS (Locally Redundant Storage) |
| Location | East US |
| VM Name | vm-1 |
| VM OS | Windows Server 2025 Datacenter |
| VM Size | Standard D2s v3 (2 vCPU, 8 GiB RAM) |
| VNet/Subnet | Vnet1 / Subnet-1 |
| Private Endpoint Target | Blob sub-resource |
| Private DNS Zone | privatelink.blob.core.windows.net |

---

## 🔧 Steps Performed

1. Created Storage Account `storage79519` (StorageV2, LRS, Hot tier) in `RG-Domain3-Lab`.
2. Created a Blob container `demo-cont` inside the storage account.
3. Deployed a Private Endpoint for the storage account, targeting the **blob** sub-resource, integrated into `Vnet1/Subnet-1`.
4. Enabled **Private DNS Zone integration** — automatically created/linked `privatelink.blob.core.windows.net` zone to the VNet.
5. Disabled/restricted **public network access** on the storage account (firewall set to deny public access).
6. Validated name resolution from `vm-1` (deployed inside the same VNet) using `nslookup`.
7. Attempted access to the storage container via the Azure Portal (from a public/non-VNet client context) to observe expected failure behavior.

---

## ✅ Validation — DNS Resolution from VM (Inside VNet)

Ran from `vm-1` (PowerShell):

```powershell
PS C:\Windows\system32> nslookup storage79519.blob.core.windows.net
Server:   UnKnown
Address:  168.63.129.16

Non-authoritative answer:
Name:    storage79519.privatelink.blob.core.windows.net
Address:  10.1.0.4
Aliases:  storage79519.blob.core.windows.net
```

**Result:** ✅ Confirms the public FQDN `storage79519.blob.core.windows.net` correctly resolves (via CNAME alias) to the private-link FQDN `storage79519.privatelink.blob.core.windows.net`, which in turn resolves to the **private IP `10.1.0.4`** inside the subnet — proving the Private DNS Zone integration and Private Endpoint are working as expected. DNS query answered by Azure-provided DNS (`168.63.129.16`).

---

## ⚠️ Issue Encountered — Portal/Public Access Blocked

### 1. AuthorizationFailure (XML Error via REST API)

```xml
<Error>
  <Code>AuthorizationFailure</Code>
  <Message>
    This request is not authorized to perform this operation.
    RequestId:df957586-301e-002c-5954-10e123000000
    Time:2026-07-10T10:08:36.6420272Z
  </Message>
</Error>
```

### 2. Azure Portal — Container Browse Failure

Navigating to **Storage Account → Containers → demo-cont** from the Azure Portal produced:

> **Network request failed - cannot access storage endpoint**
>
> - Your firewall settings may be preventing network access from this client to the storage data endpoint.
> - CORS settings on your storage account may be preventing network access from this client.

**Root Cause:** Once public network access is disabled on the storage account and access is scoped exclusively to the Private Endpoint, any client **not routed through the private VNet path** (e.g., the Azure Portal data-plane browser session, or a REST call originating from outside the VNet/without proper private connectivity) is denied — this is expected behavior, not a misconfiguration. The Portal's Storage Browser blade makes direct calls to the storage data endpoint from the browser session, which does not traverse the private endpoint unless accessed from a machine/network with that private path (e.g., via Azure Bastion session on `vm-1`, or a jumpbox inside the VNet).

---

## 🎯 Key Learnings

- Private Endpoint + Private DNS Zone integration correctly overrides public DNS resolution — the public FQDN transparently resolves to a private IP for clients inside the linked VNet.
- Disabling public network access on the storage account enforces that **only traffic via the Private Endpoint (or explicitly allowed networks) succeeds** — Portal-based blob browsing from outside that path will fail with `AuthorizationFailure` / network request errors, even for an authenticated Azure AD user with sufficient RBAC, because the network path itself is blocked before authorization is evaluated at the data plane.
- To manage/browse blob data on a privately-restricted storage account, access must originate from within the VNet (e.g., via Bastion/RDP into `vm-1`) or through an approved network exception (private endpoint, VPN/ExpressRoute, or explicit IP allow-list).
- `168.63.129.16` is the Azure platform DNS (Wire Server) IP — used here to resolve the private-link zone record automatically once the zone is linked to the VNet.

---<img width="761" height="285" alt="image" src="https://github.com/user-attachments/assets/e22969b6-cf33-4d39-bfba-d77beecad73b" />
<img width="1365" height="577" alt="image" src="https://github.com/user-attachments/assets/8642f4dc-d326-4f74-915d-1224ef21a041" />
<img width="1364" height="612" alt="image" src="https://github.com/user-attachments/assets/8aeb0045-e3be-48e9-befb-63353d3aa4bf" />
<img width="1323" height="186" alt="image" src="https://github.com/user-attachments/assets/02b32cd2-3305-41e3-8d6a-ea105cf32951" />
<img width="1347" height="604" alt="image" src="https://github.com/user-attachments/assets/85c19f02-a4e8-4a50-80b6-fb578762293e" />

## 📸 Evidence Captured
- `nslookup` output confirming private IP resolution (10.1.0.4)
- VM Overview blade (vm-1 — network/subnet context)
- Storage Account Overview blade (storage79519 — properties/security settings)
- AuthorizationFailure XML error (REST/data-plane access denied)
- Portal Storage Browser network failure with firewall/CORS guidance
