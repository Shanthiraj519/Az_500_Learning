# Lab 21: Private Endpoint for Azure Web App (App Service)

## Objective
Deploy an Azure Web App (App Service) and secure it using a Private Endpoint, validating that DNS resolution and connectivity work correctly from within a peered VNet via Azure Bastion.

**Reference:** [Create a private endpoint - Microsoft Learn](https://learn.microsoft.com/en-us/azure/private-link/create-private-endpoint-portal)

---

## Architecture

| Resource | Name | Region | Notes |
|---|---|---|---|
| Resource Group | `test-rg` | East US | Container for all lab resources |
| Web App | `webapp9519` | Canada Central | App Service Plan: Basic B1 |
| App Service Plan | `ASP-testrg-a88f` | Canada Central | B1 tier (min tier supporting Private Link) |
| Virtual Network | `vnet-1` | East US | Hosts private endpoint + test VM |
| Subnet | `Default` | East US | — |
| Private Endpoint | `private-endpoint` | East US | Target: `webapp9519`, sub-resource: `sites` |
| Private DNS Zone | `privatelink.azurewebsites.net` | Global | Auto-linked to `vnet-1` |
| NSG | `nsg-1` | East US | Attached to subnet |
| Test VM | `vm-1` | East US | Windows Server 2025, accessed via Bastion |
| Bastion | — | East US | Used for private connectivity testing |

> **Note:** Private Link supports cross-region connections — the Web App (Canada Central) and the Private Endpoint/VNet (East US) do not need to share a region. Only the Private Endpoint and its target VNet/subnet must be co-located.

---

## Deployed Resources (Resource Group: `test-rg`)

| Resource Name | Type | Region |
|---|---|---|
| `Application Insights Smart Detection` | Action group | Global |
| `ASP-testrg-a88f` | App Service plan | Canada Central |
| `nsg-1` | Network security group | East US |
| `private-endpoint` | Private endpoint | East US |
| `private-endpoint-nic` | Network interface | East US |
| `privatelink.azurewebsites.net` | Private DNS zone | Global |
| `vm-1` | Virtual machine | East US |
| `vm-1-ip` | Public IP address | East US |
| `vm-1931` | Network interface | East US |
| `vm-1_OsDisk_1_d5210fe1f03a40d8bac92a38910cb3f4` | Disk | East US |
| `webapp9519` | Web App | Canada Central |
| *(+ 4 additional resources — vnet-1, Bastion host, Bastion public IP, etc. — resource group showed 15 total)* |

> Update this table with the remaining resources from page 2 of the resource group view (VNet, Bastion host, Bastion public IP, etc.) for a complete inventory.

---

## Resource Details

**Web App (`webapp9519`):**

| Property | Value |
|---|---|
| Resource group | `test-rg` |
| Region | Canada Central |
| App Service Plan | `ASP-testrg-a88f` (B1: 1) |
| Operating System | Windows |
| Status | Running |
| Default domain | `webapp9519-gsgud5e2c7hxgmcb.canadacentral-01.azurewebsites.net` |

**Private Endpoint (`private-endpoint`):**

| Property | Value |
|---|---|
| Resource group | `test-rg` |
| Location | East US |
| Virtual network/subnet | `vnet-1/Default` |
| Network interface | `private-endpoint-nic` |
| Private link resource | `webapp9519` |
| Target sub-resource | `sites` |
| Connection status | Approved |
| Provisioning state | Succeeded |

**Private DNS Zone (`privatelink.azurewebsites.net`):**

| Property | Value |
|---|---|
| Resource group | `test-rg` |
| Location | Global |
| Recordsets | 3 |
| Virtual Network Links | 1 |

**Test VM (`vm-1`):**

| Property | Value |
|---|---|
| OS | Windows Server 2025 Datacenter |
| Private IP | 10.0.0.5 |
| Public IP | 104.211.49.39 |
| VNet/Subnet | `vnet-1/Default` |
| Size | Standard D2s v3 (2 vCPU, 8 GiB RAM) |
| Agent status | Ready |

---

## Validation — DNS Resolution to Private IP

```powershell
PS C:\Windows\system32> nslookup webapp9519-gsgud5e2c7hxgmcb.canadacentral-01.azurewebsites.net
Server:   UnKnown
Address:  168.63.129.16

Non-authoritative answer:
Name:     webapp9519-gsgud5e2c7hxgmcb.canadacentral-01.privatelink.azurewebsites.net
Address:  10.0.0.4
Aliases:  webapp9519-gsgud5e2c7hxgmcb.canadacentral-01.azurewebsites.net
```

**Result:** ✅ The public FQDN resolves via CNAME to the `privatelink.azurewebsites.net` zone, returning a **private IP (`10.0.0.4`)** inside the VNet address space — confirming Private Link DNS integration and connectivity are working end-to-end.
Screenshots:
<img width="1223" height="483" alt="image" src="https://github.com/user-attachments/assets/89f90bec-3d29-4ef7-b4c7-93138e35bab3" />
<img width="1357" height="512" alt="image" src="https://github.com/user-attachments/assets/00105a23-2a22-4489-ad9a-b18d74f17fb1" />
<img width="1359" height="516" alt="image" src="https://github.com/user-attachments/assets/5e70e3c4-cc97-4938-bc27-16b9556199c5" />
<img width="1365" height="587" alt="image" src="https://github.com/user-attachments/assets/29a3ab9f-d0ef-4eb6-9360-1a514dbf82d6" />
<img width="1365" height="597" alt="image" src="https://github.com/user-attachments/assets/cb531f82-e597-4a46-9afd-b47a961e263e" />
<img width="1364" height="584" alt="image" src="https://github.com/user-attachments/assets/108c2b16-b9be-429e-ae5c-bb1db4e8c2ee" />
<img width="1048" height="505" alt="image" src="https://github.com/user-attachments/assets/85735fcc-59b0-4de6-9aec-f07138b0265e" />
<img width="1329" height="683" alt="image" src="https://github.com/user-attachments/assets/2905db36-6f94-4798-8e58-c7f16c78ec1a" />

---

## Key Takeaways
- App Service Basic tier (B1) or above is required to support Private Endpoints — Free/Shared tiers are not eligible.
- Private Link works cross-region — Web App region and Private Endpoint/VNet region don't need to match, only the endpoint and its target subnet do.
- The Private DNS Zone (`privatelink.azurewebsites.net`) auto-created and auto-linked to the VNet when default settings were left unchanged during private endpoint creation, requiring no manual DNS configuration.
- Successful resolution to a `10.x.x.x` private IP (instead of a public IP or NXDOMAIN) is the definitive proof that Private Link + DNS integration is functioning correctly.

---

## Cleanup
```powershell
# Remove the resource group to delete all lab resources and stop billing
Remove-AzResourceGroup -Name "test-rg" -Force
```
