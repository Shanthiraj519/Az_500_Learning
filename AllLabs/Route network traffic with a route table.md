# Azure Route Table Tutorial — Verification Documentation

Reference: [Tutorial: Route network traffic with a route table](https://learn.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table?tabs=portal)

## Resource Group Overview

Resource group: **test-rg** (East US)

| Resource | Type |
|---|---|
| Bastion | Bastion |
| nsg-nva | Network security group |
| route-table-public | Route table |
| vm-nva | Virtual machine |
| vm-nva-ip | Public IP address |
| vm-nva131 | Network interface |
| vm-nva_key | SSH key |
| vm-private | Virtual machine |
| vm-private-ip | Public IP address |

## Virtual Network Subnets (vnet-1)

| Subnet name | Address range | Available IPs | Route table |
|---|---|---|---|
| sub-private | 10.0.2.0/24 | 250 | – |
| subnet-dmz | 10.0.3.0/24 | 250 | – |
| AzureBastionSubnet | 10.0.1.0/26 | 57 | – |
| subnet-1 | 10.0.0.0/24 | 250 | route-table-public |

## Route Table Configuration

**Route table:** `route-table-public`
**Associated subnet:** `subnet-1`

| Route name | Address prefix | Next hop type | Next hop IP address |
|---|---|---|---|
| to-private-subnet | 10.0.2.0/24 | Virtual appliance | 10.0.3.4 |

> **Note:** IP forwarding must be enabled on the NVA's network interface for traffic forwarding to work correctly.

## Verification

### From vm-public → vm-private

```bash
azureuser@vm-public:~$ tracepath vm-private
 1?: [LOCALHOST]                      pmtu 1500
 1:  vm-nva.internal.cloudapp.net       0.974ms
 1:  vm-nva.internal.cloudapp.net       0.878ms
 2:  vm-private.internal.cloudapp.net   1.322ms reached
    Resume: pmtu 1500 hops 2 back 1
```

Traffic from `vm-public` to `vm-private` is routed through `vm-nva` (hop 1), confirming the user-defined route is working as expected.

### From vm-private → vm-public

```bash
azureuser@vm-private:~$ tracepath vm-public
 1?: [LOCALHOST]                      pmtu 1500
 1:  vm-public.internal.cloudapp.net    1.601ms reached
 1:  vm-public.internal.cloudapp.net    1.382ms reached
    Resume: pmtu 1500 hops 1 back 2
```

Return traffic from `vm-private` to `vm-public` takes a direct path (1 hop) since `sub-private` / `subnet-dmz` has no UDR forcing traffic back through the NVA.

## Result
Screenshots:<img width="1568" height="712" alt="image" src="https://github.com/user-attachments/assets/3e10e5c9-4fcb-408b-9d36-a230b8b6e746" />
<img width="1477" height="812" alt="image" src="https://github.com/user-attachments/assets/55499d0c-66a6-4526-a9d0-c50ed7ad652e" />
<img width="1568" height="676" alt="image" src="https://github.com/user-attachments/assets/dc09b8bc-1a58-49df-a4df-49bce691c8f2" />
<img width="1568" height="686" alt="image" src="https://github.com/user-attachments/assets/0f5f3036-69ab-4e8e-a032-63723c257915" />
<img width="1568" height="611" alt="image" src="https://github.com/user-attachments/assets/d0b4827a-2682-4867-b27e-8c4e1ccff32c" />
<img width="1568" height="593" alt="image" src="https://github.com/user-attachments/assets/536b5f03-763f-4c4f-95af-40cdef2f0186" />
<img width="953" height="399" alt="image" src="https://github.com/user-attachments/assets/251073b3-a0dc-4a22-b3a1-bcfa8bce1307" />

Traffic from `vm-public` to `vm-private` is successfully routed through the NVA (`vm-nva`) at `10.0.3.4`, confirming the route table is correctly directing traffic as configured in the tutorial.

### Notes / Follow-ups

- The route table is only associated with `subnet-1`, and only one route exists (`10.0.2.0/24` → NVA). Return traffic is **not** forced back through the NVA, which is why the reverse tracepath shows 1 hop instead of 2 (asymmetric routing). Add a note if symmetric routing is required for your scenario.
- Confirm IP forwarding is enabled on `vm-nva131`'s NIC — this is a required step in the tutorial and isn't visible in the screenshots captured here.
