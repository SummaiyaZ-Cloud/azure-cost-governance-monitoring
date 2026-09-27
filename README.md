# Azure Cost Governance & Automated Monitoring

## Project Overview

This project demonstrates the implementation of Azure cost governance, automated budget monitoring, resource monitoring, and cost-optimization controls in a simulated enterprise Azure environment.

The solution uses Azure Cost Management, Azure Monitor Action Groups, Log Analytics, Azure Resource Graph, diagnostic settings, and Azure Workbooks to provide visibility into cloud spending and identify potentially unnecessary resources.

## Architecture

Azure Subscription  
↓  
Azure Cost Management Budget  
↓  
50% / 80% / 100% Cost Thresholds  
↓  
Azure Monitor Action Group  
↓  
Email Notification

Azure Subscription Activity Logs  
↓  
Diagnostic Settings  
↓  
Log Analytics Workspace

Azure Resource Graph  
↓  
Resource Inventory & Orphaned Resource Queries  
↓  
Azure Monitoring Workbook

## Technologies Used

- Microsoft Azure
- Azure Cost Management + Billing
- Azure Monitor
- Azure Action Groups
- Azure Log Analytics
- Kusto Query Language (KQL)
- Azure Resource Graph
- Azure Diagnostic Settings
- Azure Workbooks
- Azure Policy
- GitHub

## 1. Cost Budget & Threshold Alerts

A monthly Azure budget was configured at the subscription scope to provide proactive cost monitoring.

**Budget configuration**

| Setting | Configuration |
|---|---|
| Budget | $20/month |
| Scope | Azure subscription |
| Reset period | Monthly |
| 50% threshold | $10 |
| 80% threshold | $16 |
| 100% threshold | $20 |

Each threshold is associated with the `ag-cost-alerts` Azure Monitor Action Group.

This creates an automated notification path as spending approaches the defined monthly limit.

![Budget and Action Group Alerts](screenshots/06-budget-action-group-alerts.png)

## 2. Azure Monitor Action Group

An Azure Monitor Action Group was created to centralize alert notification handling.

**Configuration**

- Action Group: `ag-cost-alerts`
- Short name: `CostAlerts`
- Resource Group: `rg-cost-monitoring-lab4`
- Region: Global
- Notification method: Email

The Action Group can be reused by monitoring and alerting rules instead of configuring recipients separately for every alert.

## 3. Log Analytics Workspace

A dedicated Log Analytics workspace was deployed for centralized monitoring.

**Workspace**

`law-cost-monitoring-lab4`

The workspace provides a centralized location for Azure monitoring and diagnostic telemetry.

## 4. Diagnostic Settings

Subscription Activity Logs were configured through Azure diagnostic settings and routed to the Log Analytics workspace.

The collected categories include administrative, security, service health, alerts, recommendations, policy, autoscale, and resource health events.

This establishes the telemetry pipeline:

`Azure Subscription → Activity Logs → Diagnostic Settings → Log Analytics`

## 5. Resource Inventory

Azure Resource Graph was used to query the subscription inventory and summarize deployed resources by type.

```kusto
Resources
| summarize ResourceCount=count() by type
| order by ResourceCount desc
```

This provides a fast subscription-level view of the Azure resource footprint.

## 6. Unattached Managed Disk Detection

The following query identifies managed disks that are not attached to a resource:

```kusto
Resources
| where type =~ 'microsoft.compute/disks'
| where isempty(managedBy)
| project name, resourceGroup, location, diskState=properties.diskState
```

An unattached managed disk can continue generating storage charges even though it is no longer associated with a VM. Detecting these resources provides an opportunity for cost optimization after validating that the disk is no longer required.

## 7. Unassigned Public IP Detection

The following query identifies Public IP resources without an associated IP configuration:

```kusto
Resources
| where type =~ 'microsoft.network/publicipaddresses'
| where isempty(properties.ipConfiguration)
| project name, resourceGroup, location, ipAddress=properties.ipAddress
```

At the time of testing, the query returned no matching unused Public IP resources.

## 8. Cost Governance & Monitoring Workbook

An Azure Workbook was created to provide a centralized visualization layer for resource governance and monitoring.

The workbook contains:

- Azure resource inventory
- Resource counts by type
- Unattached managed disk detection
- Unassigned Public IP detection
- Cost-governance monitoring context

![Azure Cost Governance Dashboard](screenshots/07-cost-monitoring-dashboard.png)

## Governance Controls

This lab also operated within an Azure Policy-controlled environment.

Resources were required to contain the following governance tag:

`CostCenter = CloudLab`

This demonstrated how organizational Azure Policy controls can enforce resource metadata standards across deployments.

## Troubleshooting

### Resource blocked by Azure Policy

Initial deployment of the Log Analytics workspace was rejected with:

`RequestDisallowedByPolicy`

The existing policy required the `CostCenter` tag.

The deployment was corrected by applying:

`CostCenter = CloudLab`

This allowed the workspace deployment to complete while remaining compliant with the governance policy.

### Action Group deployment failure

Initial Action Group creation failed because the subscription was not registered for the `Microsoft.Insights` resource provider.

The resource provider was registered at the subscription level, after which the Action Group was successfully deployed.

### Log Analytics returned no data

The new Log Analytics workspace initially returned no query results because telemetry had not yet been ingested.

Subscription Activity Logs were subsequently connected to the workspace through Azure diagnostic settings.

### Orphan-resource queries returned no matches

Resource Graph queries for unattached managed disks and unassigned Public IPs returned no matching resources during testing. This indicates that no resources matching those cleanup conditions existed at the time the queries were executed.

## Security & Governance Considerations

- Cost alerts provide proactive visibility before budget limits are exceeded.
- Action Groups centralize operational notifications.
- Diagnostic settings centralize subscription telemetry.
- Azure Policy enforces standardized resource tagging.
- Resource Graph queries help detect potentially unnecessary resources.
- Workbooks provide centralized operational visibility.

## Skills demonstrated

This project strengthened my understanding of:

- Azure cost governance
- Subscription-level budgets
- Automated cost alerts
- Azure Monitor Action Groups
- Log Analytics workspaces
- Diagnostic settings
- KQL and Azure Resource Graph
- Orphan-resource detection
- Azure Policy enforcement
- Azure Workbooks
- Cloud cost optimization

## Project Outcome

The completed environment demonstrates a practical cloud-governance workflow combining:

**Cost Control + Automated Alerting + Centralized Logging + Resource Optimization + Policy Governance + Monitoring Visualization**
