# Azure Application Gateway — Deployment Documentation

## Overview
This project deploys an **Azure Application Gateway (Standard V2)** with a public frontend that load balances HTTP traffic across two backend IIS web servers.

Reference: [Azure Application Gateway quickstart — Azure Portal](https://learn.microsoft.com/en-us/azure/application-gateway/quick-create-portal)

---

## Resources Created

| # | Resource Type | Name | Details |
|---|---|---|---|
| 1 | Subscription | `subscription-test` | Subscription ID: `aab044c3-9fe6-4c41-956a-e2a6f41b7316` |
| 2 | Resource Group | `MyResourceGroup` | Location: East US |
| 3 | Virtual Network | `MyVnet` | Address space covering both subnets below |
| 4 | Subnet (Gateway) | `AGSubnet` | `10.21.0.0/24` — 251 available IPs |
| 5 | Subnet (Backend) | `MyBackendSubnet` | `10.21.1.0/24` — 251 available IPs |
| 6 | Application Gateway | `myAppGateway` | Tier: Standard V2, Zones: 1, 2, 3 |
| 7 | Public IP Address | `myAGPublicIPAddress` | `52.186.165.66` |
| 8 | Frontend IP Configuration | `appGwPublicFrontendIpIPv4` | Type: Public, linked to `myListener` |
| 9 | Listener | `myListener` | Bound to frontend public IP |
| 10 | Backend Pool | `myBackendPool` | 2 VM targets: `myvm-02283`, `myvm214` |
| 11 | Backend Settings | (HTTP, port 80) | Used by routing rule |
| 12 | Virtual Machine | `myVM` | Windows Server, IIS installed, no public IP |
| 13 | Virtual Machine | `myVM-02` | Windows Server, IIS installed, no public IP |
| 14 | VM Extension | `IIS` (CustomScriptExtension) | Deployed to both VMs — Status: OK |

---

## Architecture Summary

| Component | Name | Details |
|---|---|---|
| Resource Group | `MyResourceGroup` | Contains all resources above |
| Virtual Network | `MyVnet` | Split into gateway + backend subnets |
| Gateway Subnet | `AGSubnet` | `10.21.0.0/24` |
| Backend Subnet | `MyBackendSubnet` | `10.21.1.0/24` |
| Application Gateway | `myAppGateway` | Tier: Standard V2 |
| Public IP | `myAGPublicIPAddress` | `52.186.165.66` |
| Backend Pool | `myBackendPool` | 2 backend targets (VMs) |
| Backend VMs | `myVM`, `myVM-02` | Windows Server, IIS installed |

---

## Setup Steps

### 1. Virtual Network & Subnets
Created `MyVnet` with two subnets to separate gateway and backend traffic:
- `AGSubnet` → `10.21.0.0/24`
- `MyBackendSubnet` → `10.21.1.0/24`

Verified in **Virtual Network → Subnets**:

| Name | IPv4 | Available IPs |
|---|---|---|
| MyBackendSubnet | 10.21.1.0/24 | 251 |
| AGSubnet | 10.21.0.0/24 | 251 |

### 2. Application Gateway
Deployed `myAppGateway` (Standard V2 tier) into `MyVnet/AGSubnet`, with:
- Public frontend IP: `myAGPublicIPAddress` (52.186.165.66)
- Frontend IP config: `appGwPublicFrontendIpIPv4`
- Listener: `myListener`
- Backend pool: `myBackendPool` (created without targets initially)
- Routing rule linking listener → backend pool → backend HTTP settings (port 80)

### 3. Backend VMs
Provisioned two Windows Server VMs with no public IP (internal only, behind the gateway):
- `myVM`
- `myVM-02`

### 4. IIS Installation (via Azure Cloud Shell)
Installed IIS and wrote the VM's computer name to the default page using the Custom Script Extension:

```powershell
Set-AzVMExtension `
  -ResourceGroupName myResourceGroup `
  -ExtensionName IIS `
  -VMName myVM `
  -Publisher Microsoft.Compute `
  -ExtensionType CustomScriptExtension `
  -TypeHandlerVersion 1.4 `
  -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `
  -Location EastUS
```

Repeated with `-VMName myVM-02`. Both extensions returned `StatusCode: OK`.

### 5. Backend Pool Registration
Added both VMs as **Virtual machine** targets in `myBackendPool`:

| Target Type | Target |
|---|---|
| Virtual machine | myvm-02283 |
| Virtual machine | myvm214 |

---

## Verification / Test Results

Browsed to the gateway's public IP (`52.186.165.66`) and refreshed multiple times. The response alternated between the two backend VMs, confirming traffic is being load balanced correctly:

- Request 1 → `MyVm`
- Request 2 → `Myvm-02`

## Key Takeaways

- **Standard V2 Application Gateway** requires its own dedicated subnet (`AGSubnet`) — it can't share a subnet with VMs or other resources.
- **Backend VMs don't need public IPs.** The gateway routes traffic to them over the private VNet, which keeps the attack surface smaller.
- **Custom Script Extension** is a fast way to bootstrap IIS + a test page on a VM without RDPing in manually.
- **Load balancing was verified visually** — refreshing the gateway's public IP alternated between `MyVm` and `Myvm-02`, proving round-robin distribution works out of the box with no extra config.
- **No inbound RDP by default** — since the VMs have no public IP, `Azure Bastion` (visible in the subnet list) would be the way to manage them directly if needed.
- **Everything is tied together by the frontend IP config** (`appGwPublicFrontendIpIPv4`) → listener (`myListener`) → backend pool (`myBackendPool`) → backend settings chain. Understanding this chain is the key to troubleshooting gateway routing issues later.

---## Cleanup
To remove all resources created in this deployment:

```bash
az group delete --name MyResourceGroup --yes --no-wait
```

---



## Notes
- VMs have **no public IP** — inbound access is blocked by default. Use **Azure Bastion** for RDP access if needed.
- Gateway tier used: **Standard V2**.
---✅ **Result: Load balancing confirmed working.**
Screenshots:
<img width="678" height="205" alt="image" src="https://github.com/user-attachments/assets/e9b70d5f-c3ad-48ae-9b65-d705c7acf2f7" />
<img width="612" height="226" alt="image" src="https://github.com/user-attachments/assets/8f9fed6b-80d0-47b6-b62e-22ecf0eaffd1" />
<img width="1568" height="670" alt="image" src="https://github.com/user-attachments/assets/6b91cff1-880c-4f9f-9012-565efd1812b8" />
<img width="1568" height="397" alt="image" src="https://github.com/user-attachments/assets/8e7998b9-7d3b-4b24-83b8-5ffde1c43682" />
<img width="1525" height="784" alt="image" src="https://github.com/user-attachments/assets/43af4b89-f7ab-42d8-85e6-0b791724524f" />
<img width="1568" height="601" alt="image" src="https://github.com/user-attachments/assets/d3856b7c-bb71-485a-ab82-a1fbc24ebb68" />
<img width="1568" height="705" alt="image" src="https://github.com/user-attachments/assets/52059fd0-f376-485a-80c0-a265f9935715" />

## Notes
- VMs have **no public IP** — inbound access is blocked by default. Use **Azure Bastion** for RDP access if needed.
- Gateway tier used: **Standard V2**.
