## DNS Configuration – AWS Private Hosted Zone

The application source code references backend services using hostnames and port numbers (e.g., `webappdb01.webapp.in:3306`). For proper internal communication between EC2 instances, these hostnames must resolve to their corresponding private IP addresses.

To enable internal name resolution within the VPC, **Amazon Route 53** Private DNS is implemented.

---

# Private Hosted Zone Configuration

A Private Hosted Zone is created to provide internal DNS resolution strictly within the associated VPC. This ensures backend services remain private and are not publicly resolvable over the internet.

### Hosted Zone Details

* **Domain Name:** `webapp.in`
* **Type:** Private Hosted Zone
* **Region:** US East (N. Virginia)
* **Associated VPC:** Default VPC (or designated application VPC)

By associating the hosted zone with the VPC, only resources within that VPC can resolve these DNS records.

---

# DNS Record Creation

After creating the Private Hosted Zone, individual A records are configured for each service instance.

### Database Server

* **Record Name:** `webappdb01.webapp.in`
* **Record Type:** A
* **Value:** Private IP address of MySQL EC2 instance

### Memcached Server

* **Record Name:** `webappmc01.webapp.in`
* **Record Type:** A
* **Value:** Private IP address of Memcached EC2 instance

### RabbitMQ Server

* **Record Name:** `webapprmq01.webapp.in`
* **Record Type:** A
* **Value:** Private IP address of RabbitMQ EC2 instance

### Application Server

* **Record Name:** `webapp01.webapp.in`
* **Record Type:** A
* **Value:** Private IP address of Tomcat EC2 instance

Each record maps a hostname to its respective private IP, enabling seamless service-to-service communication within the VPC.

---

# DNS Resolution Testing

After configuring the records, internal DNS resolution is validated from an EC2 instance within the same VPC.

### Test Commands

```bash
ping -c 4 webappdb01.webapp.in
ping -c 4 webappmc01.webapp.in
ping -c 4 webapprmq01.webapp.in
ping -c 4 webapp01.webapp.in
```

Successful resolution confirms:

* Route 53 Private Hosted Zone is correctly associated with the VPC
* DNS records are properly configured
* Internal hostname-based communication is functional
* Application configuration referencing service hostnames will resolve correctly


