# AZ-104 Azure Administrator Training Log

## Core Resources Used
* **Official Courseware:** [Microsoft Learn AZ-104 Practice Paths](https://learn.microsoft.com/en-us/credentials/certifications/azure-administrator/?practice-assessment-type=certification)
* **Lab Instructions:** [Microsoft Learning AZ-104 GitHub Labs](https://github.com/MicrosoftLearning/AZ-104-MicrosoftAzureAdministrator)
* **Lab Links:** [AZ-104 labs link](https://microsoftlearning.github.io/AZ-104-MicrosoftAzureAdministrator/)
* **Videos:** [John Savill's AZ-104 Exam Cram on YouTube](https://youtube.com)
* **Linkedin Videos:** [Prepare for Microsoft Cert Exam AZ-104: Microsoft Azure Administrator](https://www.linkedin.com/events/prepareformicrosoftcertexamaz-17464372536965787649/theater/)
* **Great ARM Template Walkthrough:** [Adam Marczak - Azure for Everyone](https://www.youtube.com/@AdamMarczakYT)
* **Used Labs here until they were no longer available[LevelUp Certification Prep](https://skillupwithlevelup.com/certification-prep-on-demand)

## Progress Tracker
- [x] Module 01: Manage Microsoft Entra ID Identities
- [ ] Module 02: Manage Governance and Compliance
- [ ] Module 03: Manage Azure Resources and Tools

AZ-104 Azure Administrator Micro-Labs
This repository contains a collection of high-impact, low-cost "Micro-Labs" engineered to target tricky architectural boundaries, configuration constraints, and common exam traps found in the Microsoft AZ-104 certification.

🛠️ Sandbox Cost Optimization Rule
To keep total consumption costs under $2.00, delete all resources immediately after completing each verification step. You can run the following Azure CLI command in the Cloud Shell (Bash) to recursively purge all lab resource groups:

Bash
az group list --query "[?contains(name, 'rg-labs')].name" -o tsv | xargs -I {} az group delete --name {} --yes --no-wait
🧪 Lab 1: The Cross-Region Logging & Storage Matrix
Objective: Verify cross-region capabilities for Log Analytics vs. Event Hub / Storage destinations.

Exam Trap: Microsoft tests whether you remember that Log Analytics workspaces support cross-region collection, while Event Hubs and Storage Accounts must match the target resource's region exactly.

🏁 Step-by-Step Instructions
Deploy the Logging Hub (East US):

Create a Resource Group named rg-labs-east in East US.

Deploy a Log Analytics Workspace inside rg-labs-east.

Deploy an Event Hub Namespace inside rg-labs-east.

Deploy the Target Resource (West US):

Create a Resource Group named rg-labs-west in West US.

Deploy a standard, cheap storage account (LRS) or a small B-series Virtual Machine inside rg-labs-west.

Configure Diagnostic Settings:

Navigate to the resource inside rg-labs-west.

Scroll down the left-hand blade to Monitoring -> Diagnostic Settings and click Add diagnostic setting.

Test the Architectural Boundaries:

Check the box for Send to Log Analytics workspace. Notice that you can successfully select the Workspace located in rg-labs-east.

Check the box for Stream to an event hub. Notice if the dropdown restricts you or requires a localized endpoint match.

Teardown: Run the cleanup script or delete both resource groups manually.

🧪 Lab 2: Persistent Shared Storage & SQL Backup Architectures
Objective: Mount a persistent, multi-instance file volume and eliminate the "Azure Tables" exam distractor.

Exam Trap: Real database or virtual machine backup files (.bak) require traditional file structures or object stores. They can never be stored inside structured NoSQL key-attribute stores like Azure Tables.

🏁 Step-by-Step Instructions
Deploy Storage Infrastructure:

Create a Resource Group named rg-labs-storage.

Create a standard general-purpose v2 Storage Account (stbackuplab104).

Provision the File Share:

Open the storage account blade, navigate to Data storage -> File shares.

Create a new file share named sql-backup-share (Set a strict quota of 1 GiB to control costs).

Analyze the Multi-Instance Connection Logic:

Click into sql-backup-share and click Connect.

Toggle between the Windows, Linux, and macOS operating system tabs.

Review the generated script block. Note how Azure utilizes Port 445 (SMB) or NFS to let multiple distinct VMs or external database instances mount the exact same file volume natively.

Teardown: Delete rg-labs-storage.

🧪 Lab 3: Private DNS Zones & Virtual Network Links
Objective: Configure custom domain resolution across isolated private networks.

Exam Trap: Peering two VNets allows network routing, but it does not automatically enable name resolution. You must explicitly attach a Virtual Network Link to each VNet for Private DNS records to resolve.

🏁 Step-by-Step Instructions
Deploy Network Core:

Create a Resource Group named rg-labs-networking.

Deploy VNet-Alpha (Address space: 10.1.0.0/16).

Deploy VNet-Beta (Address space: 10.2.0.0/16).

Provision the Private DNS Zone:

Search for Private DNS zones in the top global search bar.

Create a zone named infra.local and assign it to rg-labs-networking.

Configure the Infrastructure Links:

Inside the infra.local blade, navigate to Settings -> Virtual network links.

Click Add to create a link to VNet-Alpha. Check the box marked Enable auto-registration.

Leave VNet-Beta completely unlinked.

Evaluate Exam Logic:

Any VM deployed to VNet-Alpha will automatically register its hostname (e.g., web01.infra.local).

Critical Filter: A VM inside VNet-Beta will fail to resolve web01.infra.local entirely—even if global VNet peering is active—because it lacks a matching Virtual Network Link.

Teardown: Delete rg-labs-networking.

🧪 Lab 4: App Service SKUs, Deployment Slots, and Sticky Settings
Objective: Map feature availability restrictions to App Service Pricing Tiers and implement slot-specific variables.

Exam Trap: Deployment slots require a Standard, Premium, or Isolated App Service Plan. App settings do not persist through code swaps unless they are explicitly flagged as a slot-specific setting.

🏁 Step-by-Step Instructions
Deploy App Infrastructure:

Create a Resource Group named rg-labs-apps.

Create a Web App. Under the App Service Plan configuration, deliberately scale the SKU down to the F1 (Free) or B1 (Basic) tier.

Observe SKU Restrictions:

Once deployed, open the Web App and look at the left-hand menu for Deployment slots.

Note the interface restriction or upgrade block warning out the option.

Scale the Plan Up:

Navigate to Automation / Settings -> Scale up (App Service plan).

Move the tier up to a Production-grade S1 (Standard) SKU and apply the change.

Configure Slots and Sticky Settings:

Return to Deployment slots (now unlocked) and add a slot named staging.

Navigate to Settings -> Configuration on the main production slot.

Add a new Application Setting: Name = DbConnectionString, Value = Production_Database_Prod.

The Pivot Point: Check the box next to the configuration value labeled "Deployment slot setting".

Exam Logic: Checking this box makes the variable Sticky. During a deployment swap, this connection string remains permanently bound to the slot environment instead of moving with the application source code.

Teardown: Delete rg-labs-apps.
