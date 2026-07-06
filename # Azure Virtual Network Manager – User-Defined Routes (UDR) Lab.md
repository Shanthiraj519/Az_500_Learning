# Azure Virtual Network Manager – User-Defined Routes (UDR) Lab

This document walks through deploying centrally managed **User-Defined Routes (UDRs)** using **Azure Virtual Network Manager (AVNM)**, based on a hands-on lab performed in a personal Azure sandbox.

## Overview

Azure Virtual Network Manager lets you define routing behavior once and have it automatically applied across a group of virtual networks — instead of manually creating and maintaining route tables on each subnet. This is achieved through three building blocks:

1. **Network group** – a collection of VNets, populated manually or dynamically via Azure Policy.
2. **Routing configuration** – a container for routing rules.
3. **Rule collection** – a set of rules applied to a target network group.

## Prerequisites

- An active Azure subscription with **Network Contributor** (or equivalent) permissions on the target scope.
- An existing **Virtual Network Manager** instance with the **User Defined Routing** and **Connectivity** features enabled.
  - In this lab: `NetworkManager-Test`
- At least one virtual network to act as a group member.
  - In this lab: `Vnet2-Test`

## Step 1: Create a Network Group

A network group defines the set of VNets that routing rules will apply to.

1. Navigate to **Resource groups** → your resource group → your **Network Manager** instance (`NetworkManager-Test`).
2. Under **Settings**, select **Network groups** → **+ Create**.
3. Configure:

   | Setting | Value |
   |---|---|
   | Name | `NetworkGroup-Test` |
   | Description | *(optional)* |
   | Member type | Virtual network |

4. Select **Create**.

## Step 2: Populate the Group with Azure Policy

Rather than manually adding VNets, an Azure Policy can dynamically discover and add matching VNets to the group.

1. Open **NetworkGroup-Test** → select **Create Azure Policy**.
2. Configure:

   | Setting | Value |
   |---|---|
   | Policy name | `PolicyForNetworkManager` |
   | Scope | Subscription containing the target VNets |

3. Under **Criteria**, define a conditional expression to match VNets, for example:

   | Parameter | Operator | Condition |
   |---|---|---|
   | Name | Contains | `vnet` |

4. Select **Preview Resources** to confirm the correct VNets are matched.
5. Select **Save** to create and apply the policy.

Once evaluated, matching VNets appear automatically under **Group members**. In this lab, the policy correctly discovered and added `Vnet2-Test` as a group member.

> **Note:** Policy evaluation isn't always instant — allow a few minutes and refresh the **Group members** page if expected VNets don't appear right away. Also confirm the policy's scope actually includes the subscription/resource group where those VNets live.

## Step 3: Create a Routing Configuration

The routing configuration holds one or more rule collections that get applied to network groups.

1. In the Network Manager instance, go to **Settings** → **Configurations** → **+ Create** → **Create routing configuration**.
2. Configure:

   | Setting | Value |
   |---|---|
   | Name | `Routing-configuration-01` |
   | Description | *(optional)* |

3. Move to **Rule collections** → **+ Add**.
4. Configure the rule collection:

   | Setting | Value |
   |---|---|
   | Name | `Rule-collection-1` |
   | Description | *(optional)* |
   | Enable BGP route propagation | Unchecked |
   | Target network groups | `NetworkGroup-Test` |

5. Under **Routing rules**, select **+ Add** and configure the rule:

   | Setting | Value |
   |---|---|
   | Name | `Rr-spoke` |
   | Destination type | IP address |
   | Destination IP addresses/CIDR ranges | `0.0.0.0/0` |
   | Next hop type | Virtual network |

   This rule forces **all outbound traffic** (`0.0.0.0/0` matches any destination) from member VNets to route through a designated virtual network — a common pattern in hub-and-spoke topologies, where spoke VNets send traffic through a central hub (e.g., for firewall inspection or centralized egress).

6. Select **Add** to save the rule, then **Add** again to save the rule collection.
7. Select **Review + create** → **Create** to finalize the routing configuration.

## Step 4: Deploy the Routing Configuration

Creating the configuration does **not** apply it — it must be explicitly deployed to generate the actual UDRs.

1. On the **Configurations** page, select the checkbox next to `Routing-configuration-01`.
2. Select **Deploy** from the toolbar.
3. Configure:

   | Setting | Value |
   |---|---|
   | Include user-defined routing configurations in goal state | Checked |
   | User defined routing configurations | `Routing-configuration-01` |
   | Target regions | Region of the target VNets (e.g., East US) |

4. Select **Next** → **Deploy**.

> **Caution:** Deploying a routing configuration can override existing custom route tables on affected subnets. Review the impact of existing routes before deploying in a production environment.

## Result

After deployment, Azure Virtual Network Manager automatically creates and maintains the UDRs on all VNets that are members of `NetworkGroup-Test` — currently `Vnet2-Test` — routing all outbound traffic (`0.0.0.0/0`) to the specified next-hop virtual network, with no manual route table configuration required.
<img width="948" height="406" alt="image" src="https://github.com/user-attachments/assets/6af57eed-3f51-47c9-89e3-1f7c90bf4068" />
<img width="959" height="452" alt="image" src="https://github.com/user-attachments/assets/72564fc0-fbfe-4c46-bf12-d5523269f8df" />
<img width="959" height="424" alt="image" src="https://github.com/user-attachments/assets/bdc66c83-552b-4a98-8372-aef26eebd204" />

<img width="955" height="440" alt="image" src="https://github.com/user-attachments/assets/9946de3e-e9a4-4589-bd94-9a830d4b4d9d" />
<img width="956" height="452" alt="deployement" src="https://github.com/user-attachments/assets/9d4450c3-d402-4c62-a34c-c4859a2919e1" />




## References

- [Azure Virtual Network Manager – User-Defined Routes overview](https://learn.microsoft.com/en-us/azure/virtual-network-manager/concept-user-defined-route-management)
- [Create User-Defined Routes with Azure Virtual Network Manager](https://learn.microsoft.com/en-us/azure/virtual-network-manager/how-to-create-user-defined-route)
- [Impacts of user-defined routes](https://learn.microsoft.com/en-us/azure/virtual-network-manager/concept-user-defined-route)
