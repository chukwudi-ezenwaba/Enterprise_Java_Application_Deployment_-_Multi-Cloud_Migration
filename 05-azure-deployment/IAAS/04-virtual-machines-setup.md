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

## MySQL Azure Virtual Machine Configuration

The MySQL virtual machine is dedicated to database services and is provisioned using a custom initialization script.

## Automation Script Responsibilities

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

**Region:**
East US

**Virtual Machine Name:**
`webappdb01`

**Image:**
Ubuntu Server 22.04 LTS (or Azure-supported Linux distribution)

**VM Size:**
Standard B1s (cost-optimized for lab environment)

**Virtual Network:**
`Webappvnet`

**Subnet:**
`bckndsubnet`

**Network Security Group:**
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

**Region:**
East US

**Virtual Machine Name:**
`webappmc01`

**Image:**
Ubuntu Server 22.04 LTS

**VM Size:**
Standard B1s

**Virtual Network:**
`Webappvnet`

**Subnet:**
`bckndsubnet`

**Network Security Group:**
Backend NSG allowing:

* TCP 11211 from `Webappsubnet`
* SSH restricted to administrator IP

**Extensions:**
Custom Script Extension enabled
Memcached automation script executed at deployment

This configuration ensures caching services remain internal and inaccessible from the public internet.

---

# RabbitMQ Azure Virtual Machine Configuration

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

**Region:**
East US

**Virtual Machine Name:**
`webapprmq01`

**Image:**
Ubuntu Server 22.04 LTS

**VM Size:**
Standard B1s

**Virtual Network:**
`Webappvnet`

**Subnet:**
`bckndsubnet`

**Network Security Group:**
Backend NSG allowing:

* TCP 5672 from `Webappsubnet`
* SSH restricted to administrator IP

**Extensions:**
Custom Script Extension enabled
RabbitMQ automation script executed at deployment

---

# Apache Tomcat Azure Virtual Machine Configuration

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

**Region:**
East US

**Virtual Machine Name:**
`webapp01`

**Image:**
Ubuntu Server 24.04 LTS

**VM Size:**
Standard B1s

**Virtual Network:**
`Webappvnet`

**Subnet:**
`Webappsubnet`

**Network Security Group:**
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

This Azure deployment mirrors enterprise-grade multi-tier architecture patterns and demonstrates expertise in:

* Azure Virtual Networking and Subnet Segmentation
* NSG-based traffic isolation
* VM provisioning with automation
* Secure application-tier design
* Cloud-native infrastructure best practices
