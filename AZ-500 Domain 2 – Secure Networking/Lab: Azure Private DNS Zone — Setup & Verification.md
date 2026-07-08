# Lab: Azure Private DNS Zone — Setup & Verification

## Objective
Deploy a Private DNS zone, link it to a VNet with auto-registration enabled, verify automatic A-record creation for a VM, and confirm name resolution across a manually added record.

## Environment
| Component        | Value                     |
|-------------------|---------------------------|
| Private DNS Zone  | `Private.learnig.co.in`   |
| Virtual Network   | `myvnet`                  |
| Subnet            | `Mysubnet` (10.2.0.0/24, 250 available IPs) |
| VNet Link Name    | `myventlink`               |
| Auto-Registration | Enabled                   |
| Fallback to Internet | Disabled                |
| Test VM           | `MyVM-01` (10.2.0.4)      |

## Steps Performed

1. **Created Private DNS Zone** — `Private.learnig.co.in`
2. **Linked VNet to zone**
   - Link name: `myventlink`
   - Virtual Network: `myvnet`
   - Auto-Registration: **Enabled**
   - Fallback to Internet DNS: **Disabled**
   - Link Status: `Completed`
3. **Deployed test VM** (`MyVM-01`) into `myvnet` / `Mysubnet` — no public IP, accessed via Run Command.
4. **Verified auto-registration** — Azure automatically created an A record for the VM:

   | Name      | Type | TTL | Value    | Auto Registered |
   |-----------|------|-----|----------|------------------|
   | myvm-01   | A    | 10  | 10.2.0.4 | **True**         |

5. **Added manual A record** — `db.learnig.co.in` → `10.2.0.4` (TTL 60, Auto Registered: **False**)

## Verification — DNS Resolution Test (Run Command → RunPowerShellScript)

Ping test against the manually created record resolved correctly to the VM's private IP:

Screenshots:
<img width="1350" height="602" alt="image" src="https://github.com/user-attachments/assets/cdbf6ce6-6477-45b6-8e16-a51d2bb54a35" />
<img width="1356" height="590" alt="image" src="https://github.com/user-attachments/assets/5d676944-5fcc-4c99-9119-22de24efd2d1" />
<img width="1365" height="644" alt="image" src="https://github.com/user-attachments/assets/03c72448-8e3c-4886-a34d-34ff11db47c7" />
<img width="1365" height="582" alt="image" src="https://github.com/user-attachments/assets/bbd483de-7830-4d63-b7a7-09141396a7a2" />
