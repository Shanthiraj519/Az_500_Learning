# Filtering Network Traffic with an NSG — Setup Notes

Reference: [Tutorial: Filter network traffic with a network security group](https://learn.microsoft.com/en-us/azure/virtual-network/tutorial-filter-network-traffic?tabs=portal) (Microsoft Learn)

This document records the steps completed for this deployment, based on the official Microsoft tutorial above.

## Resources Created

| Resource | Name | Notes |
|---|---|---|
| Virtual Network | `vnet-1` | Region: East US |
| Subnet | `subnet-1` | Inside `vnet-1` |
| Application Security Group | `asg-web` | Groups VMs serving web traffic (port 80) |
| Application Security Group | `asg-mgmt` | Groups VMs serving management traffic (port 8080) |
| Network Security Group | `nsg-1` | Associated with `subnet-1` |
| Virtual Machine | `test-vm` (vm-web) | NIC attached to `asg-web` |
| Virtual Machine | `vm-mgmt` | NIC attached to `asg-mgmt` |

## NSG Inbound Security Rules (`nsg-1`)

| Rule Name | Destination (ASG) | Port | Protocol | Source |
|---|---|---|---|---|
| Allow-Web-All | `asg-web` | 80 | TCP | Any |
| Allow-Mgmt-All | `asg-mgmt` | 8080 | TCP | Any |
| Allow-SSH-* | Any / Any | 22 | TCP | My IP or Any (temporary, for setup access) |

> **Note:** The SSH rule was added temporarily to allow initial configuration access. Consider removing it or restricting it to a specific IP range once setup is complete, since leaving port 22 open to "Any" is a standing security risk.

## Setup Steps Performed

1. Created the virtual network and subnet.
2. Created the two application security groups (`asg-web`, `asg-mgmt`).
3. Created `nsg-1` and associated it with `subnet-1`.
4. Added inbound rules for port 80 (→ `asg-web`) and port 8080 (→ `asg-mgmt`).
5. Created `test-vm`, attached its NIC to `asg-web`.
6. Created `vm-mgmt`, attached its NIC to `asg-mgmt`.
7. Installed and started nginx on both VMs using the Azure **Run Command** feature (avoided needing direct SSH access):

   **On `test-vm` (port 80, default):**
   ```bash
   sudo apt-get update -y
   sudo apt-get install -y nginx
   sudo systemctl enable nginx
   sudo systemctl start nginx
   ```

   **On `vm-mgmt` (reconfigured to listen on 8080):**
   ```bash
   sudo apt-get update -y
   sudo apt-get install -y nginx
   sudo sed -i 's/listen 80 default_server;/listen 8080 default_server;/' /etc/nginx/sites-available/default
   sudo sed -i 's/listen \[::\]:80 default_server;/listen [::]:8080 default_server;/' /etc/nginx/sites-available/default
   sudo systemctl enable nginx
   sudo systemctl restart nginx
   ```

## Verification / Expected Results

| Test | Expected Result |
|---|---|
| `http://<test-vm-ip>` (port 80) | nginx welcome page loads |
| `http://<test-vm-ip>:8080` | Connection times out (not in `asg-web` rule) |
| `http://<vm-mgmt-ip>:8080` | nginx welcome page loads |
| `http://<vm-mgmt-ip>` (port 80) | Connection times out (not in `asg-mgmt` rule) |

This confirms the NSG rules, scoped by application security group, are correctly filtering traffic to each VM based on its assigned role.

## Troubleshooting Notes (issues hit during setup)

- **`ERR_SSL_PROTOCOL_ERROR` on port 8080** — caused by the browser auto-prepending `https://`. Fixed by explicitly typing `http://` before the IP address.
- **SSH inaccessible** — `nsg-1` only allowed ports 80/8080 at the subnet level, so port 22 was blocked by NSG default-deny rules. Resolved by adding a temporary inbound rule for port 22.
- **SSH connection hung with no prompt** — caused by scoping the SSH rule's source to "My IP," which didn't match Cloud Shell's actual outbound IP (Cloud Shell runs from Azure's network, not the local browser's network). Temporarily widened the rule to "Any" to connect, or used **Run Command** instead of SSH entirely.
- **Case-sensitive filenames in Cloud Shell** — uploaded key file was `Test-keys.pem`, but a lowercase `test-keys.pem` was typed, causing "No such file or directory."
- **Port 80 not reachable on `test-vm` after nginx install** — traced to the port 80 rule accidentally being configured for port 8080 destination instead. Corrected and confirmed working.

## Cleanup 

Screenshots:
<img width="940" height="379" alt="image" src="https://github.com/user-attachments/assets/8fb723ba-5087-4770-a4f8-d526fd3c3bba" />
<img width="938" height="422" alt="image" src="https://github.com/user-attachments/assets/990c4829-6280-4af9-b537-eddc9c4c3699" />
<img width="940" height="418" alt="image" src="https://github.com/user-attachments/assets/12413455-f0cd-42f9-97a6-ab14ae9e35f8" />
<img width="830" height="287" alt="image" src="https://github.com/user-attachments/assets/ef8ab76a-8200-4f60-ae7a-4e05601f34a1" />
<img width="716" height="268" alt="image" src="https://github.com/user-attachments/assets/04b3e368-85b2-4dbc-bf4f-2cdb1fc78ac8" />






Once testing and validation are complete, delete the resource group containing these resources to avoid ongoing charges for VMs and public IPs.
