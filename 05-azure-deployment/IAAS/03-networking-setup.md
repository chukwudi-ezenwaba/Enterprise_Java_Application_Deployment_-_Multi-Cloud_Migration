## Networking Configuration

The network security architecture is designed following the principle of least privilege and layered defense. Three dedicated Network Security Groups (NSGs) are implemented to enforce granular traffic control between the internet-facing layer, application tier, and backend services.

---

## Create Virtual Network (Vnet)

## Virtual Network Architecture

All compute resources are deployed inside a dedicated Azure Virtual Network to enforce proper segmentation and secure traffic flow.

* **VNet Name:** `Webappvnet`

## Subnet Configuration

* **Application Gateway Subnet:** `AGsubnet`

 AGsubnet is a dedicated subnet that hosts the Azure Application Gateway instance. This subnet serves as the public-facing entry point for the web application and is responsible for handling all inbound HTTP/HTTPS traffic from the internet.

The Application Gateway deployed within this subnet performs Layer 7 (application-layer) routing, including URL-based routing, host-based routing, SSL termination, and health probing of backend services.

For security and compliance with Azure requirements (especially for Application Gateway v2 SKU), this subnet is reserved exclusively for the Application Gateway and does not contain any other resources. Network Security Group (NSG) rules are configured to allow required Azure infrastructure ports and inbound web traffic while preventing unauthorized access.

* **Web Application Subnet:** `Webapp01subnet`

  Webapp01subnet hosts the virtual machine running the Apache Tomcat web application service. This subnet represents the application tier in the architecture.

Access to this subnet is strictly controlled. The web server is not directly exposed to the public internet. Instead, it only accepts inbound traffic originating from the Application Gateway in AGsubnet. This ensures that all external traffic is first inspected, routed, and managed at the application gateway layer before reaching the web application.

Network Security Group rules are configured to:

* Allow inbound traffic from the Application Gateway subnet on the required application port 8080 for Tomcat.
* Deny direct internet access.
* Permit necessary outbound connections to backend services hosted in the backend subnet.

This design enforces proper tier isolation and reduces the attack surface of the application layer.

* **Backend Subnet:** `bckndsubnet`

  bckndsubnet hosts the backend infrastructure components, including the MySQL database server, RabbitMQ message broker, and Memcached caching service. This subnet represents the data and messaging tier of the application architecture.

It is strictly restricted to internal communication within the virtual network. Backend services are not accessible from the internet or from the Application Gateway directly. Instead, they are accessible only from the Webapp01subnet on the specific service ports required for application functionality (e.g., MySQL 3306, RabbitMQ 5672, Memcached 11211).

Network Security Group rules enforce:

* Inbound access only from the Webapp01subnet.
* Port-level restrictions based on service requirements.
* Denial of all unauthorized traffic from other sources.

This segmentation ensures secure east-west traffic flow and enforces a clear separation of responsibilities between the presentation, application, and data tiers.

Each subnet is protected using Network Security Groups (NSGs) aligned with least-privilege access control principles.

---

## Create Application Security Groups
* Webapp application security group (Named WEBAPPSG)
* Backend application security group (Named BCKNDASG)

---

## Create Network Security Groups (NSGs)
The following security groups are created.

**Inbound Rules:**

* **TCP 80 (HTTP)** – Allowed temporarily for initial testing and redirection purposes.
* **TCP 443 (HTTPS)** – Allowed for secure production traffic.

This NSG ensures that only web traffic is exposed publicly while preventing direct access to internal application or backend resources.

---

### 1. Application Gateway NSG (Named APGNSG)

The Application Gateway  Network Security Group allows only HTTP traffic inbound from the internet.

**Inbound Rules:**

* **TCP 80 (HTTP)** - Allow all HTTP traffic from the public internet.

---

### 2. Web Application NSG (Named WEBAPPNSG)

The Web Application Network Security Group is attached to the virtual machines running the Apache Tomcat service.

**Inbound Rules:**

* **TCP 8080 (Tomcat)** – Allowed only from the Application Gateway subnet to ensure that application traffic flows exclusively through the load balancing layer.
* **TCP 22 (SSH)** – Restricted to the administrator’s public IP address for secure remote management.
* **TCP 80 (HTTP)** - Temporarily restricted to and allowed from the administrator's pubic IP for testing purposes.

This configuration prevents direct internet exposure of the application servers and ensures that all user traffic is inspected and routed through the Application Gateway.

---

### 3. Backend Services NSG (Named BCKNDNSG)

The Backend Network Security Group is associated with database and supporting service virtual machines. Access is tightly restricted to application-tier resources only.

**Inbound Rules:**

* **TCP 3306 (MySQL)** – Allowed from the web application security group (WEBAPPASG).
* **TCP 11211 (Memcached)** – Allowed from web application security group (WEBAPPASG).
* **TCP 5672 (RabbitMQ)** – Allowed from the web application security group (WEBAPPASG)
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


