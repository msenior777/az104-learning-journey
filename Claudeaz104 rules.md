# AZ-104 Exam Rules — Things That Caught Me Out

> This document is a living list of rules, traps, and "I didn't know that" moments built from real exam questions across multiple attempts. Every rule here came from a question that cost marks. Review this the night before and the morning of the exam.

---

## 🗄️ Storage

### Lifecycle Management
- **Multiple lifecycle rules apply to the same blob → the lowest cost action wins**
  - Delete beats Archive beats Cool beats Hot
  - Even if Rule1 says Cool and Rule3 says Archive, if Rule2 says Delete — the blob is deleted
- **Lifecycle rules run once per day**
- **Lifecycle tiering applies to Block Blobs only** — not Page Blobs or Append Blobs
- **Lifecycle can delete Append Blobs but cannot tier them** to Cool or Archive
- **`daysAfterModificationGreaterThan`** = use when question says "not updated" or "not modified"
- **`daysAfterLastAccessTimeGreaterThan`** = use when question says "not accessed" or "not read"
- **`daysAfterCreationGreaterThan`** = use when question says "since it was created"

### Access Tiers
- **Archive blobs are offline** — must rehydrate before reading
- **Standard rehydration = up to 15 hours**
- **High Priority rehydration = under 1 hour** for blobs under 10GB
- **Minimum storage durations:** Cool = 30 days, Cold = 90 days, Archive = 180 days
- Deleting a blob before its minimum duration incurs early deletion charges

### Redundancy
- **LRS** = 3 copies, 1 datacenter, no zone or region protection
- **ZRS** = 3 copies across 3 availability zones, 1 region
- **GRS** = LRS primary + LRS secondary region. Secondary is **read-only on failover only**
- **RA-GRS** = GRS + read access to secondary region at all times
- **GZRS** = ZRS primary + LRS secondary region
- **RA-GZRS** = GZRS + read access to secondary region at all times
- **RA prefix = read access to secondary.** Without RA you cannot read from secondary during normal operation
- Region failure protection → need GRS or GZRS
- Zone failure protection → need ZRS or GZRS
- Read access during region outage → need RA-GRS or RA-GZRS

### Encryption
- **Customer-managed keys stored in Key Vault = RSA key type**
- **Maximum supported RSA bit length = 4096** (options are 2048, 3072, 4096)
- AES is used internally by Azure to encrypt data at rest — you do not store AES keys in Key Vault
- **Encryption type (MMK → CMK) CAN be changed after storage account creation**
- **Infrastructure encryption CANNOT be changed after creation** — must be set at creation time

### Storage Account Settings — What Can and Cannot Be Changed After Creation
| Setting | Changeable After Creation? |
|---|---|
| Encryption type (MMK → CMK) | ✅ Yes |
| Replication type | ✅ Yes |
| Access tier (Hot/Cool) | ✅ Yes |
| Networking settings | ✅ Yes |
| Soft delete settings | ✅ Yes |
| Performance tier (Standard/Premium) | ❌ No — locked at creation |
| Premium account type | ❌ No — locked at creation |
| Hierarchical namespace (ADLS Gen2) | ❌ No — locked at creation |
| Infrastructure encryption | ❌ No — locked at creation |
| Location | ❌ No — locked at creation |

### Hierarchical Namespace (ADLS Gen2)
- **Cannot be enabled after storage account creation**
- Must create a new storage account to enable it

### SAS Tokens
- **SAS Allowed Services overrides everything** — if Blob is unchecked, no blob access regardless of RBAC
- **Ad-hoc SAS token compromised → rotate the storage account key** to revoke (invalidates all SAS tokens signed with that key)
- **Stored Access Policy backed SAS → delete or modify the policy** to revoke
- User Delegation SAS = backed by Entra ID credentials, most secure option
- "Grant temporary access" = SAS token

