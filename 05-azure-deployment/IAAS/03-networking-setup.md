## Networking Configuration

The network security architecture is designed following the principle of least privilege and layered defense. Three dedicated Network Security Groups (NSGs) are implemented to enforce granular traffic control between the internet-facing layer, application tier, and backend services.

---

## Create Virtual Network (Vnet)

## Virtual Network Architecture

All compute resources are deployed inside a dedicated Azure Virtual Network to enforce proper segmentation and secure traffic flow.

* **VNet Name:** `Webappvnet`

## Subnet Configuration

* **Application Gateway Subnet:** `AGsubnet`

  * Dedicated subnet for Azure Application Gateway
  * Public-facing entry point
  * Handles HTTPS traffic and Layer 7 routing

* **Web Application Subnet:** `Webapp01subnet`

  * Hosts Apache Tomcat virtual machines
  * Accessible only from Application Gateway

* **Backend Subnet:** `bckndsubnet`

  * Hosts MySQL, RabbitMQ, and Memcached virtual machines
  * Restricted to internal application-tier communication only

Each subnet is protected using Network Security Groups (NSGs) aligned with least-privilege access control principles.

---

## Create Network Security Groups (NSGs)
The following security groups are created.

### 1. Application Gateway NSG (Named APGNSG)

The Application Gateway Network Security Group is configured to control inbound traffic from the public internet.

**Inbound Rules:**

* **TCP 80 (HTTP)** – Allowed temporarily for initial testing and redirection purposes.
* **TCP 443 (HTTPS)** – Allowed for secure production traffic.

This NSG ensures that only web traffic is exposed publicly while preventing direct access to internal application or backend resources.

## Create Application Gateway Application Security Groups
* **Name:** `APGASG`
---

### 2. Web Application NSG (Named WEBAPPNSG)

The Web Application Network Security Group is attached to the virtual machines running the Apache Tomcat service.

**Inbound Rules:**

* **TCP 8080 (Tomcat)** – Allowed only from the Application Gateway NSG to ensure that application traffic flows exclusively through the load balancing layer.
* **TCP 22 (SSH)** – Restricted to the administrator’s public IP address for secure remote management.

This configuration prevents direct internet exposure of the application servers and ensures that all user traffic is inspected and routed through the Application Gateway.

---

### 3. Backend Services NSG (Named BCKNDNSG)

The Backend Network Security Group is associated with database and supporting service virtual machines. Access is tightly restricted to application-tier resources only.

**Inbound Rules:**

* **TCP 3306 (MySQL)** – Allowed from the Web/Application subnet or corresponding NSG.
* **TCP 11211 (Memcached)** – Allowed from the Web/Application subnet or corresponding NSG.
* **TCP 5672 (RabbitMQ)** – Allowed from the Web/Application subnet or corresponding NSG.
* **TCP 22 (SSH)** – Restricted to the administrator’s public IP address for controlled management access.
* **All Traffic (Any Protocol, Any Port)** – Permitted within the Backend Services security group only, allowing internal service-to-service communication among backend servers while preventing access from external or untrusted sources.

Backend services are not exposed to the public internet under any circumstance. All traffic is limited to trusted internal sources to maintain isolation and minimize the attack surface.

---

## Security Design Summary

* Public exposure is limited strictly to HTTPS via the load balancer.
* Application servers accept traffic only from the load balancing layer.
* Backend services are accessible solely from the application tier.
* Administrative SSH access is restricted to a specific trusted IP address.

This structured NSG implementation enforces traffic segmentation, reduces lateral movement risk, and aligns with cloud security best practices for multi-tier application architectures.


