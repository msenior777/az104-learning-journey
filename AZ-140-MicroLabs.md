# AZ-104 Azure Administrator Micro-Labs

This repository contains a collection of high-impact, low-cost "Micro-Labs" engineered to target tricky architectural boundaries, configuration constraints, and common exam traps found in the Microsoft AZ-104 certification.

## 🛠️ Sandbox Cost Optimization Rule
To keep total consumption costs under $2.00, **delete all resources immediately** after completing each verification step. You can run the following Azure CLI command in the Cloud Shell (Bash) to recursively purge all lab resource groups:

```bash
az group list --query "[?contains(name, 'rg-labs')].name" -o tsv | xargs -I {} az group delete --name {} --yes --no-wait
