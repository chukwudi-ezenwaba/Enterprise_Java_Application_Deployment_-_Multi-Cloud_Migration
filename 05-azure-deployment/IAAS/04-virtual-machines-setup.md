# Microsoft Azure Infrastructure Configuration

The application infrastructure is deployed in Microsoft Azure using dedicated Azure Virtual Machines, each assigned a single responsibility to ensure service isolation, scalability, and maintainability.

The following services are hosted on individual Azure Virtual Machines:

* **Apache Tomcat** – Application server hosting the Java web application
* **MySQL** – Relational database backend
* **RabbitMQ** – Message broker for asynchronous processing
* **Memcached** – In-memory caching layer

Each virtual machine is provisioned using **Custom Script Extensions (cloud-init or Azure VM extensions)** during deployment. The automation scripts are stored in the repository under the **`/automation`** directory and are responsible for installing packages, configuring services, and applying runtime configurations.

This approach ensures repeatable, consistent, Infrastructure-as-Code–aligned deployments within Azure.

---

### MySQL Azure Virtual Machine Configuration

The MySQL virtual machine is dedicated to database services and is provisioned using a custom initialization script.

## #Automation Script Responsibilities

The MySQL deployment script performs the following:

* Installs MySQL server packages
* Configures secure database settings
* Creates the application database
* Creates an administrative database user
* Grants appropriate privileges
* Deploys the database schema file
* Configures MySQL to start automatically on boot

This ensures the database server is fully configured during initial provisioning without manual intervention.

---

## Instance Specifications

* **Region:** East US
* **Virtual Machine Name:** `webappdb01`
* **Image:** Ubuntu Server 22.04 LTS
* **VM Size:** Standard D2s_v3 (cost-optimized for lab environment)
* **Authentication type:** SSH public key
* **SSH public key source:** Generate new key pair
* **SSH Key Type:** RSA SSH Format
* **Public inbound port rules:** Allow selected ports
* **Select inbound ports:** SSH (22)
* **OS disk size:** 30 GiB
* **OS disk type:** Premium SSD
* **Key management:** Platform-managed key
* **Virtual Network:** `Webappvnet`
* **Subnet:** `bckndsubnet`
* **Network Security Group:**
Backend NSG allowing:

* TCP 3306 from `Webappsubnet`
* SSH (22) restricted to administrator IP

**Extensions:**
Custom Script Extension enabled
MySQL automation script executed during provisioning

---

## Memcached Azure Virtual Machine Configuration

The Memcached virtual machine provides high-speed in-memory caching for the application tier.

## Automation Script Responsibilities

The Memcached deployment script:

* Installs Memcached
* Starts and enables the service
* Verifies service status
* Updates configuration to allow controlled remote access
* Configures service to listen on port 11211
* Restarts the service to apply changes

Access is strictly limited to application-tier virtual machines within `Webappsubnet`.

---

## Instance Specifications

* **Region:** East US
* **Virtual Machine Name:** `webappmc01`
* **Image:** Ubuntu Server 22.04 LTS
* **VM Size:** Standard D2s_v3 (cost-optimized for lab environment)
* **Authentication type:** SSH public key
* **SSH public key source:** Generate new key pair
* **SSH Key Type:** RSA SSH Format
* **Public inbound port rules:** Allow selected ports
* **Select inbound ports:** SSH (22)
* **OS disk size:** 30 GiB
* **OS disk type:** Premium SSD
* **Key management:** Platform-managed key
* **Virtual Network:** `Webappvnet`
* **Subnet:** `bckndsubnet`


**Network Security Group:**
Backend NSG allowing:

* TCP 11211 from `Webappsubnet`
* SSH restricted to administrator IP

**Extensions:**
Custom Script Extension enabled
Memcached automation script executed at deployment

This configuration ensures caching services remain internal and inaccessible from the public internet.

---

## RabbitMQ Azure Virtual Machine Configuration

The RabbitMQ virtual machine is responsible for asynchronous message handling between application components.

Deployment is automated using Azure VM Custom Script Extensions.

## Automation Script Responsibilities

The RabbitMQ provisioning script:

* Imports GPG signing keys for RabbitMQ and Erlang repositories
* Configures package repositories
* Installs Erlang runtime (dependency)
* Installs RabbitMQ server
* Enables and starts the service
* Creates administrative user
* Assigns appropriate role permissions
* Restarts the service to apply configuration

RabbitMQ listens on port 5672, restricted to the application subnet.

---

## Instance Specifications

**Region:** East US
**Virtual Machine Name:** `webapprmq01`
**Image:** Ubuntu Server 22.04 LTS
**VM Size:** Standard B1s
**Virtual Network:** `Webappvnet`
**Subnet:** `bckndsubnet`
**Network Security Group:**
Backend NSG allowing:

* TCP 5672 from `Webappsubnet`
* SSH restricted to administrator IP

