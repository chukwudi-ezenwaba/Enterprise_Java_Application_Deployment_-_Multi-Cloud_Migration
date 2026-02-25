## Configure Auto Scaling Group

To ensure the application can automatically adjust to fluctuating traffic demands, an **Auto Scaling Group (ASG)** is implemented for the `webapp01` EC2 instance. Auto Scaling is a core AWS capability that maintains application availability while dynamically scaling compute capacity up or down based on demand.

Rather than relying on a single static EC2 instance, the Auto Scaling Group ensures:

* High availability
* Fault tolerance
* Automatic instance replacement if unhealthy
* Elastic scaling during peak traffic
* Cost optimization during low demand

The Auto Scaling architecture is built using three key components:

* A **custom Amazon Machine Image (AMI)**
* A **Launch Template**
* The **Auto Scaling Group (ASG)** itself

---

## Create EC2 Instance AMI

An Amazon Machine Image (AMI) serves as a blueprint for launching new EC2 instances. It contains:

* Operating system
* Installed software (Tomcat, application dependencies)
* Configuration settings
* Deployed application artifacts

Creating an AMI ensures that every new instance launched by the Auto Scaling Group is identical to the original `webapp01` server.

### Steps to Create the AMI

1. Select the checkbox for the `webapp01` EC2 instance.
2. Navigate to:

   ```
   Actions → Image and templates → Create image
   ```
3. Configure:

   * **Image name:** `webapp01-ami`
   * **Image description:** `webapp01-ami`
4. Click **Create image**.

To verify creation:

```
EC2 → Images → AMIs
```

Wait until the AMI status shows **Available** before proceeding.

This AMI will now act as the golden image for scaling operations.

---

## Create Launch Template

A Launch Template defines the configuration used when launching EC2 instances. It standardizes instance parameters such as:

* AMI ID
* Instance type
* Security groups
* Key pair
* IAM role
* Resource tags

This ensures consistency across all scaled instances.

### Steps to Create Launch Template

Navigate to:

```
EC2 → Launch Templates → Create launch template
```

### Configuration Details

* **Launch template name:** `webapp01-LT`

Under **Application and OS Images**:

* Select **My AMIs**
* Choose the previously created `webapp01-ami`

Under **Instance Type**:

* Select `t3.micro` (or appropriate instance type)

Under **Key Pair**:

* Select the SSH key pair created earlier

Under **Network Settings → Security Groups**:

* Select `WEBAPP01SG`

This security group should:

* Allow inbound port 8080 from the Load Balancer Security Group
* Allow SSH (22) only from administrative IP

---

### Resource Tagging

Under **Resource Tags**, add:

* **Key:** Name

  * **Value:** webapp01
  * **Resource types:** Instances, Volumes

* **Key:** Project

  * **Value:** webapp
  * **Resource types:** Instances, Volumes

Tagging improves resource organization, cost tracking, and governance.

---

### IAM Role Attachment

Under **Advanced Details**:

* **IAM Instance Profile:** Select `s3-admin` (or appropriate role)

This ensures new instances can retrieve artifacts securely from S3 without embedded credentials.

Click **Create launch template** to complete.

---

## Create Auto Scaling Group

The Auto Scaling Group manages a fleet of EC2 instances and enforces scaling policies.

Navigate to:

```
EC2 → Auto Scaling Groups → Create Auto Scaling group
```

### Basic Configuration

* **Name:** `webapp01-asg`
* **Launch Template:** `webapp01-LT`

---

### Network Configuration

* **VPC:** Select project VPC
* **Availability Zones:** Select all available AZs

Selecting multiple Availability Zones ensures high availability and fault tolerance in case one zone fails.

---

### Attach to Load Balancer

Under **Load Balancing**:

* Select **Attach to an existing load balancer**
* Choose **From your load balancer target groups**
* Select `webapp01-tg`

This ensures that any instance launched by the ASG is automatically registered with the Application Load Balancer.

---

### Health Checks

Enable:

* **Turn on Elastic Load Balancing health checks**

This allows the Auto Scaling Group to use ALB health status in addition to EC2 status checks. If an instance fails health checks, it is automatically terminated and replaced.

---

## Configure Group Size

Auto Scaling groups require three capacity settings:

* **Minimum Capacity:** 1
* **Desired Capacity:** 2
* **Maximum Capacity:** 4

### Meaning of These Values

* **Minimum:** Ensures at least one instance always runs.
* **Desired:** Number of instances launched initially.
* **Maximum:** Upper limit during scale-out events.

This configuration ensures baseline availability while allowing growth under load.

---

## Configure Automatic Scaling Policy

Under **Automatic Scaling**, select:

* **Scaling Policy Type:** Target Tracking
* **Metric Type:** Average CPU Utilization
* **Target Value:** 50%

### Scaling Logic

* If average CPU exceeds 50%, new instances are launched (scale out).
* If average CPU drops significantly below 50%, instances are terminated (scale in).

Target tracking policies simplify scaling management by automatically maintaining the defined utilization threshold.

Click **Create Auto Scaling Group** to finalize deployment.

---

## Enable Sticky Sessions (Session Affinity)

To maintain consistent user sessions across requests:

1. Navigate to:

   ```
   EC2 → Target Groups → webapp01-tg
   ```
2. Select **Attributes → Edit**
3. Enable **Stickiness**

Stickiness ensures a client is routed to the same backend instance for the duration of their session. This is important for applications that store session state locally rather than in a shared cache.

---

## Verify Auto Scaling Deployment

Navigate to:

```
EC2 → Target Groups → webapp01-tg → Registered Targets
```

Confirm:

* Multiple instances are registered
* Health status shows **Healthy**

Then navigate to:

```
EC2 → Instances
```

You should see instances launched automatically by the Auto Scaling Group.

---

## Remove Standalone Instance

To fully transition to an elastic architecture:

* Delete the original standalone `webapp01` EC2 instance

This ensures all compute capacity is managed exclusively by the Auto Scaling Group.

---

## Architectural Benefits of Auto Scaling Implementation

This configuration provides:

* High availability across multiple Availability Zones
* Automatic replacement of failed instances
* Elastic scaling based on CPU demand
* Integration with Application Load Balancer
* Cost efficiency during low traffic periods
* Foundation for future horizontal scaling architecture
