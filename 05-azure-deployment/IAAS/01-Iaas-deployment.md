## Azure IaaS Deployment

The webapp01 application is deployed in Microsoft Azure using an Infrastructure-as-a-Service (IaaS) architecture designed for high availability, scalability, and secure enterprise-grade operations. This deployment mirrors the AWS-based architecture but leverages native Azure services for compute, load balancing, storage, DNS, and certificate management.

The core Azure services used include **Azure Virtual Machines**, **Azure Application Gateway**, **Virtual Machine Scale Sets**, **Azure Storage Account**, **Azure blobs**, **Azure DNS**.

---

## 1. Compute Layer – Azure Virtual Machines

All application components were hosted on **Microsoft Azure Virtual Machines**, which provide scalable and customizable compute capacity in the cloud.

The following services are deployed on separate VMs:

* **Apache Tomcat** – Hosts the Java web application (WAR file).
* **RabbitMQ** – Manages asynchronous message queuing.
* **Memcached** – Provides in-memory caching for performance optimization.
* **MySQL** – Serves as the relational database backend.

### 2. Network Architecture

The infrastructure is deployed within an **Azure Virtual Network (VNet)** and segmented into subnets:

* **Public Subnet** – Hosts the load balancing layer.
* **Front End Subnet** – Hosts the web application servers.
* **Back End  Subnet** – Hosts the backend servers.

Access control is enforced using:

* Network Security Groups (NSGs) to filter traffic.
* Application Security Groups to group the virtual machines and apply different NSG inbound rules.
* Role-Based Access Control (RBAC) for identity governance.

This segmentation ensures that both the frontend and backend services are not directly exposed to the public internet.

### 3. Load Balancing Layer

Instead of NGINX, Azure-native load balancing services are used.

### 3.1 Azure Application Gateway 

**Azure Application Gateway** is deployed as the primary HTTP load balancer.

It provides:

* Layer 7 (application-level) load balancing
* SSL/TLS termination
* Web Application Firewall (WAF) capability
* URL-based routing
* Health probes for backend instances

Application Gateway distributes incoming user traffic across multiple Tomcat instances deployed in a scale set.

## A3.2 uto Scaling – Virtual Machine Scale Sets

Elastic scalability is achieved using **Azure Virtual Machine Scale Sets**.

VM Scale Sets (VMSS):

* Automatically scale out (add VMs) during high CPU utilization or increased HTTP request rates.
* Scale in during low traffic periods to optimize cost.
* Distribute instances across Availability Zones.
* Maintain a defined minimum and maximum instance count.

This ensures high availability while maintaining cost efficiency.

---

## 4. Storage Services

Azure provides both object and shared file storage options:

### 1. Object Storage

**Azure Blob Storage** is used for:

* Static web assets (images, CSS, JS)
* Log backups
* Database exports
* Application artifacts

Blob Storage provides massive scalability and high durability.

---

## DNS Management

Domain resolution is handled using **Azure DNS**.

Azure DNS provides:

* Public and private DNS zones
* Custom domain name management
* Alias records for Azure resources
* Integration with Azure networking services

This allows users to access the application using a domain name instead of a public IP address.

---

## Security – Certificates and Secrets Management

TLS/SSL certificates are managed using **Azure Key Vault**.

Azure Key Vault:

* Securely stores SSL certificates, secrets, and keys
* Integrates with Application Gateway for HTTPS termination
* Enables automatic certificate rotation
* Enforces strict access policies using RBAC

This ensures encrypted HTTPS communication between users and the application.

---

## High-Level Traffic Flow

1. A user enters the application domain name in a browser.
2. Azure DNS resolves the domain to the public IP of the Application Gateway.
3. Application Gateway terminates HTTPS and forwards the request to healthy backend VM instances running Apache Tomcat.
4. The Java web application processes authentication and business logic.
5. MySQL handles persistent storage.
6. Memcached accelerates performance by caching frequently accessed data.
7. RabbitMQ processes asynchronous backend messaging tasks.
8. Static files are retrieved from Azure Blob Storage or Azure Files where applicable.

---

## High Availability and Fault Tolerance

* VMs are distributed across multiple Availability Zones.
* Health probes ensure traffic is routed only to healthy instances.
* Scale Sets automatically replace failed VMs.
* Backup and snapshot policies can be configured using Azure Backup.

---

## Architectural Benefits

* **Enterprise-Grade Security:** NSGs, RBAC, Key Vault integration.
* **Elastic Scalability:** VM Scale Sets respond dynamically to demand.
* **High Availability:** Multi-zone deployment ensures resilience.
* **Operational Efficiency:** Managed load balancing and DNS services reduce administrative overhead.
* **Cloud-Native Design:** Follows Azure Well-Architected Framework principles.

This Azure IaaS deployment demonstrates proficiency in Azure networking (VNet, NSGs), compute (VMs, VMSS), load balancing (Application Gateway), storage architecture, identity governance, and secure cloud infrastructure design aligned with real-world enterprise environments.
