## Resource Group Configuration

Before deploying any infrastructure components in Microsoft Azure, all resources are logically organized within a dedicated Resource Group. This approach enables centralized lifecycle management, streamlined governance, and simplified cost tracking.

A Resource Group in Azure acts as a logical container for related resources such as virtual machines, virtual networks, storage accounts, load balancers, and security components. Grouping resources together ensures they can be managed, monitored, updated, or deleted collectively when necessary.

---

### Resource Group Details

**Resource Group Name:**
`Webapp01`

**Region:**
East US

All infrastructure components for this deployment including virtual machines, networking resources (VNet and subnets), Application Gateway, network security groups, and associated storage—are deployed within this resource group.

This structure ensures:

* Centralized management of all application components
* Consistent regional deployment for reduced latency
* Simplified access control using Azure RBAC
* Easier cleanup and environment teardown when required

Organizing resources under a single, clearly defined resource group aligns with Azure governance best practices and supports production-style infrastructure management.
