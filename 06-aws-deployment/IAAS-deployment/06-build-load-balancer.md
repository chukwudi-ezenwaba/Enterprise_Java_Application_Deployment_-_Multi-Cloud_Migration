## Build and Deploy Load Balancer

To provide high availability, fault tolerance, and secure external access to the application, an **Application Load Balancer (ALB)** is deployed in front of the EC2 instance hosting Apache Tomcat. The ALB distributes incoming client traffic and performs continuous health monitoring of backend targets.

Before creating the load balancer, a **Target Group** must be created. The target group defines which resources will receive traffic and how health checks are performed.

---

## Set Target Group

Navigate to:

**EC2 Console → Load Balancing → Target Groups → Create Target Group**

### Target Group Configuration

* **Target Type:** Instances
* **Target Group Name:** `webapp01-tg`
* **Protocol:** HTTP
* **Port:** 8080

Port 8080 is used because the Tomcat application server is configured to listen on this port internally.

### Health Check Configuration

Under **Advanced health check settings**:

* **Health Check Protocol:** HTTP
* **Override Port:** 8080

Health checks ensure that traffic is only routed to healthy backend instances. If an instance fails the health check, it is automatically removed from the load balancing rotation until it recovers.

---

### Register Targets

In the **Register Targets** section:

1. Select the EC2 instance `webapp01`.
2. Confirm that under **Ports for the selected instances**, port **8080** is selected.
3. Click **Include as pending below**.
4. Verify the instance appears under **Review Targets**.
5. Click **Create Target Group**.

At this stage, the target group is configured and ready to receive traffic from the load balancer.

---

## Create Load Balancer

Navigate to:

**EC2 Console → Load Balancers → Create Load Balancer**

### Load Balancer Type

* **Type:** Application Load Balancer

The Application Load Balancer operates at Layer 7 (HTTP/HTTPS), enabling advanced routing, SSL termination, and host-based rules.

---

### Basic Configuration

* **Load Balancer Name:** `webapp01-elb`
* **Scheme:** Internet-facing
* **IP Address Type:** IPv4
* **VPC:** Default VPC (or project VPC if customized)

---

### Network Mapping

Under **Availability Zones**, select all available zones within the region.

This ensures:

* High availability
* Multi-AZ redundancy
* Fault tolerance in case of zone failure

---

### Security Group

Attach the security group:

* **Security Group:** `LBSG`

The Load Balancer Security Group (LBSG) should allow:

* Inbound HTTP (Port 80) from Internet (0.0.0.0/0)
* Inbound HTTPS (Port 443) from Internet (0.0.0.0/0)

The EC2 instance security group must allow inbound traffic on port 8080 **only from the Load Balancer security group**, not from the internet directly. This enforces network isolation and improves security posture.

---

### Listeners and Routing Configuration

Configure the following listeners:

#### HTTP Listener

* **Protocol:** HTTP
* **Port:** 80
* **Forward To:** `webapp01-tg`

This allows users to access the application over standard HTTP.

---

#### HTTPS Listener

Add another listener:

* **Protocol:** HTTPS
* **Port:** 443
* **Forward To:** `webapp01-tg`

Under **Default SSL/TLS Server Certificate**:

* **Certificate Source:** From ACM
* Select the validated SSL certificate provisioned in AWS Certificate Manager (ACM).

The HTTPS listener performs SSL termination at the load balancer level, ensuring encrypted client communication while forwarding unencrypted traffic internally to port 8080.

---

Click **Create Load Balancer** and allow AWS to provision the resource.

Once deployed, AWS generates a DNS name similar to:

```
webapp01-elb-xxxxxxxx.region.elb.amazonaws.com
```

---

## DNS Setup on GoDaddy

To associate a custom domain with the load balancer, DNS configuration is completed through the domain registrar. In this lab, GoDaddy was used.

### Steps:

1. Log in to the GoDaddy account.
2. Navigate to the **Domain** section.
3. Open the **DNS Management** tab.
4. Select **Add a New Record**.

### Record Configuration

* **Type:** CNAME
* **Name:** webapp01
* **Value:** Load Balancer DNS Name (e.g., `webapp01-elb-xxxx.amazonaws.com`)
* **TTL:** 0.5 Hour

This configuration maps:

```
webapp01.yourdomain.com → AWS Load Balancer
```

DNS propagation may take several minutes depending on TTL settings.

---

## Verify EC2 Instance Health

To confirm that traffic is being routed correctly:

Navigate to:

**EC2 Console → Target Groups → webapp01-tg → Targets**

Under the **Registered Targets** tab:

* Click **Refresh**
* Confirm the target status shows **Healthy**

If the target is unhealthy, verify:

* Port 8080 is open on the EC2 security group
* Security group allows inbound from the Load Balancer security group
* Application is running on Tomcat

---

### Validate Health Check Settings

Switch to the **Health Check Settings** tab and confirm:

* Protocol: HTTP
* Port: 8080
* Path: (default `/` or application-specific health endpoint)

---

## Application Access Verification

### Test via Load Balancer DNS

Copy the Load Balancer DNS name and paste into a web browser:

```
http://<load-balancer-dns>
```

You should be presented with the login page of `webapp01`.

At this stage, the connection will display as **HTTP (Not Secure)**.

---

### Test Secure HTTPS Access

Access the application using:

```
https://webapp01.yourdomain.com
```

Example:

```
https://webapp01.nig-e-mart.store
```

You should again see the login page, but now with a secure HTTPS connection.

The browser should indicate:

* Valid SSL certificate
* Encrypted communication
* Secure padlock icon

---

## Architectural Benefits of the Load Balancer Implementation

This load balancing configuration provides:

* High availability across multiple Availability Zones
* Automatic health monitoring of backend instances
* SSL/TLS termination for secure client communication
* Isolation of backend EC2 instances from direct internet exposure
* Scalable foundation for future Auto Scaling integration

By placing the Application Load Balancer in front of the EC2 instance, the architecture adheres to AWS best practices for secure, scalable, and resilient web application deployment.



