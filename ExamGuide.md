# Domain 1 — Secure Identity & Access (15–20%)

## Topic 1: Azure RBAC
- Built-in roles — Owner, Contributor, Reader, User Access Administrator
- Custom roles — assignable scopes, actions, notActions, dataActions
- Role assignment at management group / subscription / RG / resource level
- Difference between Azure RBAC vs Entra directory roles
- Deny assignments — what they are, when they apply

## Topic 2: Microsoft Entra PIM
- Eligible vs Active assignments
- Activation flow — MFA, approval, justification, time-bound
- PIM role settings — max activation duration, require approval, notifications
- Access Reviews — who creates, who reviews, what happens on no response
- PIM for Groups (not just roles)

## Topic 3: Conditional Access
- Include vs Exclude logic
- Sign-in risk vs User risk — which policy handles which
- Named locations — trusted IPs, countries
- Grant controls vs Session controls
- CA policy evaluation order (all matching policies apply — no priority)
- Break-glass accounts and why they're excluded

## Topic 4: MFA & Authentication
- Per-user MFA vs CA-based MFA — difference
- Authentication methods — FIDO2, Authenticator app, OATH tokens
- MFA registration policy in Entra ID Protection
- SSPR (Self-Service Password Reset) — authentication methods required

## Topic 5: Entra App Registrations & Service Principals
- App registration vs Enterprise app (service principal) — relationship
- Delegated permissions vs Application permissions
- Admin consent vs User consent
- OAuth 2.0 permission scopes
- Certificate vs client secret for app authentication

## Topic 6: Managed Identities
- System-assigned vs User-assigned — key differences
- When to use each
- How managed identity authenticates to Azure resources (no credentials in code)
- Assigning RBAC role to a managed identity

## Topic 7: Entra ID Protection
- Risk detections — anonymous IP, atypical travel, leaked credentials
- Risk policies — Sign-in risk policy vs User risk policy
- Remediation — self-remediation vs admin remediation
- Integration with Conditional Access (risk-based CA)

# Domain 2 — Secure Networking (20–25%)

## Topic 1: Network Security Groups (NSGs) & Application Security Groups (ASGs)
- NSG rules — priority, source/destination, port, protocol, allow/deny
- Default NSG rules — VNet-to-VNet, Azure Load Balancer, deny all inbound
- ASGs — group VMs by role, use ASG names in NSG rules instead of IPs
- NSG association — subnet level vs NIC level
- Effective security rules — how to troubleshoot

## Topic 2: Azure Virtual Network Manager
- Network groups — static vs dynamic membership
- Connectivity configurations — hub-and-spoke, mesh
- Security admin rules — enforce rules across subscriptions
- Difference between security admin rules vs NSG rules

## Topic 3: User-Defined Routes (UDRs)
- Route tables — custom routes override system routes
- Next hop types — Virtual appliance, VNet gateway, Internet, None
- Force tunneling — route all internet traffic through on-premises
- BGP route propagation — enable/disable per route table

## Topic 4: VNet Peering & VPN Gateway
- VNet peering — non-transitive by default
- Global VNet peering vs regional peering
- VPN Gateway SKUs — Basic, VpnGw1–5, VpnGw1AZ–5AZ
- VNet-to-VNet vs Site-to-Site vs Point-to-Site
- Active-active vs active-passive gateway configuration
- ExpressRoute — private connectivity, encryption over ExpressRoute

## Topic 5: Virtual WAN
- Virtual WAN types — Basic vs Standard
- Secured virtual hub — Azure Firewall integrated into vWAN hub
- Hub-to-hub routing
- Difference between Virtual WAN hub vs standard VNet hub-and-spoke

## Topic 6: Azure Firewall
- Azure Firewall vs NSG — layer 7 vs layer 4
- Firewall policy — rule collection groups, rule collections, rules
- DNAT, Network, Application rule collections — priority order
- Threat intelligence — alert mode vs deny mode
- Azure Firewall Manager — centrally manage policies across firewalls
- Forced tunneling support in Azure Firewall

