# AZ-104 Labs — Identities, Monitoring & Storage

> **Why these three?** Based on performance data across three exam attempts, these sections are the highest-priority weak spots. Identities & Governance alone is worth 20–25% of the exam. Monitoring collapsed to near-zero in attempt 2. Storage has been consistently underperforming across all attempts.

---

## Lab 1 — Manage Azure Identities and Governance

**Exam weight:** 20–25% &nbsp;|&nbsp; **Estimated time:** 45–60 mins &nbsp;|&nbsp; **Azure Free Tier:** ✅ Compatible

### Overview

This lab covers creating and managing Azure AD users and groups, assigning RBAC at different scopes, and enforcing governance with Azure Policy. These are the highest-weight topics on the exam — small gains here have the biggest score impact.

---

### Task 1 — Create and Manage Azure AD Users

Navigate to **Azure Active Directory → Users → New User** and create a user with the following settings:

| Field | Value |
|---|---|
| Display name | `Lab Test User` |
| Username | `labtestuser@<yourdomain>.onmicrosoft.com` |
| Usage location | Australia |
| Password | Auto-generate — copy it |

Once created, navigate to the user → **Assigned Roles → Add assignment** → select **Reports Reader**.

> 💡 **Exam Tip:** Usage Location must be set **before** you can assign a license. The portal will silently block license assignment if it's missing — this trips people up constantly.

> ⚠️ **Watch Out:** The portal won't always show a clear error message when license assignment fails due to missing Usage Location. If license assignment seems broken, check this first.

**Knowledge Check**

> **Q: A user cannot be assigned a Microsoft 365 license. What is the most likely cause?**
> A: Usage Location has not been set on the user account.

> **Q: What role allows a user to reset passwords for non-admin users only?**
> A: Authentication Administrator (can reset for non-admins). Password Administrator has a more limited scope.

---

### Task 2 — Create Groups and Dynamic Membership

Navigate to **Azure Active Directory → Groups → New Group** and create two groups:

**Group 1 — Assigned (Static)**

| Field | Value |
|---|---|
| Group type | Security |
| Group name | `AZ104-Static-Group` |
| Membership type | Assigned |
| Members | Add your lab test user |

