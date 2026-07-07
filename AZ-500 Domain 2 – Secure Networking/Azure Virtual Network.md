# Lab – Create an Azure Virtual Network

## Objective
Deploy a Virtual Network (VNet) with a subnet, secure it with an NSG, connect it via Azure Bastion, and validate private connectivity between two VMs in the same VNet.

## Resources Used

| Resource               | Name         |
|------------------------|--------------|
| Resource Group         | test-rg      |
| Region                 | East US 2    |
| Virtual Network        | vnet-1       |
| Subnet                 | subnet-1     |
| Network Security Group | nsg-1        |
| Bastion Host           | bastion      |
| Virtual Machine 1      | vm-1         |
| Virtual Machine 2      | vm-2         |

## Steps Performed

### 1. Resource Group
- Created a resource group `test-rg` in **East US 2** to contain all lab resources.

### 2. Virtual Network
- Created `vnet-1` with default address space `10.0.0.0/24`.
- Configured subnet `subnet-1` within the VNet using the default subnet template.

### 3. Azure Bastion
- Deployed `bastion` (Developer tier) inside `vnet-1` to enable secure SSH/RDP access without exposing VMs via public IPs.

### 4. Virtual Machines
- Deployed `vm-1` and `vm-2` (Ubuntu Server 22.04 LTS) into `subnet-1`.
- No public IPs assigned — VMs rely on Bastion for access and default outbound access for internet egress.
- Created NSG `nsg-1` and attached it to both VMs' NICs (Advanced networking config).
- Authentication via SSH key pairs generated per VM.

### 5. Connectivity Validation
- Connected to `vm-1` via **Bastion**.
- Ran `ping -c 4 vm-2` — successful reply confirming private (10.0.0.x) connectivity within the VNet.
- Repeated in reverse from `vm-2` to `vm-1` — successful.

```output
azureuser@vm-1:~$ ping -c 4 vm-2
64 bytes from vm-2.internal.cloudapp.net (10.0.0.5): icmp_seq=1 ttl=64 time=1.83 ms
...
```

## Key Learnings
- VMs behind Bastion don't need public IPs — Bastion supplies the browser-based secure tunnel while VMs communicate over private IPs.
- Default outbound access is automatically granted to VMs without a public IP, NAT Gateway, or standard LB backend membership — but this mechanism is non-configurable and not meant for production use.
- NSG association at the NIC level (vs subnet level) gives more granular control per VM.
- Successful intra-VNet ping confirms name resolution and routing work out-of-the-box for VMs in the same VNet/subnet.

## Cleanup
- Deleted `test-rg` resource group to remove all associated resources (VNet, Bastion, VMs, NSG) and avoid ongoing Bastion hourly charges.

## Reference
- [Microsoft Learn – Quickstart: Create an Azure Virtual Network](https://learn.microsoft.com/en-us/azure/virtual-network/quickstart-create-virtual-network?tabs=portal)