## Topic 7: Private Access to Azure Resources
- Service Endpoints — extends VNet identity to Azure service, traffic stays on backbone
- Private Endpoints — private IP in your VNet for Azure service
- Private Link Service — expose your own service via Private Link
- Key difference — Service Endpoint vs Private Endpoint (exam trap)
- Network integration for App Service — VNet Integration vs Private Endpoint
- Azure SQL Managed Instance — network security configurations

## Topic 8: Public Access & Edge Security
- Application Gateway — layer 7 load balancer, WAF integration
- Azure Front Door — global HTTP load balancer, CDN, WAF at edge
- WAF — Prevention mode vs Detection mode, custom rules vs managed rules
- TLS termination — App Service, API Management, Application Gateway
- DDoS Protection — Basic (free) vs Standard (paid, per VNet)
- When to use Front Door vs Application Gateway vs Traffic Manager

## Topic 9: Network Watcher
- IP flow verify — test if traffic is allowed/denied by NSG
- NSG flow logs — log traffic through NSGs
- Connection troubleshoot — test connectivity between endpoints
- Packet capture — capture traffic on a VM NIC
- Topology — visualize VNet resources

---

# Domain 3 — Secure Compute, Storage & Databases (20–25%)

## Topic 1: Remote Access to Virtual Machines
- Azure Bastion — browser-based RDP/SSH, no public IP on VM
- Bastion SKUs — Basic vs Standard (features difference)
- JIT VM Access — Defender for Cloud feature, reduces attack surface
- JIT flow — request access, approved, NSG rule opens for limited time
- JIT vs Bastion — when to use which

## Topic 2: Azure Kubernetes Service (AKS) Security
- Network isolation — kubenet vs Azure CNI
- Network policies — control pod-to-pod traffic
- Private AKS cluster — API server not exposed to internet
- Authentication — Entra ID integration, RBAC for Kubernetes
- Monitoring — Defender for Containers, Container Insights
- ACR integration — pull images securely using managed identity

## Topic 3: Container Security
- Azure Container Instances (ACIs) — security monitoring, no persistent OS
- Azure Container Apps (ACAs) — managed environment, security monitoring
- Azure Container Registry (ACR) — access control, admin user vs RBAC
- Image scanning — Defender for Containers vulnerability assessment
- Private registry access — managed identity vs service principal

## Topic 4: Disk Encryption
- Azure Disk Encryption (ADE) — BitLocker (Windows) / DM-Crypt (Linux), uses Key Vault
- Encryption at host — encrypts temp disk and cache, end-to-end encryption
- Confidential disk encryption — VM-level encryption, protects from host
- Platform-managed keys (PMK) vs Customer-managed keys (CMK)
- When to use each encryption type — exam scenarios

## Topic 5: Azure API Management Security
- Authentication — subscription keys, OAuth 2.0, client certificates
- Policies — inbound, backend, outbound, on-error
- Network isolation — internal mode vs external mode
- Managed identity for backend authentication

## Topic 6: Storage Account Security
- Access control — RBAC vs access keys vs SAS tokens vs Entra ID
- Shared Access Signatures (SAS) — account SAS, service SAS, user delegation SAS
- Storage firewall — allow trusted Microsoft services
- Secure transfer required — enforce HTTPS
- Azure Files — authentication methods (Entra ID, on-premises AD, storage key)
- Blob Storage — access tiers, public access settings

## Topic 7: Data Protection for Storage
- Soft delete — blobs, containers, file shares
- Blob versioning — automatic versions on every write
- Immutable storage — time-based retention, legal hold
- Point-in-time restore for block blobs
- Backup — Azure Backup for blobs, operational backup

## Topic 8: Storage Encryption
- Encryption at rest — enabled by default, platform-managed keys
- Bring Your Own Key (BYOK) — customer-managed keys in Key Vault
- Double encryption — infrastructure-level encryption on top of service encryption
- Scope — encryption scope per container or blob

## Topic 9: Azure SQL Database Security
- Entra ID authentication — Entra-only authentication mode
- Database auditing — Log Analytics, Storage, Event Hub destinations
- Dynamic Data Masking (DDM) — mask data for non-privileged users, does not encrypt
- Transparent Data Encryption (TDE) — encrypts data at rest, on by default
- Always Encrypted — client-side encryption, protects data from DBAs
- Difference — TDE vs DDM vs Always Encrypted (exam trap)
- SQL Managed Instance — network security, VNet-native deployment

