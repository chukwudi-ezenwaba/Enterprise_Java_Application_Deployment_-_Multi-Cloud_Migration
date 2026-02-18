## EC2 Instance Configuration

The application infrastructure is deployed using four dedicated Amazon EC2 instances, each assigned a single responsibility to ensure service isolation, scalability, and maintainability.

The following services are hosted on individual EC2 instances:

* **Apache Tomcat** – Application server hosting the Java web application
* **MySQL** – Relational database backend
* **RabbitMQ** – Message broker for asynchronous processing
* **Memcached** – In-memory caching layer

Each instance is provisioned with automated bootstrap scripts supplied via EC2 User Data during launch. These scripts are maintained in the repository under the **`/automation`** directory and are responsible for installing packages, configuring services, and applying necessary runtime configurations.

This approach ensures repeatable, consistent, and infrastructure-as-code–aligned deployments.

---

# MySQL EC2 Instance Configuration

The MySQL instance is dedicated to database services and is provisioned using a custom initialization script.

### Automation Script Responsibilities

The MySQL deployment script performs the following tasks:

* Installs MySQL server packages
* Configures secure database settings
* Creates the application database
* Creates an administrative database user
* Grants appropriate privileges to the user
* Deploys the database schema file
* Configures MySQL to start on boot

This ensures the database server is fully configured automatically upon instance initialization without manual intervention.

---

## Instance Specifications

**Region:** US East (N. Virginia)

**Instance Name:** `db01`

**Tags:**

* Key: `Project`
* Value: `webapp01`
* Applied to: Instances and Volumes

**Amazon Machine Image (AMI):**

* Amazon Linux 2023

**Instance Type:**

* t3.micro (Free Tier eligible)

**Key Pair:**

* `web-app-key1`

**Network Configuration:**

* Attached to existing security group: `BCKNDSG`
* Deployed in private subnet 

**Advanced Configuration:**

* User Data script enabled
* MySQL automation script injected during launch

---

## Deployment Approach

The EC2 instance is launched with a User Data script that executes during first boot. This enables:

* Automated database provisioning
* Elimination of manual post-deployment configuration
* Consistency across environments
* Faster recovery in case of instance replacement

By separating the database workload onto a dedicated EC2 instance and enforcing controlled network access via backend security groups, the architecture aligns with production-grade cloud deployment standards and follows best practices for secure multi-tier application design.

---

If you would like, I can now:

* Rewrite the Tomcat, RabbitMQ, and Memcached instance sections in the same professional format
* Convert this entire EC2 section into a clean GitHub README production-ready layout
* Or optimize it for a resume/cloud portfolio summary