**Group 2 — Dynamic** *(requires AAD Premium P1 — read the rule syntax even if you can't apply it on Free)*

| Field | Value |
|---|---|
| Group type | Security |
| Group name | `AZ104-Dynamic-Group` |
| Membership type | Dynamic User |
| Dynamic rule | `(user.department -eq "IT")` |

> 💡 **Exam Tip:** Dynamic groups require **Azure AD Premium P1**. If a question mentions dynamic membership — think P1 license requirement.

> ⚠️ **Watch Out:** Dynamic group membership is **not instant**. It can take up to 24 hours to process. A common exam trap is asking what to do when a user hasn't appeared in a dynamic group 5 minutes after being set to the right department — the answer is simply wait.

**Knowledge Check**

> **Q: What license is required to use Dynamic group membership?**
> A: Azure AD Premium P1 (or P2).

> **Q: A dynamic group rule is `(user.jobTitle -eq "Developer")`. A user has been set to Developer but hasn't appeared in the group after 10 minutes. What do you do?**
> A: Wait — dynamic group membership updates can take up to 24 hours.

---

### Task 3 — Assign RBAC Roles at Different Scopes

Navigate to your **Resource Group → Access Control (IAM) → Add role assignment** and assign your lab test user the **Reader** role scoped to the Resource Group.

Open a private browser window, sign in as the lab test user, and verify:
- ✅ They can view resources
- ❌ They cannot create or delete anything

Return to the portal. Now assign the same user the **Contributor** role at the **Subscription** level.

Verify effective permissions: **IAM → Check Access → select the user**

> 💡 **Exam Tip:** RBAC is **additive**. The user now has Contributor access inside that Resource Group even though Reader is assigned at the RG scope — the higher-scope permission wins. There is no implicit deny in standard RBAC. Deny only comes from explicit Deny Assignments, which override everything including Owner.

**Key RBAC Roles to Know**

| Role | What It Can Do |
|---|---|
| Owner | Full control including access management |
| Contributor | Full control of resources — cannot manage access |
| Reader | View only |
| User Access Administrator | Manage access only — cannot touch resources |

**Knowledge Check**

> **Q: A user is assigned Contributor at the Subscription scope and Reader at the Resource Group scope. What can they do in that Resource Group?**
> A: Full Contributor access — the higher scope permission applies and RBAC is additive.

> **Q: Which built-in role allows managing access assignments without being able to create or modify resources?**
> A: User Access Administrator.

---

### Task 4 — Azure Policy

Navigate to **Policy → Definitions** and find the built-in policy **"Allowed locations"**.

Assign it to your Resource Group with the following setting:
- Allowed locations: **Australia East** only

Attempt to create any resource in **East US** — the deployment should be blocked.

Navigate to **Policy → Compliance** to see the compliance state of your resources.

> 💡 **Exam Tip:** Policy assignment can take up to **30 minutes** to take effect. Don't panic if a newly assigned policy doesn't immediately block deployments.

> ⚠️ **Watch Out:** Resources created **before** a policy was assigned will show as **Non-Compliant** but will **not** be automatically deleted. This is one of the most common exam traps — Policy flags and blocks, it doesn't clean up.

> ⚠️ **Watch Out:** Policies with `DeployIfNotExists` or `Modify` effects need a **Managed Identity** assigned to the policy to perform remediation. The exam tests this distinction.

**Policy Effects — Quick Reference**

| Effect | What It Does |
|---|---|
| Deny | Blocks the deployment |
| Audit | Allows the deployment but logs non-compliance |
| Modify | Adds or changes properties (e.g. tags) on resources |
| DeployIfNotExists | Deploys a related resource if it doesn't exist |
| Append | Adds fields to the resource during creation |

**Knowledge Check**

> **Q: A Policy is assigned to deny resources outside Australia East. An existing VM is in East US. What happens?**
> A: The VM is unaffected — Policy does not retroactively delete resources. It shows as Non-Compliant.

> **Q: What policy effect would you use to automatically add a missing tag to resources?**
> A: Modify (or DeployIfNotExists for more complex remediation scenarios).

---

### 📋 Lab 1 Exam Cheat Sheet

| Concept | What to Remember |
|---|---|
| Usage Location | Must be set before assigning a license |
| Dynamic Groups | Requires AAD P1 · Can take up to 24 hours to update |
| RBAC | Additive · No implicit deny · Higher scope wins |
| Owner vs Contributor | Owner can manage access · Contributor cannot |
| Azure Policy | Existing non-compliant resources are flagged, NOT deleted |
| Policy — Modify/DeployIfNotExists | Requires a Managed Identity for remediation |
| PIM | Just-in-time access · Requires AAD P2 |
| B2B | Guest users from external orgs via Invitation |
| B2C | Customer identity management · Separate tenant entirely |

---

## Lab 2 — Monitor and Maintain Azure Resources

**Exam weight:** 10–15% &nbsp;|&nbsp; **Estimated time:** 30–45 mins &nbsp;|&nbsp; **Azure Free Tier:** ✅ Compatible

### Overview

Monitoring was the single biggest collapse between exam attempts — dropping to near-zero from a previously passing score. This lab targets the exact services the exam tests: Azure Monitor, Log Analytics, KQL, Diagnostic Settings, Alerts, and Advisor.

**Key Services at a Glance**

| Service | What It Does |
|---|---|
| Azure Monitor | Umbrella service — collects metrics and logs from all Azure resources |
| Log Analytics Workspace | Stores and queries log data using KQL |
| Diagnostic Settings | Configures where logs and metrics are sent |
| Azure Alerts | Notifies or triggers actions when conditions are met |
| Azure Advisor | Personalised recommendations across cost, security, reliability, performance |
| Application Insights | Application-level monitoring (APM) — part of Azure Monitor |

---

### Task 1 — Create a Log Analytics Workspace

Navigate to **Log Analytics workspaces → Create** and configure:

| Field | Value |
|---|---|
| Resource Group | `AZ104-Lab-RG` |
| Name | `az104-law-01` |
| Region | Australia East |

Once deployed, navigate to the workspace → **Logs** and run this query to verify data is flowing:

```kql
AzureActivity
| where TimeGenerated > ago(1d)
| summarize count() by OperationNameValue
| order by count_ desc
```

> 💡 **Exam Tip:** Log Analytics Workspace is where everything feeds into. If a question asks where to send diagnostic logs for querying and analysis — the answer is Log Analytics Workspace.

---

### Task 2 — Configure Diagnostic Settings

Navigate to any existing resource (e.g. a Storage Account) → **Monitoring → Diagnostic Settings → Add diagnostic setting** and configure:

| Field | Value |
|---|---|
| Name | `diag-to-law` |
| Logs | Select all available categories |
| Metrics | AllMetrics |
| Destination | Send to Log Analytics workspace → `az104-law-01` |

> 💡 **Exam Tip:** Diagnostic Settings can send to four destinations. Know which one to use:
> - **Log Analytics Workspace** — querying and analysis
> - **Storage Account** — long-term cheap archiving
> - **Event Hub** — streaming to a third-party SIEM (Splunk, Sentinel, etc.)
> - **Partner Solution** — integrated monitoring partners

> ⚠️ **Watch Out:** Diagnostic Settings are **not retroactive**. Logs only flow from the moment you configure them. There is no way to backfill historical data.

**Knowledge Check**

> **Q: You need to retain Azure Activity Logs for 2 years for compliance. What should you configure?**
> A: Export via Diagnostic Settings to a Storage Account with a lifecycle management policy.

> **Q: A security team wants to stream Azure logs to their Splunk SIEM. Which destination should you use?**
> A: Event Hub — third-party SIEMs consume from Event Hub.

---

### Task 3 — Write KQL Queries

In your Log Analytics Workspace → **Logs**, practise the following:

**Filter rows** (equivalent to SQL WHERE):
```kql
AzureActivity
| where OperationNameValue contains "delete"
| project TimeGenerated, Caller, ResourceGroup
```

**Aggregate data** (equivalent to SQL GROUP BY):
```kql
Heartbeat
| where TimeGenerated > ago(1h)
| summarize count() by Computer
```

**KQL Operator Cheat Sheet**

| Operator | What It Does | SQL Equivalent |
|---|---|---|
| `where` | Filter rows | WHERE |
| `summarize` | Aggregate / group data | GROUP BY |
| `project` | Select specific columns | SELECT |
| `order by` | Sort results | ORDER BY |
| `ago(1d)` | Time filter shorthand | — |

> 💡 **Exam Tip:** The exam often gives you a scenario and asks which KQL operator to use. "Filter results" = `where`. "Count per group" = `summarize`. "Show only certain columns" = `project`.

---

### Task 4 — Create an Alert Rule

Navigate to **Azure Monitor → Alerts → Create → Alert rule** and configure:

**Scope:** Select your Resource Group

**Condition:**
- Signal type: **Activity Log**
- Signal: **Delete** (any resource delete operation)

**Action Group:** Create new

| Field | Value |
|---|---|
| Name | `ag-email-alert` |
| Action type | Email/SMS/Push/Voice |
| Email | Your email address |

**Alert rule details:**

| Field | Value |
|---|---|
| Severity | Sev 2 - Warning |
| Name | `resource-delete-alert` |

> 💡 **Exam Tip:** Action Groups are **reusable** — one Action Group can be attached to multiple Alert rules. If you need to notify the same team from multiple rules, create one Action Group and attach it to all of them.

> ⚠️ **Watch Out:** Deleting an Alert rule does **not** delete the Action Group. They are independent resources.

**Alert Severity Reference**

| Severity | Level |
|---|---|
| Sev 0 | Critical |
| Sev 1 | Error |
| Sev 2 | Warning |
| Sev 3 | Informational |
| Sev 4 | Verbose |

**Knowledge Check**

> **Q: You need to alert the operations team when any resource in a resource group is deleted. What signal type should you use?**
> A: Activity Log signal — specifically the "Delete" operation.

> **Q: Multiple alert rules need to notify the same on-call team. What is the most efficient approach?**
> A: Create one Action Group for the on-call team and attach it to all relevant alert rules.

---

### Task 5 — Azure Advisor

Search for **Advisor** in the portal and review recommendations across the five pillars:

| Pillar | What It Looks For |
|---|---|
| Cost | Underused resources, reserved instance opportunities |
| Security | Aligned with Microsoft Defender for Cloud recommendations |
| Reliability | Availability and resilience improvements |
| Operational Excellence | Monitoring gaps, best practice violations |
| Performance | Speed and throughput improvements |

> 💡 **Exam Tip:** Advisor is **free** and automatic — it reads your subscription and generates recommendations without any configuration. "Get recommendations to reduce cost or improve security" → Azure Advisor.

> ⚠️ **Watch Out:** Advisor recommendations are **not real-time**. They update periodically (typically daily). You can Postpone or Dismiss individual recommendations.

---

### 📋 Lab 2 Exam Cheat Sheet

| Concept | What to Remember |
|---|---|
| Azure Monitor | Umbrella — collects everything |
| Log Analytics Workspace | Query logs with KQL |
| Diagnostic Settings | 4 destinations: Law / Storage / Event Hub / Partner |
| Storage Account (diag) | Must be in same region as the resource |
| KQL `where` | Filter rows |
| KQL `summarize` | Aggregate / group |
| KQL `project` | Select columns |
| Alerts | Condition + Action Group · Action Groups are reusable |
| Activity Log | Who did what and when (control plane) |
| Metrics | Numerical real-time performance data |
| Application Insights | App-level APM inside Azure Monitor |
| Azure Advisor | Free · 5 pillars · Not real-time |

---

## Lab 3 — Implement and Manage Storage

**Exam weight:** 15–20% &nbsp;|&nbsp; **Estimated time:** 45–60 mins &nbsp;|&nbsp; **Azure Free Tier:** ✅ Compatible

### Overview

Storage has been a consistent weak spot across all three exam attempts. This lab covers storage account types, redundancy tiers, blob access tiers, lifecycle management, SAS tokens, and AzCopy — all high-frequency exam topics. It also addresses the Azure Tables vs Azure Files trap that came up in the last exam sitting.

---

### Task 1 — Create a Storage Account

Navigate to **Storage accounts → Create** and configure:

| Field | Value |
|---|---|
| Resource Group | `AZ104-Lab-RG` |
| Storage account name | `az104storage01[yourinitials]` (globally unique, lowercase only) |
| Region | Australia East |
| Performance | Standard |
| Redundancy | LRS |

On the **Advanced** tab, review (but do not enable for Free tier):
- **Hierarchical namespace** — enables Azure Data Lake Storage Gen2. **Must be set at creation — cannot be added later.**
- **Blob public access** — controls anonymous access to containers.

Once deployed, navigate to **Containers → New container**:

| Field | Value |
|---|---|
| Name | `lab-container` |
| Public access level | Private |

> ⚠️ **Watch Out:** Hierarchical Namespace (ADLS Gen2) **cannot** be enabled after account creation. If a question asks how to enable it on an existing storage account — the answer is you must create a new one.

**Storage Account Types — Quick Reference**

| Type | Use Case |
|---|---|
| Standard GPv2 | Recommended for most scenarios — supports all storage services |
| Premium Block Blobs | High transaction, low latency blob workloads |
| Premium File Shares | SMB/NFS shares needing low latency |
| Premium Page Blobs | Unmanaged VM disks |

> 💡 **Exam Tip:** GPv2 is almost always the correct answer unless the question specifically calls out "premium performance" or "low latency" requirements.

---

### Task 2 — Blob Access Tiers and Lifecycle Management

Upload any small test file to `lab-container`, click into it, and select **Change tier** to explore the available options.

**Blob Access Tiers**

| Tier | Min Storage Duration | Access Cost | Storage Cost |
|---|---|---|---|
| Hot | None | Low | High |
| Cool | 30 days | Medium | Medium |
| Cold | 90 days | Higher | Lower |
| Archive | 180 days | High (rehydration needed) | Lowest |

> ⚠️ **Watch Out:** **Archive blobs are offline.** You cannot read them directly — you must rehydrate to Hot or Cool first. Standard rehydration takes **up to 15 hours**. High Priority rehydration can be under 1 hour for blobs under 10GB. This is a common exam trick — the question implies urgency then lists Archive as a tempting option.

Navigate to **Data Management → Lifecycle Management → Add a rule** and configure:

| Condition | Action |
|---|---|
| Last modified > 30 days | Move to Cool |
| Last modified > 90 days | Move to Archive |
| Last modified > 365 days | Delete the blob |

> 💡 **Exam Tip:** Lifecycle management rules run **once per day** and apply to **block blobs only** — not page blobs or append blobs.

**Knowledge Check**

> **Q: A company wants blobs automatically moved to Archive after 90 days and deleted after 2 years. What feature do you use?**
> A: Blob Lifecycle Management policy.

> **Q: A blob in Archive tier needs to be accessed urgently. What are your options?**
> A: Rehydrate by changing the tier to Hot or Cool, or copy to a new blob in Hot/Cool. Standard rehydration takes up to 15 hours. High Priority can be under 1 hour for blobs under 10GB.

---

### Task 3 — Redundancy Options

No portal steps needed for this task — review and memorise the tiers:

| Redundancy | Copies | Scope | Secondary Read Access |
|---|---|---|---|
| LRS | 3 | Single datacenter | ❌ |
| ZRS | 3 | 3 availability zones, 1 region | ❌ |
| GRS | 6 | LRS primary + LRS in secondary region | ❌ |
| RA-GRS | 6 | Same as GRS | ✅ |
| GZRS | 6 | ZRS primary + LRS in secondary region | ❌ |
| RA-GZRS | 6 | Same as GZRS | ✅ |

> 💡 **Exam Tip:** The **RA** prefix (Read Access) is the key distinction. With GRS, the secondary region exists for failover but you **cannot read from it** unless it's RA-GRS. This is tested repeatedly.

**Knowledge Check**

> **Q: A storage account needs to survive a full region outage and allow read access to data during the outage. What redundancy option should you choose?**
> A: RA-GRS or RA-GZRS — only these options provide read access to the secondary region.

> **Q: A storage account needs zone-level resilience within a single region, at lowest cost. What redundancy option?**
> A: ZRS.

---

### Task 4 — Shared Access Signatures (SAS)

Navigate to **Security + Networking → Shared Access Signature** and configure:

| Field | Value |
|---|---|
| Allowed services | Blob |
| Allowed resource types | Container, Object |
| Allowed permissions | Read, List |
| Expiry | 1 hour from now |
| Protocol | HTTPS only |

Click **Generate SAS and connection string** → copy the Blob service SAS URL.

> 💡 **Exam Tip:** SAS tokens grant **time-limited, permission-scoped access** without sharing your account key. "Grant temporary read access to a specific container" = SAS token.

> ⚠️ **Watch Out:** If a SAS token is compromised and it was **not** backed by a Stored Access Policy — the only way to revoke it is to **rotate the storage account key**. This invalidates ALL SAS tokens signed with that key. This is a common exam scenario.

**SAS Types**

| Type | Backed By |
|---|---|
| Account SAS | Storage account key — access to account-level services |
| Service SAS | Storage account key — access to a specific resource |
| User Delegation SAS | Azure AD credentials — most secure option |

**Knowledge Check**

> **Q: A developer needs temporary read-only access to a blob container for 48 hours. What should you provide?**
> A: A SAS token with Read and List permissions, expiring in 48 hours.

> **Q: A SAS token has been accidentally exposed. How do you revoke it immediately?**
> A: If it was backed by a Stored Access Policy — delete or modify the policy. If it was an ad-hoc SAS — rotate the storage account key (this invalidates all SAS tokens signed with that key).

---

### Task 5 — AzCopy (Syntax Review)

No Azure cost required — review the command syntax and understand the `copy` vs `sync` distinction:

```bash
# copy — transfers all files regardless of whether they exist at destination
azcopy copy "C:\local\folder" "https://<account>.blob.core.windows.net/<container>?<SAS>" --recursive

# sync — only transfers files that are new or different at destination
azcopy sync "C:\local\folder" "https://<account>.blob.core.windows.net/<container>?<SAS>"

# blob-to-blob copy between storage accounts
azcopy copy "https://<source>.blob.core.windows.net/<container>?<SAS>" \
            "https://<dest>.blob.core.windows.net/<container>?<SAS>"
```

> 💡 **Exam Tip:** `copy` always transfers everything. `sync` only transfers what's new or different — think of it like `robocopy /MIR`. This distinction is tested directly.

> ⚠️ **Watch Out:** AzCopy requires authentication — either a SAS token appended to the URL, or `azcopy login` for Azure AD auth. It is not authenticated by default.

**Large-Scale Transfers — Import/Export Service**

For datasets where network transfer would take too long (TB/PB scale):

| Direction | What Happens |
|---|---|
| Import | You ship physical drives to Azure — they upload the data |
| Export | Azure copies data to drives and ships them to you |

> 💡 **Exam Tip:** If a question mentions "100TB of data", "limited bandwidth", or "physical drives" — the answer is Azure Import/Export or Azure Data Box.

---

### 📋 Lab 3 Exam Cheat Sheet

| Concept | What to Remember |
|---|---|
| LRS | 3 copies · 1 datacenter · No zone/region protection |
| ZRS | 3 copies · 3 availability zones · Same region |
| GRS | 2 regions · Secondary is read-only (failover only) |
| RA-GRS | GRS + read access to secondary region |
| GZRS | ZRS primary + GRS secondary · Best protection |
| Blob tiers | Hot → Cool (30d) → Cold (90d) → Archive (180d, offline) |
| Archive rehydration | Up to 15 hours · High Priority < 1 hour for small blobs |
| Lifecycle Management | Automates tier transitions and deletion · Block blobs only · Runs daily |
| Hierarchical Namespace | ADLS Gen2 · Must be set at creation · Cannot add later |
| SAS Token | Time-limited scoped access · Revoke via SAP or key rotation |
| AzCopy copy | Always copies all files |
| AzCopy sync | Only copies new or changed files |
| Azure Files | SMB/NFS file share · Used for SQL backup files and persistent shared storage |
| Azure Tables | NoSQL key-value store · NOT for file or backup storage |
| Import/Export | Offline data transfer via physical disks for TB/PB scale |
