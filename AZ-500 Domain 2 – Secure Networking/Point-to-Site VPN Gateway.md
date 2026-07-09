# Lab 09: Point-to-Site VPN Gateway — Full Step-by-Step (Replay Guide)

## Prerequisites
- Resource Group: `TestRG1`
- VNet: `VNet1` (address space `10.1.0.0/18`), East US
- On-prem simulation: Windows Server DC (`DC-Prx-02`, Proxmox nested in Hyper-V), `192.168.31.11`

---

## Part 1: VPN Gateway Creation

### 1.1 Initial gateway (created via Portal)
- Name: `Vnet1GW`
- SKU: `VpnGw2AZ` (zone-redundant)
- VPN type: Route-based
- Attached to `VNet1`
- Public IP: `VnetGWPIP1`

> ⚠️ **Lesson learned**: If you create this via Portal with default settings on a Basic/Zone-redundant SKU, it may provision in **Active-Active mode**, which requires exactly 3 IP configurations for P2S. Avoid this by specifying only one public IP at creation (see 1.2 for the CLI recreate that fixed this).

### 1.2 Check provisioning status
```bash
az network vnet-gateway show \
  --resource-group TestRG1 \
  --name Vnet1GW \
  --query provisioningState -o tsv
```
Wait until it returns `Succeeded` (takes 30–45 min).

### 1.3 If gateway ends up Active-Active and blocks P2S (as happened here)

Confirm current IP configs:
```bash
az network vnet-gateway show \
  --resource-group TestRG1 \
  --name Vnet1GW \
  --query "ipConfigurations[].name" -o tsv
```
If this returns **two** entries (e.g. `default`, `activeActive`), you're in Active-Active mode.

Attempting to fix in place will fail:
```bash
# This fails — Azure won't downgrade Active-Active → Active-Standby on an existing gateway
az network vnet-gateway update \
  --resource-group TestRG1 \
  --name Vnet1GW \
  --public-ip-address VnetGWPIP1
# Error: PrimaryIpConfigurationCannotBeChanged
```

**Fix: delete and recreate with a single public IP.**
```bash
az network vnet-gateway delete --resource-group TestRG1 --name Vnet1GW
```
(~10–15 min)

```bash
az network vnet-gateway create \
  --resource-group TestRG1 \
  --name Vnet1GW \
  --vnet VNet1 \
  --public-ip-address VnetGWPIP1 \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw2AZ \
  --no-wait
```
> Note: Non-AZ SKUs (VpnGw1–5) are deprecated — only `*AZ` SKUs can be created now. Passing a single `--public-ip-address` (not two) keeps it in Active-Standby by default.

Poll again until `Succeeded`:
```bash
az network vnet-gateway show \
  --resource-group TestRG1 \
  --name Vnet1GW \
  --query provisioningState -o tsv
```

---

## Part 2: Generate Certificates (Cloud Shell — OpenSSL, most reliable method)

> ⚠️ **Lesson learned**: Generating certs in Windows PowerShell and transferring the base64 text via Notepad/portal-paste repeatedly caused corruption (`Data for certificate ... is invalid` / `VpnClientRootCertificateExpired` errors) even though the certs themselves were valid. Generating directly in Cloud Shell and referencing files (not manual paste) avoided this entirely.

Run in **Azure Cloud Shell (Bash)**:

```bash
# Root cert
openssl genrsa -out P2SRoot.key 2048
openssl req -x509 -new -nodes -key P2SRoot.key -sha256 -days 1825 \
  -out P2SRoot.pem -subj "/CN=P2SRootCertFinal"
openssl x509 -in P2SRoot.pem -outform der -out P2SRoot.cer
base64 -w 0 P2SRoot.cer > P2SRootBase64.txt

# Client cert (signed by root)
openssl genrsa -out P2SClient.key 2048
openssl req -new -key P2SClient.key -out P2SClient.csr -subj "/CN=P2SClientCertFinal"
openssl x509 -req -in P2SClient.csr -CA P2SRoot.pem -CAkey P2SRoot.key \
  -CAcreateserial -out P2SClient.pem -days 365 -sha256

# Bundle client cert + key for installation on DC-Prx-02
openssl pkcs12 -export -in P2SClient.pem -inkey P2SClient.key \
  -certfile P2SRoot.pem -out P2SClient.p12 -password pass:TempPass123!
```

