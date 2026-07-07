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

# AZ-104 Azure Administrator Micro-Labs

This repository contains a collection of high-impact, low-cost "Micro-Labs" engineered to target tricky architectural boundaries, configuration constraints, and common exam traps found in the Microsoft AZ-104 certification.

## 🛠️ Sandbox Cost Optimization Rule
To keep total consumption costs under $2.00, **delete all resources immediately** after completing each verification step. You can run the following Azure CLI command in the Cloud Shell (Bash) to recursively purge all lab resource groups:

```bash
az group list --query "[?contains(name, 'rg-labs')].name" -o tsv | xargs -I {} az group delete --name {} --yes --no-wait
