## Configure Virtual Machine Scale Set (VMSS)

To enable the `webapp01` Virtual Machine to scale automatically based on workload demand, an **Azure Virtual Machine Scale Set (VMSS)** is implemented.

A VM Scale Set allows you to deploy and manage a group of identical, load-balanced virtual machines. It ensures:

* High availability
* Automatic horizontal scaling (scale-out and scale-in)
* Automatic replacement of unhealthy instances
* Even distribution of traffic via Application Gateway
* Cost efficiency during low utilization periods

Instead of relying on a single static VM, VMSS provides an elastic and resilient compute layer aligned with Azure cloud-native architecture principles.

To configure a VM Scale Set, the following components are required:

* A **custom VM image** (or source image)
* A **Scale Set configuration**
* Integration with the **Application Gateway backend pool**
* An **Autoscale policy**

---

## Create Virtual Machine Image

A VM image acts as the blueprint for all instances within the scale set. It contains:

* Operating system
* Installed software (Apache Tomcat)
* Application dependencies
* Configuration settings
* Deployed web application artifact

Creating a custom image ensures every scaled instance is consistent and preconfigured.

### Steps to Create Image

1. Ensure the existing `webapp01` VM is fully configured and tested.
2. Deallocate the VM:

   ```
   Virtual Machines → webapp01 → Stop
   ```
3. Navigate to:

   ```
   Virtual Machines → webapp01 → Capture
   ```
4. Configure:

   * **Image Name:** `webapp01-image`
   * **Resource Group:** Webapp01
   * Select **Automatically delete VM after image creation** (optional)

Once completed, the image will appear under:

```
Azure Portal → Images
```

Wait until the provisioning state shows **Succeeded** before proceeding.

This image will serve as the golden image for the scale set.

---

## Create Virtual Machine Scale Set

Navigate to:

```
Azure Portal → Virtual Machine Scale Sets → Create
```

### Basic Configuration

* **Name:** `webapp01-vmss`
* **Region:** Canada Central
* **Availability Zones:** Select 1, 2, and 3 (for high availability)
* **Orchestration Mode:** Uniform
* **Image:** Select `webapp01-image`
* **Instance Size:** B2s (lab) or D-series (production)

---

### Administrator and Authentication

* Use SSH key authentication (recommended)
* Avoid password-based authentication for security best practices

---

### Networking Configuration

* **Virtual Network:** Select existing VNet
* **Subnet:** Application subnet
* **Public IP:** Disabled (Application Gateway will handle public access)

This ensures the scale set instances are not directly exposed to the internet.

---

## Integrate with Application Gateway

Under **Load Balancing**:

* Select **Use an existing Application Gateway**
* Choose `webapp01-agw`
* Add the VMSS to the backend pool

This allows:

* Application Gateway to distribute traffic across scale set instances
* Health probes to monitor instance availability
* Automatic registration/deregistration of instances

---

## Configure Health Monitoring

VMSS integrates with:

* Azure Load Balancer or Application Gateway health probes
* VM instance health extensions

If an instance fails health checks, Azure automatically:

* Removes it from rotation
* Replaces it with a new instance

This ensures application continuity.

---

## Configure Scaling Settings

VM Scale Sets require capacity configuration:

* **Initial Instance Count:** 2
* **Minimum Instances:** 1
* **Maximum Instances:** 4

### Meaning of These Values

* **Minimum:** Guarantees at least one instance is always running
* **Initial:** Number of instances deployed at creation
* **Maximum:** Upper scaling boundary

---

## Configure Autoscale Rules

Navigate to:

```
VM Scale Set → Scaling → Custom Autoscale
```

Select:

* **Scale based on a metric**

### Scale-Out Rule

* **Metric:** Percentage CPU
* **Condition:** Greater than 50%
* **Duration:** 5 minutes
* **Action:** Increase instance count by 1

### Scale-In Rule

* **Metric:** Percentage CPU
* **Condition:** Less than 30%
* **Duration:** 10 minutes
* **Action:** Decrease instance count by 1

### Scaling Logic

* When CPU utilization exceeds 50%, new instances are automatically created.
* When CPU drops below 30%, instances are gradually removed to optimize cost.

Cooldown periods prevent rapid scaling fluctuations.

---

## Network Security Group (NSG) Requirements

Ensure the VMSS subnet NSG allows:

| Priority | Source                     | Port | Protocol | Action |
| -------- | -------------------------- | ---- | -------- | ------ |
| 100      | Application Gateway Subnet | 8080 | TCP      | Allow  |
| 110      | Admin IP                   | 22   | TCP      | Allow  |
| 200      | Internet                   | Any  | Any      | Deny   |

This configuration ensures:

* Backend instances are not internet-facing
* Only Application Gateway can communicate with the application port

---

## Enable Diagnostic and Monitoring

For production readiness, enable:

* Azure Monitor
* Boot diagnostics
* Log Analytics integration
* Autoscale history tracking

This provides visibility into scaling events and system health.

---

## Verify VM Scale Set Deployment

Navigate to:

```
Virtual Machine Scale Sets → webapp01-vmss → Instances
```

Confirm:

* Multiple instances are provisioned
* Provisioning state shows **Running**

Then verify backend health:

```
Application Gateway → Backend Health
```

All instances should show **Healthy**.

---

## Decommission Standalone VM

Once the VM Scale Set is verified and stable:

* Delete the original standalone `webapp01` VM

This ensures the environment operates entirely on elastic infrastructure managed by VMSS.

---

## Architectural Benefits of VM Scale Set Implementation

This Azure implementation provides:

* High availability across multiple Availability Zones
* Automatic horizontal scaling
* Self-healing infrastructure
* Integrated load balancing
* Cost optimization through dynamic scaling
* Cloud-native elasticity

By implementing VM Scale Sets, the architecture transitions from a single-instance deployment model to a resilient, scalable Azure-native design aligned with Microsoft Azure Well-Architected Framework principles.