Confirm files exist:
```bash
ls -la ~
```

---

## Part 3: Configure P2S on the Gateway (CLI — more reliable than Portal)

> ⚠️ **Lesson learned**: The Portal's Point-to-site configuration page repeatedly threw `Deployment validation failed` with no useful detail, and separately `File download error` when trying to download the VPN client. Doing this via CLI was more transparent and let us see the real error codes (`VpnClientAddressPoolNotSpecified`, `ActiveActiveP2SGatewayMustHaveExactlyThreeIpConfigurations`).

### 3.1 Set address pool + protocol
```bash
az network vnet-gateway update \
  --resource-group TestRG1 \
  --name Vnet1GW \
  --address-prefixes 172.16.201.0/24 \
  --client-protocol OpenVPN
```
> Address pool must **not overlap** VNet address space (`10.1.0.0/18`) or on-prem network (`192.168.31.0/24`).

### 3.2 Add the root certificate (file reference, not manual paste)
```bash
az network vnet-gateway root-cert create \
  --resource-group TestRG1 \
  --gateway-name Vnet1GW \
  --name P2SRootCertFinal \
  --public-cert-data P2SRootBase64.txt
```

### 3.3 Verify full P2S config
```bash
az network vnet-gateway show \
  --resource-group TestRG1 \
  --name Vnet1GW \
  --query vpnClientConfiguration
```
Confirm `vpnClientAddressPool`, `vpnClientProtocols: ["OpenVPN"]`, and `vpnClientRootCertificates` all show correctly.

---

## Part 4: Generate & Download VPN Client Config

```bash
az network vnet-gateway vpn-client generate \
  --resource-group TestRG1 \
  --name Vnet1GW \
  --authentication-method EAPTLS
```
Returns a signed download URL (valid ~1hr) — open in a browser to download `vpnclientconfiguration.zip`. Extract it; contains `azurevpnconfig.xml` used in Part 6.

---

## Part 5: Transfer Client Certificate to DC-Prx-02

1. In Cloud Shell, use the **Manage files → Download** option in the toolbar (not the terminal) to download `P2SClient.p12` to your local PC.
   - If the download path dialog splits into two fields, clear both and retype the **full path** fresh: `/home/<username>/P2SClient.p12`
2. Transfer the `.p12` from your local PC to DC-Prx-02 (via RDP clipboard/drive redirection, or upload to reachable storage and download from within the VM).
3. On DC-Prx-02, import it:
```powershell
Import-PfxCertificate `
  -FilePath "C:\Path\To\P2SClient.p12" `
  -CertStoreLocation Cert:\CurrentUser\My `
  -Password (ConvertTo-SecureString -String "TempPass123!" -Force -AsPlainText)
```
4. Verify import + private key:
```powershell
$cert = Get-ChildItem Cert:\CurrentUser\My | Where-Object {$_.Subject -eq "CN=P2SClientCertFinal"}
$cert.HasPrivateKey   # must return True
```
5. Clean up any old/mismatched certs from earlier attempts to avoid ambiguity:
```powershell
Get-ChildItem Cert:\CurrentUser\My | Where-Object {$_.Subject -in @("CN=P2SClientCert","CN=P2SClientCert2","CN=P2SRootCert","CN=P2SRootCert2")} | Remove-Item
```

---

## Part 6: Install Azure VPN Client on Windows Server

> ⚠️ **Lesson learned**: Standard Microsoft Store install links (`https://go.microsoft.com/fwlink/?linkid=2117554`) redirect to the Store, which often isn't accessible on Windows Server. Use the sideload package instead.

