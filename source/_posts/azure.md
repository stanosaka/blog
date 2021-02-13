---
title: azure
date: 2020-06-14 19:38:55
tags: azure
---
# Azure architecturue foundations
> subscriptions, management, governance
## Resource organization
- Management group for each environment (Prod, Dev, and Test)
- Subscription for each "application categorization"
- Separate resource group for each application
- Consistent nomenclature at each level of this hierarchy
![resource organization](https://i.imgur.com/jpBakdZ.png "resource organization") 
## Naming Standards
- Azure resource names are important
- it's difficult to change names later
- Names must meet requirements specific to their resource type
[naming and tagging](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging)
## Organize resources by Tag
- Allows you to add metadata to Azure resources
- Allows you to query resources using the same tags
- Not all resource types support tags
## Azure policy and RBAC
- Policies enforce rules against your resources
- Role-based access control (RBAC) controls user actions
## Resource locks
- Prevent accidental resource changes
## Monitorinig and alerting
![monitoring and alerting](https://i.imgur.com/5mS3fbB.png) 

# Cloud application architecture
[Azure architecture center](https://docs.microsoft.com/en-us/azure/architecture/ "Azure architecture center")

# N-tier application architecture
![n-tier applicatioin architecuture](https://i.imgur.com/ziZzQrN.png)

# Hybrid cloud architecture
![hybrid cloud architecture](https://i.imgur.com/VxYsaXo.png)

Azure bastion