---

# Domain 4 — Defender for Cloud & Microsoft Sentinel (30–35%)

## Topic 1: Azure Policy
- Policy definitions — built-in vs custom
- Initiatives (policy sets) — group multiple policies
- Assignment scope — management group, subscription, RG
- Effects — Audit, Deny, DeployIfNotExists, Modify, Append
- Compliance — view non-compliant resources
- Remediation tasks — fix non-compliant resources via DeployIfNotExists

## Topic 2: Azure Key Vault
- Access models — Vault access policies vs Azure RBAC (prefer RBAC)
- Secrets, Keys, Certificates — what each stores
- Key types — RSA, EC, HSM-backed
- Key rotation — automatic rotation policy
- Soft delete and purge protection — prevent accidental deletion
- Backup and recovery — per-object backup, restore to same or different vault
- Network settings — firewall, private endpoint, trusted services
- Key Vault vs Managed HSM — when to use which

## Topic 3: Defender for Cloud — Security Posture
- Secure Score — percentage of healthy controls
- Security recommendations — grouped under security controls
- Inventory — view all resources and their security state
- Compliance dashboard — map recommendations to frameworks (CIS, NIST, PCI DSS)
- Custom standards — add your own compliance framework
- Microsoft Cloud Security Benchmark (MCSB) — default standard

## Topic 4: Defender for Cloud — Multi-Cloud & Hybrid
- Connect AWS — via AWS connector, agentless or agent-based
- Connect GCP — via GCP connector
- Connect on-premises — Azure Arc + Defender for Servers
- Defender EASM — External Attack Surface Management, discover internet-facing assets

## Topic 5: Defender for Cloud — Workload Protection Plans
- Defender for Servers — Plan 1 vs Plan 2 features
- Defender for Databases — SQL, Cosmos DB, open-source DBs
- Defender for Storage — malware scanning, sensitive data discovery
- Defender for Containers — AKS, ACR image scanning
- Defender for App Service, Key Vault, Resource Manager, DNS
- Plans are per resource type — can enable selectively
- Agentless scanning — no agent on VM, uses disk snapshot

## Topic 6: Defender for Cloud — Threat Protection & Alerts
- Security alerts — how generated, severity levels
- Alert suppression rules — reduce noise
- Workflow automation — trigger Logic App on alert or recommendation
- Email notifications — configure alert notifications
- Defender for Cloud DevOps Security — connect GitHub, Azure DevOps, GitLab
- DevOps security recommendations — exposed secrets, IaC misconfigurations

## Topic 7: Microsoft Sentinel — Data Collection
- Workspace — Sentinel sits on top of Log Analytics workspace
- Data connectors — Microsoft, partner, custom (CEF, Syslog, REST API)
- Data Collection Rules (DCRs) — Azure Monitor, control what data is collected
- Tables — SecurityEvent, SigninLogs, AzureActivity, CommonSecurityLog

## Topic 8: Microsoft Sentinel — Analytics Rules
- Scheduled — KQL query runs on schedule, generates alerts
- NRT (Near Real-Time) — runs every minute, low latency
- Microsoft Security — import alerts from Defender products
- Fusion — ML-based, correlates signals into multi-stage attack incidents
- Anomaly — ML-based behavioral analytics
- Threat Intelligence — match IOCs against logs
- Alert vs Incident — alerts grouped into incidents

## Topic 9: KQL for Sentinel
- Core operators — where, project, extend, summarize, join, render
- Time functions — ago(), now(), between()
- String operators — contains, startswith, has, matches regex
- Aggregations — count(), sum(), avg(), max(), min()
- Common exam queries — failed sign-ins, rare locations, high-volume events

## Topic 10: Microsoft Sentinel — Automation
- Playbooks — Logic Apps triggered by analytics rules or incidents
- Automation rules — first layer, run before playbooks, can assign/close incidents
- Automation rule vs Playbook — know which does what
- SOAR use cases — enrich incident, notify teams, block IP, disable user
- Managed identity for playbook authentication to Azure resources