1. Download the sideload package: search "Azure VPN Client sideload download" or use the direct MSIX bundle package (`AzVpnAppx_x.x.x_sideload.zip`).
2. Extract, then run as Administrator:
```powershell
cd "C:\Path\To\AzVpnAppx_x.x.x_sideload"
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force
.\Install.ps1
```
Confirms with: `Success: Your app was successfully installed.`

---

## Part 7: Import Profile & Connect

1. Open **Azure VPN Client** app
2. Click **+ Add** → **Import**
3. Browse to the extracted `azurevpnconfig.xml` from Part 4
4. Confirm auto-filled **Connection Name** and **VPN Server** fields, click **Save**
5. Select the profile → **Connect**

---

## Part 8: Troubleshooting Failed Connection

Result: `Server did not respond properly to VPN Control Packets. Session State: Key Material sent`

### 8.1 Rule out stale config
Regenerate a fresh client config (repeat Part 4), remove old profile in the app, re-import, retry connect.

### 8.2 Verify network path is not blocked outright
```powershell
Test-NetConnection -ComputerName <GatewayPublicIP> -Port 443
```
`TcpTestSucceeded: True` confirms basic reachability (does NOT confirm full handshake success).

```powershell
Get-NetFirewallRule -Direction Outbound -Enabled True | Where-Object {$_.Action -eq "Block"}
```
Confirms no outbound blocks on the VM itself.

### 8.3 Diagnose deeper routing issues
```powershell
tracert <GatewayPublicIP>
```
Look for:
- Repeated private IPs early in the trace (e.g. `192.0.0.1` multiple times) → CGNAT from ISP
- High latency / erratic international routing
- Timeouts mid-trace

**Conclusion in this lab**: Full OpenVPN key exchange failed due to multiple layers of NAT (ISP CGNAT + Hyper-V + Proxmox virtual networking) plus unstable routing — not an Azure misconfiguration. All Azure-side config (gateway, P2S settings, certs) was verified correct via `vpnClientConfiguration` query.

---

## Key Takeaways
- P2S on Active-Active gateways needs exactly 3 IP configs; Active-Standby needs only 1
- Can't downgrade Active-Active → Active-Standby via `update`; must delete/recreate
- Non-AZ VPN Gateway SKUs are deprecated; only `*AZ` SKUs available now
- Generate certs via OpenSSL in Cloud Shell and reference by file — avoids cross-platform copy/paste corruption
- P2S address pool must not overlap VNet or on-prem ranges
- TCP reachability ≠ successful VPN handshake — nested NAT can silently break control channel exchange
- Sideload the Azure VPN Client MSIX package on Windows Server if Store access is unavailable
Screenshots:
<img width="1135" height="587" alt="image" src="https://github.com/user-attachments/assets/73bc9e01-68d3-4292-a73c-0f2a0db37038" />
<img width="992" height="229" alt="image" src="https://github.com/user-attachments/assets/6fe5748a-1297-4832-9802-d4114b53bb4b" />
<img width="990" height="92" alt="image" src="https://github.com/user-attachments/assets/30ffec74-3996-496f-b5d3-30d9fa97a322" />
<img width="591" height="554" alt="image" src="https://github.com/user-attachments/assets/f9dea751-5c6f-493b-baad-09c8d20c76a4" />
<img width="1365" height="414" alt="image" src="https://github.com/user-attachments/assets/4833315a-52af-4c0e-88b6-55666a66e8c3" />
<img width="1013" height="231" alt="image" src="https://github.com/user-attachments/assets/50d9cdef-60fd-4965-a4c1-bcc03698bc4d" />
<img width="1366" height="304" alt="image" src="https://github.com/user-attachments/assets/9c3db878-ef23-40fa-846b-680bb2dd1704" />

## Deferred Next Steps
- Retest P2S from a non-nested host (e.g., physical Windows 11 machine) to confirm nested virtualization as definitive root cause
- Consider IKEv2 tunnel type as an alternative if retrying
