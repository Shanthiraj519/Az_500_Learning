# Lab– Azure Bastion (Secure VM Access)

Ref: https://learn.microsoft.com/en-us/azure/bastion/quickstart-host-portal?tabs=default

## Objective
Deploy Azure Bastion to enable secure RDP/SSH access to a VM over its private IP, eliminating the need for a public IP address on the VM itself.

## Environment
| Resource | Name | Type | Location |
|---|---|---|---|
| Resource Group | Test-Rg | Resource group | East US |
| Virtual Machine | myvm | Linux (Ubuntu 24.04), Standard_D2s_v3 | East US |
| Virtual Network | myvm-vnet | Virtual network | East US |
| NSG | myvm-nsg | Network security group | East US |
| Bastion Host | myvm-vnet-bastion | Bastion | East US |
| Public IP | myvm-vnet-IPv4 | Public IP address (Bastion) | East US |
| NIC | myvm654 | Network interface | East US |

## Steps Performed
1. Navigated to the VM (`myvm`) or VNet (`myvm-vnet`) → **Connect** → **Bastion**.
2. Selected **Deploy Bastion** using default settings (Standard SKU).
3. Deployment provisioned `myvm-vnet-bastion` along with a dedicated public IP (`myvm-vnet-IPv4`) in a `AzureBastionSubnet` within `myvm-vnet`.
4. Verified the Bastion resource appeared under **Test-Rg** with type `Bastion`.
5. Removed the VM's own public IP association (dissociated and deleted) since Bastion now handles connectivity via private IP.
6. Confirmed via VM **Overview** pane that `myvm` no longer shows a Primary NIC public IP.

## Observation / Note
While reviewing the VM Overview pane, a platform banner flagged that the VM currently relies on **default outbound access**, which Azure is deprecating for new subnets after March 2026. Guidance recommends explicitly configuring outbound connectivity (e.g., NAT Gateway or an explicit public IP) and setting subnets to private, rather than relying on default outbound behavior.

This is a good tie-in to AZ-500 Domain 4 (secure networking) — explicit outbound methods vs. default outbound access is worth a dedicated concept review.

## Verification
- Bastion resource `myvm-vnet-bastion` visible and listed as type **Bastion** in the resource group.
- VM `myvm` has no public IP attached (Primary NIC public IP shows blank).
- VM remains reachable via Bastion using its private IP over RDP/SSH through the Azure portal.

## Key Takeaways
- Azure Bastion provides browser-based RDP/SSH to VMs over their **private IP**, removing the need to expose a public IP on the VM.
- Default Bastion deployment (Standard SKU) provisions its own dedicated public IP and subnet (`AzureBastionSubnet`) — separate from the VM's networking.
- Removing the VM's public IP after Bastion deployment reduces attack surface — a key defense-in-depth practice for AZ-500 Domain 4.
- Default outbound access is being deprecated (March 2026) — plan to configure explicit outbound methods on future labs/VMs going forward.

## Status
<img width="1568" height="652" alt="image" src="https://github.com/user-attachments/assets/97a0b4c6-b754-4c9b-8551-87d13b56ebd2" />
<img width="1568" height="680" alt="image" src="https://github.com/user-attachments/assets/ff6be6b3-1cfe-4168-b7ef-9dfba1b4c023" />

✅ Lab 10 (Bastion) — Complete
