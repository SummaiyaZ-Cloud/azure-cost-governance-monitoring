# Azure Cost Governance & Monitoring

## Project Overview

This project demonstrates the implementation of an Azure cost governance and monitoring solution using Azure Cost Management, Azure Monitor, Log Analytics, Azure Resource Graph, Action Groups, and Azure Workbooks.

The goal was to create a centralized solution for monitoring Azure spending, detecting potentially unused resources, configuring automated budget notifications, and visualizing the Azure environment.

---

## Architecture

The solution uses:

- Azure Cost Management for budget tracking
- Azure Monitor for centralized monitoring
- Azure Action Groups for automated email notifications
- Log Analytics Workspace for centralized log collection
- Azure Resource Graph for resource inventory and optimization queries
- Azure Workbooks for dashboard visualization
- Azure Policy/Tags for cost governance

---

## Implementation

### 1. Cost Budget

Created a monthly Azure budget:

**Budget:** `$20/month`

Alert thresholds:

| Threshold | Amount | Notification |
|---|---:|---|
| 50% | $10 | Action Group Email |
| 80% | $16 | Action Group Email |
| 100% | $20 | Action Group Email |

This provides proactive notification before cloud spending exceeds the configured budget.

### 2. Action Group

Created an Azure Monitor Action Group:

`ag-cost-alerts`

The Action Group is connected to the budget thresholds and configured to send email notifications when spending reaches the defined limits.

### 3. Log Analytics

Created a Log Analytics Workspace:

`law-cost-monitoring-lab4`

The workspace provides a centralized location for Azure monitoring and log data.

Subscription Activity Logs were configured to send supported log categories to the workspace.

### 4. Azure Resource Graph

Azure Resource Graph was used to query resources across the subscription.

Example resource inventory query:

```kusto
Resources
| summarize count() by type
| order by count_ desc
```

An additional query identifies unattached managed disks that could represent unnecessary cloud spending:

```kusto
Resources
| where type =~ 'microsoft.compute/disks'
| where isempty(managedBy)
| project name, resourceGroup, location, diskState = properties.diskState
```

The queries are stored in the [`queries`](./queries) directory.

### 5. Azure Workbook

Created an Azure Workbook named:

`Azure Cost Governance & Monitoring Dashboard`

The workbook provides a centralized visualization of Azure resources and governance information using Resource Graph queries.

---

## Cost Governance Strategy

The solution combines several layers of governance:

**Prevent:** Azure Policy and tagging standards help enforce resource governance.

**Monitor:** Azure Monitor, Log Analytics, Resource Graph, and Workbooks provide visibility into the environment.

**Alert:** Azure Cost Management budgets and Action Groups provide automated cost notifications.

**Optimize:** Resource Graph queries help identify resources that may be candidates for cleanup or cost optimization.

---

## Challenges & Troubleshooting

### Azure Provider Registration

During Action Group creation, Azure reported that the subscription was not registered for the `Microsoft.Insights` resource provider.

The provider was registered before recreating the Action Group.

### Azure Policy Enforcement

Resource creation was initially blocked by an existing `CostCenter` tagging policy.

Required tags were added to resources to satisfy the governance policy.

### Log Analytics Data

Initial Log Analytics queries returned no results because the workspace had not yet received relevant log data.

Subscription Activity Log diagnostic settings were configured to send supported categories to the Log Analytics workspace.

### Resource Optimization Query

The unattached disk query returned no results. This indicated that no managed disks currently matched the unused-disk criteria rather than indicating a query failure.

---

## What I Learned

Through this project I gained hands-on experience with:

- Azure Cost Management and budgets
- Automated cost alerting
- Azure Monitor Action Groups
- Log Analytics Workspaces
- Diagnostic settings
- Azure Resource Graph and KQL
- Azure Workbooks
- Azure Policy enforcement
- Resource tagging and cost governance
- Troubleshooting Azure resource-provider and policy errors

Most importantly, I learned how Azure governance, monitoring, and cost-management services can work together rather than treating each service as an isolated feature.

---

## Interview Talking Points

This project demonstrates my ability to design a basic Azure cost-governance workflow rather than simply deploy individual resources.

I can explain:

- How budgets and thresholds can help prevent unexpected Azure spending
- How Action Groups automate notifications
- How tagging supports cost allocation and governance
- How Azure Policy can enforce organizational standards
- How Resource Graph can identify resources across a subscription
- How KQL can be used to investigate potential optimization opportunities
- How Log Analytics centralizes operational data
- How Workbooks provide centralized monitoring views
- How I troubleshot provider-registration, policy, and data-availability issues

---

## Repository Structure

```text
azure-cost-governance-monitoring/
├── queries/
│   ├── resource-inventory.kql
│   └── unused-disks.kql
└── README.md
```

---

## Technologies

`Microsoft Azure` `Azure Cost Management` `Azure Monitor` `Log Analytics` `Azure Resource Graph` `KQL` `Azure Workbooks` `Azure Policy` `GitHub`

---

## Key Outcome

Built an Azure cost governance and monitoring solution that combines budget controls, automated notifications, centralized logging, resource discovery, optimization queries, policy-based governance, and dashboard visualization.