### AzCopy
- **`azcopy copy`** = always transfers all files regardless of whether they exist at destination
- **`azcopy sync`** = only transfers files that are new or different at destination (like robocopy /MIR)
- AzCopy requires authentication — SAS token in URL or `azcopy login` for Entra ID auth

### Azure Files vs Azure Tables
- **Azure Files** = SMB/NFS file share. Used for SQL backup files and persistent shared storage
- **Azure Tables** = NoSQL key-value store. NOT for file storage or backups
- SQL backup question → the answer is Azure Files (or Blob Storage for backup-to-URL), never Azure Tables

### Import/Export Service
- Use when question mentions "100TB of data", "limited bandwidth", or "physical drives"
- Import = ship drives to Azure, they upload
- Export = Azure copies to drives, ships to you

### Default Routing Tier
- **Microsoft network routing** = premium, lower latency, higher cost
- **Internet routing** = cheaper, uses public internet
- "Minimize network costs" → Internet routing

---

## 🌐 Networking

### DNS — Private Zones
- **Private DNS zones can only be linked to VNets** — public zones cannot be linked
- **Auto-registration = private zones only**, never public zones
- **A VNet can only have one private zone with auto-registration enabled**
- A VM can only resolve a private DNS zone if its VNet is **linked** to that zone
- VNet linked to zone = can resolve records in it. No link = cannot resolve

