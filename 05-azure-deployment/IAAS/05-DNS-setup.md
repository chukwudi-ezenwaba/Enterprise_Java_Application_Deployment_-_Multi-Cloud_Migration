## DNS Configuration – Azure Private DNS

The application source code references backend services using hostnames and port numbers (e.g., `webappdb01.webapp.in:3306`). For proper communication between virtual machines within the virtual network, these hostnames must resolve to their respective private IP addresses.

To enable secure internal name resolution, **Azure Private DNS** is implemented. This allows DNS resolution strictly within the Azure Virtual Network without exposing records to the public internet.

---

## Private DNS Zone Configuration

A Private DNS Zone is created to provide internal hostname resolution for all application and backend services.

### Private DNS Zone Details

* **Zone Name:** `webapp.in`
* **Resource Group:** `Webapp01`
* **Region:** Canada Central
* **Virtual Network Link:** `Webappvnet`

The Private DNS Zone is linked to the virtual network (`Webappvnet`) to enable automatic name resolution for all virtual machines deployed within the VNet.

This configuration ensures that DNS records are resolvable only internally and not accessible from external networks.

---

## DNS Record Creation

Within the Private DNS Zone, individual A records are created for each service instance. Each record maps a hostname to the corresponding private IP address of the virtual machine.

### Database Server

* **Record Name:** `webappdb01`
* **FQDN:** `webappdb01.webapp.in`
* **Record Type:** A
* **Value:** Private IP address of MySQL VM

### Memcached Server

* **Record Name:** `webappmc01`
* **FQDN:** `webappmc01.webapp.in`
* **Record Type:** A
* **Value:** Private IP address of Memcached VM

### RabbitMQ Server

* **Record Name:** `webapprmq01`
* **FQDN:** `webapprmq01.webapp.in`
* **Record Type:** A
* **Value:** Private IP address of RabbitMQ VM

### Application Server

* **Record Name:** `webapp01`
* **FQDN:** `webapp01.webapp.in`
* **Record Type:** A
* **Value:** Private IP address of Tomcat VM

This configuration enables reliable hostname-based communication between application-tier and backend-tier resources.

---

## DNS Resolution Testing

After DNS records are created and the Private DNS Zone is linked to the VNet, name resolution is validated from a virtual machine within `Webappvnet`.

### Test Commands

```bash
ping -c 4 webappdb01.webapp.in
ping -c 4 webappmc01.webapp.in
ping -c 4 webapprmq01.webapp.in
ping -c 4 webapp01.webapp.in
```

Alternatively, more precise DNS validation can be performed using:

```bash
nslookup webappdb01.webapp.in
```

Successful resolution confirms:

* The Private DNS Zone is properly linked to `Webappvnet`
* DNS records are correctly configured
* Internal hostname-based service communication is functional
* The application configuration referencing backend hostnames will resolve successfully

