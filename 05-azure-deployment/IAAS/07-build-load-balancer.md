## Build and Deploy Load Balancer

To provide high availability, fault tolerance, and secure external access to the application in Azure, an **Application Gateway (Layer 7 Load Balancer)** is deployed in front of the Azure Virtual Machine hosting Apache Tomcat.

Unlike AWS, where an Application Load Balancer is used, Azure’s **Application Gateway** operates at Layer 7 and supports:

* HTTP/HTTPS routing
* SSL termination
* Health probes
* Web Application Firewall (optional)
* Path-based routing

This ensures traffic is securely distributed to healthy backend targets while maintaining centralized TLS management.

---

## Configure Backend Target

Before creating the Application Gateway, the backend pool (target) must be defined.

Navigate to:

**Azure Portal → Application Gateway → Create**

---

### Basic Configuration

* **Application gateway name:** `webapp01-agw`
* **Region:** Canada Central
* **Tier:** Standard v2 (Recommended for production)
* **Autoscaling:** Enabled (Minimum: 1 instance)
* **Minimum instance count** 0
* **Maximum instance count** 3
* **address type** IPv4
* **HTTP2** Enabled
* **Virtual Network:** `AGsubnet`
> Azure requires Application Gateway to reside in its own dedicated subnet.
* **Subnet:** `AppGatewaySubnet`
* **Frontend IP address type** Public
* **Public IPv4 address** x.x.x.x (Public IP)
* **Backend pool name** `webapp01`
* **Target type** Virtual machine
* **Target** webapp01

### Add a routing rule 
* **Rule name** webapp01-rule
* **Priority** 100

## Configure HTTP Listener

Under **Listeners**:

* **Name:** `http-listener`
* **Frontend IP:** Public IP
* **Protocol:** HTTP
* **Port:** 80
* **Listener type** Basic

Under **Backend targets**:
* **Target type** Backend pool

Under **Backend settings**:
* **Backend settings name** webapp01-settings
* **Backend protocol** HTTP
* **Backend port** 80

---

## Configure Health Probe

Health probes determine whether the backend instance is healthy before routing traffic.

Under **Health Probes**:

* **Protocol:** TCP
* **Port:** 8080
* **Path:** `/`
* **Interval:** 30 seconds
* **Unhealthy Threshold:** 3

This ensures:

* Unhealthy instances are automatically removed from rotation
* Traffic is routed only to responsive application servers

---

Associate this listener with a routing rule forwarding traffic to the backend pool.

---

## Configure HTTPS Listener

To enable secure communication:

### SSL Certificate Requirement

An SSL certificate must be uploaded or stored in **Azure Key Vault**.

Steps:

1. Obtain certificate (.pfx format).
2. Navigate to:

   ```
   Application Gateway → Listeners → Add Listener
   ```
3. Configure:

   * **Protocol:** HTTPS
   * **Port:** 443
   * **Upload Certificate** or reference from Key Vault

---

### HTTPS Listener Configuration

* **Name:** `https-listener`
* **Frontend IP:** Public
* **Protocol:** HTTPS
* **Port:** 443
* **SSL Certificate:** Select uploaded certificate

---

## Configure Routing Rules

Under **Rules**:

Create two rules:

### Rule 1 – HTTP

* **Listener:** http-listener
* **Backend Target:** webapp01 backend pool
* **Backend Settings:** HTTP, Port 8080

### Rule 2 – HTTPS

* **Listener:** https-listener
* **Backend Target:** webapp01 backend pool
* **Backend Settings:** HTTP, Port 8080

Application Gateway will terminate SSL at port 443 and forward traffic internally to port 8080.

---

## Network Security Group (NSG) Requirements

Ensure the VM’s NSG allows:

| Priority | Source                     | Port | Protocol | Action |
| -------- | -------------------------- | ---- | -------- | ------ |
| 100      | Application Gateway Subnet | 8080 | TCP      | Allow  |
| 110      | Admin IP                   | 22   | TCP      | Allow  |
| 200      | Internet                   | Any  | Any      | Deny   |

This ensures:

* The VM is not directly exposed to the internet
* Only the Application Gateway can communicate with the backend

---

## DNS Setup

After deployment, Azure generates a public IP address for the Application Gateway.

To map a custom domain:

1. Log in to your DNS provider (e.g., GoDaddy).
2. Navigate to DNS Management.
3. Add a new record:

### DNS Record Configuration

* **Type:** A Record
* **Name:** webapp01
* **Value:** Public IP of Application Gateway
* **TTL:** 30 Minutes

This maps:

```
webapp01.yourdomain.com → Azure Application Gateway
```

---

## Verify Backend Health

Navigate to:

**Application Gateway → Backend Health**

Confirm:

* Backend status shows **Healthy**
* Health probe responses are successful

If unhealthy, verify:

* Tomcat service is running
* Port 8080 is open in NSG
* Health probe path is correct

---

## Application Access Verification

### Test via HTTP

Access:

```
http://<Application-Gateway-Public-IP>
```

You should see the webapp01 login page.
Connection will show as **Not Secure**.

---

### Test via HTTPS

Access:

```
https://webapp01.yourdomain.com
```

You should:

* See the login page
* Observe secure HTTPS connection
* See browser padlock indicating valid SSL

---

## Architectural Benefits of Azure Load Balancer Implementation

This Azure load balancing configuration provides:

* Layer 7 intelligent routing
* SSL/TLS termination at the gateway
* Backend health monitoring
* Network isolation of the VM
* Autoscaling capability (v2 SKU)
* Integration with Azure Key Vault (optional)
* Optional Web Application Firewall (WAF) support

By placing Application Gateway in front of the Virtual Machine, the architecture follows Azure best practices for secure, scalable, and resilient web application deployment.

---

If you'd like, I can now generate a comparison section for your GitHub repo showing AWS ALB vs Azure Application Gateway to highlight your multi-cloud competence.