**Extensions:**
Custom Script Extension enabled
RabbitMQ automation script executed at deployment

---

## Apache Tomcat Azure Virtual Machine Configuration

The Tomcat virtual machine hosts the Java-based web application and operates within the application tier.

## Automation Script Responsibilities

The Tomcat deployment script performs the following:

* Installs Java runtime environment
* Downloads and installs Apache Tomcat 10
* Configures systemd service
* Deploys application WAR file
* Sets appropriate file permissions
* Starts and enables Tomcat
* Verifies service availability on port 8080

The Tomcat VMs are not publicly exposed. Traffic flows exclusively through Azure Application Gateway deployed in `AGsubnet`.

---

## Instance Specifications

* **Region:** East US
* **Virtual Machine Name:** `webapp01`
* **Image:** Ubuntu Server 24.04 LTS
* **VM Size:** Standard B1s
* **Virtual Network:** `Webappvnet`
* **Subnet:** `Webappsubnet`
* **Network Security Group:**
Web Tier NSG allowing:

* TCP 8080 from `AGsubnet`
* SSH restricted to administrator IP

**Extensions:**
Custom Script Extension enabled
Tomcat automation script executed during deployment

---

# Deployment Design Summary

* All virtual machines are deployed within `Webappvnet`
* Application Gateway is deployed in `AGsubnet`
* Application servers reside in `Webappsubnet`
* Backend services reside in `bckndsubnet`
* NSGs enforce strict east-west and north-south traffic control
* Automation scripts ensure consistent provisioning
* No backend services are publicly exposed
## Service Validation – Azure Virtual Machine Deployment

After provisioning the Azure Virtual Machines, each service is validated to ensure successful installation, configuration, and operational readiness. Administrative access is performed securely using SSH through either a public IP (restricted to a trusted IP) or via Azure Bastion for enhanced security.

> **Security Note:** For production-style deployments, SSH access should be performed through Azure Bastion or restricted using Network Security Groups (NSGs) to a specific administrator public IP address.

---

# Testing MySQL Azure Virtual Machine

### Step 1: Connect via SSH

Using a public IP:

```bash
ssh -i path-to-key/web-app-key1.pem azureuser@<Public-IP>
```
Using Azure Bastion:

* Navigate to the VM in the Azure Portal
* Select **Connect → Bastion**
* Authenticate using SSH private key

---

### Step 2: Verify Database Service Status

```bash
sudo systemctl status mysql
```

Ensure the service status shows **active (running)**.

---

### Step 3: Connect to the Database

```bash
mysql -u admin -p accounts
```

Enter the configured password (e.g., `admin123` for lab purposes).

---

### Step 4: Validate Database Schema

```sql
SHOW TABLES;
```

Successful output confirms:

* The database was successfully created
* The schema file was deployed correctly
* The administrative user has appropriate privileges

---

# Testing Memcached Azure Virtual Machine

### Step 1: Connect via SSH

```bash
ssh -i path-to-key/web-app-key1.pem azureuser@<Public-IP>
```

Or connect securely via Azure Bastion.

---

### Step 2: Verify Service Status

```bash
sudo systemctl status memcached
```

Confirm the service is **active (running)**.

---

### Optional: Verify Port Listening

```bash
sudo ss -tulnp | grep 11211
```

This confirms Memcached is listening on the expected port and accessible only from authorized internal subnets (e.g., `Webappsubnet`).

---

# Testing RabbitMQ Azure Virtual Machine

### Step 1: Connect via SSH

```bash
ssh -i path-to-key/web-app-key1.pem azureuser@<Public-IP>
```

Or use Azure Bastion.

---

### Step 2: Verify RabbitMQ Service Status

```bash
sudo systemctl status rabbitmq-server
```

Ensure the service is **active (running)**.

---

### Optional: Verify Listening Port

```bash
sudo ss -tulnp | grep 5672
```

This confirms RabbitMQ is listening on its default messaging port and restricted to internal application-tier access.

---

# Testing Apache Tomcat Azure Virtual Machine

### Step 1: Connect via SSH

```bash
ssh -i path-to-key/web-app-key1.pem azureuser@<Public-IP>
```

---

### Step 2: Verify Tomcat Service Status

```bash
sudo systemctl status tomcat
```

Confirm that the service is **active (running)**.

---

### Step 3: Validate Application Accessibility

From a browser:

* Navigate to the Application Gateway public IP or DNS name
* Confirm the application loads successfully

Or verify locally from the VM:

```bash
curl http://localhost:8080
```

A valid HTTP response confirms the application server is operational.

---

# Validation Summary

Successful validation confirms:

* Azure VM Custom Script Extensions executed correctly
* Services are installed, enabled, and persistent across reboots
* NSG rules are enforcing proper subnet-level isolation
* Application-tier services are accessible only through Azure Application Gateway
* Backend services remain private within `bckndsubnet`


