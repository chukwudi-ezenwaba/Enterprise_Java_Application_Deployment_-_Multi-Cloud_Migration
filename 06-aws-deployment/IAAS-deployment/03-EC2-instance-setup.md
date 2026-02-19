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

## MySQL EC2 Instance Configuration

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

**Instance Name:** `webappdb01`

**Tags:**

* Key: `Project`
* Value: `webappdb01`
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

## Memcached EC2 Instance Configuration

The Memcached instance is dedicated to providing high-speed, in-memory caching for the application layer. This improves performance by reducing repeated database queries and lowering application response latency.

The instance is provisioned using an automated bootstrap script supplied through EC2 User Data. The automation script is maintained in the repository under the **`/automation`** directory.

### Automation Script Responsibilities

The Memcached deployment script performs the following tasks:

* Installs Memcached packages
* Starts and enables the Memcached service at boot
* Verifies service status
* Modifies the configuration file to allow remote connections
* Configures Memcached to listen on required ports (11211 and 11111 where applicable)
* Restarts the service to apply configuration changes

Remote access is restricted to trusted backend or application-tier security groups in accordance with least-privilege principles.

---

### Instance Specifications

**Region:**
US East (N. Virginia)

**Instance Name:**
`webappmc01`

**Tags:**

* Key: `Project`
* Value: `webappmc01`
* Applied to: Instances and Volumes

**Amazon Machine Image (AMI):**

* Amazon Linux 2023

**Instance Type:**

* t3.micro (Free Tier eligible)

**Key Pair:**

* `web-app-key1`

**Network Configuration:**

* Attached to existing security group: `BCKNDSG`
* Deployed in a private subnet

**Advanced Configuration:**

* User Data script enabled
* Memcached automation script injected during instance launch

This configuration ensures that caching services are fully provisioned automatically and remain isolated from public exposure.

---

## RabbitMQ EC2 Instance Configuration

The RabbitMQ instance is responsible for handling asynchronous messaging between application components. This decouples frontend request handling from backend processing, improving scalability and reliability.

Deployment is fully automated using a custom User Data script stored in the **`/automation`** directory.

### Automation Script Responsibilities

The RabbitMQ provisioning script performs the following:

* Imports GPG signing keys for RabbitMQ and Erlang repositories
* Configures repository definitions (including Erlang dependency)
* Installs Erlang runtime (required dependency)
* Installs RabbitMQ server packages
* Enables and starts the RabbitMQ service
* Configures a test administrative user
* Assigns administrative role permissions
* Restarts the service to apply configuration changes

Repository configuration files are securely downloaded and placed in `/etc/yum.repos.d` to enable proper package management.

Access to RabbitMQ (default port 5672) is restricted to application-tier security groups.

---

### Instance Specifications

**Region:**
US East (N. Virginia)

**Instance Name:**
`webapprmq01`

**Tags:**

* Key: `Project`
* Value: `webapprmq01`
* Applied to: Instances and Volumes

**Amazon Machine Image (AMI):**

* Amazon Linux 2023

**Instance Type:**

* t3.micro (Free Tier eligible)

**Key Pair:**

* `web-app-key1`

**Network Configuration:**

* Attached to existing security group: `BCKNDSG`
* Deployed in a private subnet

**Advanced Configuration:**

* User Data script enabled
* RabbitMQ automation script injected during instance launch

This automated approach ensures consistent message broker deployment aligned with production-ready practices.

---

## Apache Tomcat EC2 Instance Configuration

The Tomcat EC2 instance hosts the Java-based web application and serves as the application-tier compute layer.

This instance is provisioned using a custom automation script that installs and configures Apache Tomcat 10 on an Ubuntu Server environment.

### Automation Script Responsibilities

The Tomcat deployment script performs the following:

* Installs required system dependencies (Java runtime environment)
* Downloads and installs Apache Tomcat 10
* Configures systemd service for Tomcat
* Deploys the application WAR file
* Sets appropriate file permissions
* Starts and enables the Tomcat service
* Verifies service availability on port 8080

The application server is accessible only through the Load Balancer, ensuring it is not directly exposed to the public internet.

---

### Instance Specifications

**Region:**
US East (N. Virginia)

**Instance Name:**
`webapp01`

**Tags:**

* Key: `Project`
* Value: `webapp01`
* Applied to: Instances and Volumes

**Amazon Machine Image (AMI):**

* Ubuntu Server 24.04 LTS

**Instance Type:**

* t3.micro (Free Tier eligible)

**Key Pair:**

* `web-app-key1`

**Network Configuration:**

* Attached to existing security group: `WEBAP01SG`
* Deployed in a private subnet

**Advanced Configuration:**

* User Data script enabled
* Tomcat automation script injected during instance launch

---

## Service Validation – EC2 Deployment

After provisioning the EC2 instances, each service is validated to confirm successful installation, configuration, and operational status. Administrative access is performed securely using SSH and the associated private key.

> **Note:** SSH access is restricted to a trusted administrator IP address via Security Group rules.

---

## Testing MySQL EC2 Instance

### Step 1: Connect via SSH

```bash
ssh -i path-to-key/web-app-key1.pem ec2-user@<Public-IP>
```

### Step 2: Verify Database Service Status

```bash
sudo systemctl status mariadb
```

Ensure the service is in an **active (running)** state.

### Step 3: Connect to the Database

```bash
mysql -u admin -p accounts
```

When prompted, enter the configured password (e.g., `admin123` for lab purposes).

### Step 4: Validate Database Schema

```sql
SHOW TABLES;
```

Successful output confirms that:

* The database was created
* The schema file was properly deployed
* The administrative user has appropriate privileges

---

# Testing Memcached EC2 Instance

### Step 1: Connect via SSH

```bash
ssh -i path-to-key/web-app-key1.pem ec2-user@<Public-IP>
```

### Step 2: Verify Service Status

```bash
sudo systemctl status memcached
```

Confirm the service is **active (running)**.

### Optional: Verify Port Listening

```bash
sudo ss -tulnp | grep 11211
```

This confirms Memcached is listening on the expected port and accessible only from authorized security groups.

---

# Testing RabbitMQ EC2 Instance

### Step 1: Connect via SSH

```bash
ssh -i path-to-key/web-app-key1.pem ec2-user@<Public-IP>
```

### Step 2: Verify RabbitMQ Service Status

```bash
sudo systemctl status rabbitmq-server
```

Confirm the service is **active (running)**.

### Optional: Verify Listening Port

```bash
sudo ss -tulnp | grep 5672
```

This ensures RabbitMQ is properly listening on its default messaging port.

---

# Testing Tomcat EC2 Instance

### Step 1: Connect via SSH

```bash
ssh -i path-to-key/web-app-key1.pem ubuntu@<Public-IP>
```

### Step 2: Verify RabbitMQ Service Status

```bash
sudo systemctl status tomcat10
```

Confirm the service is **active (running)**.

This ensures RabbitMQ is properly listening on its default messaging port.
## Validation Summary

Successful validation confirms:

* Services were provisioned correctly via EC2 User Data automation scripts
* Systemd services are enabled and persistent across reboots
* Application dependencies are operational
* Network security rules are correctly allowing restricted access

This structured validation process aligns with production deployment verification practices and ensures infrastructure reliability before application integration testing.
