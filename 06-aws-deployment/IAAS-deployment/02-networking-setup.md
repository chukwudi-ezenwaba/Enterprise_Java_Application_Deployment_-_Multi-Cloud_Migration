## Networking Configuration

The network security architecture is designed following the principle of least privilege and layered defense. Three dedicated Security Groups (SGs) are implemented to enforce granular traffic control between the internet-facing layer, application tier, and backend services.

---

## Create Security Groups (SGs)
The following security groups are created.

### 1.load Balancer SG (Named SLSG)

The load balancer SG is configured to control inbound traffic from the public internet.

**Inbound Rules:**

* **TCP 80 (HTTP)** – Allowed temporarily for initial testing and redirection purposes.
* **TCP 443 (HTTPS)** – Allowed for secure production traffic.

This SG ensures that only web traffic is exposed publicly while preventing direct access to internal application or backend resources.

---

### 2. Web Application SG (Named WEBAPPSG)

The Web Application Security Group is attached to the EC2 instances running Tomcat service.

**Inbound Rules:**

* **TCP 8080 (Tomcat)** – Allowed only from the Load Balancer SG to ensure that application traffic flows exclusively through the load balancing layer.
* **TCP 22 (SSH)** – Restricted to the administrator’s public IP address for secure remote management.

This configuration prevents direct internet exposure of the application servers and ensures that all user traffic is inspected and routed through the load balancer.

---

### 3. Backend Services SG (Named BCKNDSG)

The Backend Security Group is associated with database and supporting service EC2 instance tightly restricted to application-tier resources only.

**Inbound Rules:**

* **TCP 3306 (MySQL)** – Allowed from the Web/Application subnet or corresponding SG.
* **TCP 11211 (Memcached)** – Allowed from the Web/Application subnet or corresponding SG.
* **TCP 5672 (RabbitMQ)** – Allowed from the Web/Application subnet or corresponding SG.
* **TCP 22 (SSH)** – Restricted to the administrator’s public IP address for controlled management access.
* **All Traffic (Any Protocol, Any Port)** – Permitted within the Backend Services security group only, allowing internal service-to-service communication among backend servers while preventing access from external or untrusted sources.

Backend services are not exposed to the public internet under any circumstance. All traffic is limited to trusted internal sources to maintain isolation and minimize the attack surface.

---

## Security Design Summary

* Public exposure is limited strictly to HTTPS via the load balancer.
* Application servers accept traffic only from the load balancing layer.
* Backend services are accessible solely from the application tier.
* Administrative SSH access is restricted to a specific trusted IP address.

This structured SG implementation enforces traffic segmentation, reduces lateral movement risk, and aligns with cloud security best practices for multi-tier application architectures.

### Create Key Pairs 
Create key pairs with following details.

To securly access the EC2 instances, key pairs are created. To create key pairs, define the following parameters 

* Name **web-app-key1**
* Key pair type **RSA**
* Private key file format **.pem** (This format is preferred for terminal or gitbash, if using putty for ssh, **.ppk** is prefered).