### DNS — Custom DNS Servers
- **Custom DNS server configured on VNet = Azure Private DNS zones are bypassed**
- To use private DNS zones with a custom DNS server → configure a **forwarder to 168.63.129.16** (Azure's recursive resolver)
- Without the forwarder, private zone records are invisible to anything using that custom DNS server
- This is the most common DNS trap on the exam

### DNS Decision Tree
When you see a DNS resolution question, ask in order:
1. What DNS server is the VNet using? (Azure-provided or Custom?)
2. If Azure-provided → is the VNet linked to the private zone? If yes → private zone record wins
3. If Custom DNS → goes to that server first, private zone bypassed (unless forwarder to 168.63.129.16 is configured)

### DNS Record Types
- **A record** = hostname → IP address (forward lookup) ✅ use for ping
- **PTR record** = IP → hostname (reverse lookup only) ❌ cannot use PTR to ping by name
- **CNAME** = alias pointing to another hostname. Chain breaks if the target is a TXT record
- **TXT record** = arbitrary text. Never used for name resolution. Cannot ping using a TXT record
- **MX record** = mail exchanger

### Public DNS Zones
- Azure hosts the zone but the **registrar must point name servers to Azure DNS**
- If VM can resolve internet but not a public Azure DNS zone → configure name servers at the domain registrar
- Azure name servers: ns1-xx.azure-dns.com format

### VNet Peering
- VNet peering is **non-transitive** — VNET1 peered with VNET2, VNET2 peered with VNET3 does NOT mean VNET1 can talk to VNET3
- Peering must be configured in both directions

### NSG Rules
- **Lower number = higher priority** (100 beats 200)
- Rules are processed in priority order, first match wins
- Default rules cannot be deleted but can be overridden with lower numbers

### AKS Networking
- **Azure CNI** = each pod gets a real VNet IP, routable from on-premises
- **Kubenet** = pods get internal overlay IPs, NOT routable from outside the cluster
- "On-premises clients connect to pod IP directly" → Azure CNI

---

## 🔐 Identity & Governance (Entra ID)

### Users
- **Usage Location must be set before assigning a license** — portal silently blocks it
- Authentication Administrator = can reset passwords for non-admin users
- Password Administrator = more limited scope than Authentication Administrator

### Groups
- **Dynamic groups require Entra ID Premium P1**
- **Dynamic group membership updates can take up to 24 hours** — not instant
- Assigned groups = manual membership

### RBAC
- **RBAC is additive** — higher scope permission wins
- **No implicit deny** in standard RBAC — deny only comes from explicit Deny Assignments
- **Deny Assignments override everything** including Owner
- Owner = full control including access management
- Contributor = full control of resources, cannot manage access
- Reader = view only
- User Access Administrator = manage access only, cannot touch resources

### Azure Policy
- **Policy does NOT retroactively delete non-compliant resources** — existing resources are flagged as Non-Compliant only
- New deployments that violate the policy are blocked
- Policy assignment can take up to **30 minutes** to take effect
- `DeployIfNotExists` and `Modify` effects require a **Managed Identity** to perform remediation
- Policy effects: Deny → Audit → Modify → DeployIfNotExists → Append

### PIM (Privileged Identity Management)
- **Requires Entra ID Premium P2**
- Just-in-time access — activates roles on demand

### B2B vs B2C
- **B2B** = Guest users from external organisations via Invitation
- **B2C** = Customer identity management, separate tenant entirely

---

## 📊 Monitoring

### Service Overview
- **Azure Monitor** = umbrella service, collects everything
- **Log Analytics Workspace** = where you query logs with KQL
- **Diagnostic Settings** = configures where logs and metrics are sent
- **Application Insights** = application-level APM, part of Azure Monitor
- **Azure Advisor** = free recommendations, not real-time, updates periodically

### Diagnostic Settings Destinations
| Destination | Use Case |
|---|---|
| Log Analytics Workspace | Querying and analysis |
| Storage Account | Long-term cheap archiving |
| Event Hub | Streaming to third-party SIEM (Splunk, Sentinel) |
| Partner Solution | Integrated monitoring partners |

- **Storage Account for diagnostics must be in the same region as the resource**
- **Diagnostic Settings are not retroactive** — logs only flow from the moment configured

### KQL
| Operator | What It Does | SQL Equivalent |
|---|---|---|
| `where` | Filter rows | WHERE |
| `summarize` | Aggregate / group | GROUP BY |
| `project` | Select columns | SELECT |
| `order by` | Sort results | ORDER BY |
| `ago(1d)` | Time filter shorthand | — |

### Alerts
- **Alerts = Condition + Action Group**
- **Action Groups are reusable** — one Action Group can attach to many Alert rules
- Deleting an Alert rule does NOT delete the Action Group
- Activity Log signal = who did what and when (control plane actions like Delete)
- Metrics signal = numerical performance data (CPU %, memory etc.)

### Azure Advisor
- **Free and automatic** — no configuration needed
- 5 pillars: Cost, Security, Reliability, Operational Excellence, Performance
- Not real-time — updates periodically (typically daily)
- "Get recommendations to reduce cost or improve security" → Azure Advisor

---

## 💻 Compute

### App Services
- **Deployment slots require Standard tier or above** — not available on Free or Shared
- Slot swap = swaps the VIP (Virtual IP) between slots
- Auto-swap, custom domains, SSL = Standard tier minimum

### ARM Templates
- **`New-AzResourceGroupDeployment -Mode Incremental`** = safe default, only adds/updates resources
- **Complete mode = deletes resources in the resource group NOT in the template** — dangerous
- `platformFaultDomainCount` = how many fault domains for availability set
- `platformUpdateDomainCount` = how many update domains for availability set

### VM Availability
- **Availability Set** = protects against hardware failures within a datacenter
- **Availability Zone** = protects against entire datacenter failure
- **Scale Set** = auto-scaling group of identical VMs

---

## 🔑 General Exam Technique

- **Read every question twice** before answering
- **"Not updated"** in a storage question = modification, not access
- **"Minimize cost"** = look for the cheapest redundancy or routing option
- **"After creation"** = think about what settings are locked vs changeable
- **Custom DNS on VNet** = private zones bypassed, always check for 168.63.129.16 forwarder
- **PTR records** = reverse lookup only, cannot use to ping
- **Multiple rules on same blob** = lowest cost action wins, Delete always beats tier changes
- **Only change an exam answer if you have a specific concrete reason** — not just doubt
- **First instinct is usually right** on questions where you have some knowledge
