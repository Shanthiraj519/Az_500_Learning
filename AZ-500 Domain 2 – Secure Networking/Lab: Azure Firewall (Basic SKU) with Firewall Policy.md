# Lab: Azure Firewall (Basic SKU) with Firewall Policy

## Objective
Deploy Azure Firewall Basic, manage it via a centralized Firewall Policy, and enforce outbound filtering + inbound DNAT for a workload VM — reinforcing AZ-500 Domain 4 (network security) concepts.

## Architecture

| Component | Value |
|---|---|
| Resource Group | Test-FW-RG |
| VNet | Test-FW-VN (10.0.0.0/16) |
| AzureFirewallSubnet | 10.0.0.0/26 |
| AzureFirewallManagementSubnet | 10.0.1.0/26 |
| Workload-SN | 10.0.2.0/24 |
| Firewall | Test-FW01 (Basic SKU) |
| Firewall Policy | fw-test-pol |
| Workload VM | Srv-Work (10.0.2.4) |
| Firewall Public IP (fw-pip) | 20.120.37.159 |

## Steps Performed

### 1. Resource Group & Firewall Deployment
Created `Test-FW-RG`, deployed `Test-FW01` (Basic tier) with a new Firewall Policy (`fw-test-pol`), new VNet (`Test-FW-VN`), and both required public IPs (data-plane `fw-pip` + mandatory management IP for Basic SKU).

### 2. Workload Subnet & VM
Added `Workload-SN` (10.0.2.0/24) to the VNet. Deployed `Srv-Work` (Windows Server 2019) into this subnet with **no public IP** and **no inbound NSG rules** — all access routed through the firewall.

Verified VM network config via PowerShell:

```
PS C:\Windows\system32> whoami
srv-work\azureuser
PS C:\Windows\system32> hostname
Srv-Work
PS C:\Windows\system32> ipconfig
IPv4 Address. . . . . . . . . . . : 10.0.2.4
Subnet Mask . . . . . . . . . . . : 255.255.255.0
Default Gateway . . . . . . . . . : 10.0.2.1
```

### 3. Route Table (force-tunnel through firewall)
Created `Firewall-route`, associated to `Workload-SN` only. Added default route:

| Setting | Value |
|---|---|
| Destination | 0.0.0.0/0 |
| Next hop type | Virtual appliance |
| Next hop address | 10.0.2.4 *(verify — should be firewall's private IP, not VM IP)* |

### 4. Application Rule
`fw-test-pol` → Application Rules → `App-Coll01` (priority 200):

| Rule | Source | Protocol | Destination | Action |
|---|---|---|---|---|
| Allow-Google | 10.0.2.0/24 | Https:443, Http:80 | www.google.com | Allow |

### 5. Network Rule
`fw-test-pol` → Network Rules → `Net-Coll01` (priority 200, DefaultNetworkRuleCollectionGroup):

| Rule | Source | Port | Protocol | Destination |
|---|---|---|---|---|
| Allow-DNS | 10.0.2.0/24 | 53 | UDP | 209.244.0.3, 209.244.0.4 |

### 6. DNAT Rule
`fw-test-pol` → DNAT Rules → `rdp` (priority 200, DefaultDnatRuleCollectionGroup):

| Rule | Source | Port | Protocol | Destination | Translated |
|---|---|---|---|---|---|
| RDP-NAT | * | 3389 | TCP | 20.120.37.159 (fw-pip) | 10.0.2.4:3389 |

### 7. DNS Configuration on VM NIC
Set custom DNS servers on `srv-work481` NIC to match the network rule's allowed DNS IPs:
- 209.244.0.3
- 209.244.0.4

## Verification
- RDP to `20.120.37.159` (firewall public IP) successfully lands on `Srv-Work` via the DNAT rule.
- Browsing `https://www.google.com` succeeds (application rule allow).
- Browsing any non-whitelisted FQDN is blocked, confirming default-deny behavior of the app rule collection.

Screenshots:
<img width="1357" height="536" alt="image" src="https://github.com/user-attachments/assets/9e46ea4d-cc30-4312-a392-b08e1148e94f" />
<img width="1364" height="433" alt="image" src="https://github.com/user-attachments/assets/1647ae75-1492-4a8a-b6f3-43e399c6e282" />
<img width="1365" height="363" alt="image" src="https://github.com/user-attachments/assets/5b97ac5a-79f5-4e49-a58c-b058cec4669b" />
<img width="1365" height="362" alt="image" src="https://github.com/user-attachments/assets/22411ef8-75a6-4119-a807-eb78568c87f4" />
<img width="1363" height="649" alt="image" src="https://github.com/user-attachments/assets/f7965b9b-09e6-4ad8-aa72-542bb36d8cac" />
<img width="910" height="506" alt="image" src="https://github.com/user-attachments/assets/8208ff81-e83d-4d70-bd99-a7a6073a1f58" />
<img width="1334" height="667" alt="image" src="https://github.com/user-attachments/assets/d54a47cf-056c-4680-bfc7-e4dba4736d16" />


## Key Takeaways
- Basic SKU **requires** a dedicated AzureFirewallManagementSubnet — Standard/Premium do not.
- Network rules take precedence over application rules regardless of priority.
- UDR must be associated **only** to the workload subnet, never to AzureFirewallSubnet.
- Basic SKU policy uses pre-provisioned default rule collection groups (`DefaultNetworkRuleCollectionGroup`, `DefaultApplicationRuleCollectionGroup`, `DefaultDnatRuleCollectionGroup`).

## Reference
[Deploy and configure Azure Firewall Basic using Azure Portal](https://learn.microsoft.com/en-us/azure/firewall/deploy-firewall-basic-portal-policy)
