## AWS IaaS Deployment

The VProfile application is deployed in the AWS cloud using an Infrastructure-as-a-Service (IaaS) architecture designed for high availability, scalability, and fault tolerance. The deployment leverages core AWS services including **EC2**, **Application Load Balancer (ALB)**, **Auto Scaling Groups (ASG)**, **Amazon S3**, **Amazon EFS**, **Route 53**, and **AWS Certificate Manager (ACM)**.

This cloud-based architecture replaces the local NGINX reverse proxy with a managed load balancing solution and introduces dynamic scaling capabilities to meet fluctuating demand.

---

## Core Compute Layer – Amazon EC2

All application components are hosted on **Amazon Elastic Compute Cloud (EC2)** instances. Each backend service runs on dedicated EC2 instances to preserve architectural separation and allow independent scaling where necessary.

The following services are deployed on EC2:

* **Apache Tomcat** – Hosts the Java web application.
* **RabbitMQ** – Handles asynchronous message queuing.
* **Memcached** – Provides distributed in-memory caching.
* **MySQL** – Serves as the relational database engine.

Each EC2 instance is provisioned within a Virtual Private Cloud (VPC), segmented into public and private subnets:

* **Public Subnet:** Hosts the Application Load Balancer.
* **Private Subnet:** Hosts backend EC2 instances (Tomcat, RabbitMQ, Memcached, MySQL).

Security Groups and Network ACLs are configured to enforce least-privilege access control. For example:

* Only the ALB accepts inbound HTTP/HTTPS traffic from the internet.
* Backend services accept traffic only from trusted security groups.
* Database access is restricted to application servers.

---

## Load Balancing Layer – Application Load Balancer

Instead of NGINX, traffic distribution is managed by an **Elastic Load Balancing Application Load Balancer (ALB)**.

The ALB performs the following functions:

1. Distributes incoming HTTP/HTTPS traffic across multiple Tomcat instances.
2. Performs health checks to ensure traffic is routed only to healthy targets.
3. Enables high availability across multiple Availability Zones.
4. Terminates SSL/TLS connections (when integrated with ACM).

This managed service eliminates the operational overhead of maintaining a self-managed reverse proxy while improving resilience.

---

## Auto Scaling – Elastic Scalability

To ensure the system adapts dynamically to varying workloads, **Amazon EC2 Auto Scaling** is implemented.

Auto Scaling Groups (ASGs):

* Automatically launch new EC2 instances when CPU utilization or request count exceeds defined thresholds.
* Terminate excess instances during low-traffic periods.
* Maintain a minimum number of healthy instances.
* Distribute instances across multiple Availability Zones for redundancy.

This ensures cost efficiency while maintaining performance and uptime.

---

## Storage Layer – S3 and EFS

Two AWS storage services are integrated:

### 1. Object Storage

**Amazon S3 (Simple Storage Service)** is used for:

* Static assets (images, CSS, JavaScript files)
* Application backups
* Log archival

S3 provides highly durable (11 9’s durability) and scalable object storage.

### 2. Shared File Storage

**Amazon Elastic File System (EFS)** is used when shared file storage is required across multiple EC2 instances. This ensures consistency when scaling horizontally.

---

## DNS Management – Route 53

Domain name resolution is managed through **Amazon Route 53**.

Route 53 provides:

* Public or private DNS zones.
* Domain registration and record management.
* Routing policies (simple, weighted, failover, latency-based).
* Health-check-based routing.

This enables the application to be accessed via a domain name rather than a public IP address.

---

## Security – AWS Certificate Manager

To secure web traffic, **AWS Certificate Manager (ACM)** provisions and manages SSL/TLS certificates.

ACM:

* Issues and renews certificates automatically.
* Integrates directly with the Application Load Balancer.
* Enables HTTPS encryption for secure client-server communication.
* Eliminates manual certificate management overhead.

This ensures encrypted communication between users and the application, protecting credentials and sensitive data.

---

## High-Level Traffic Flow

1. User enters the application domain name.
2. Route 53 resolves the domain to the Application Load Balancer.
3. The ALB terminates HTTPS and forwards traffic to healthy Tomcat EC2 instances.
4. The application processes requests.
5. MySQL handles persistent data storage.
6. Memcached accelerates performance via caching.
7. RabbitMQ manages asynchronous backend tasks.
8. Static assets are retrieved from S3 or shared storage via EFS when required.

---

## Architectural Benefits

* **High Availability:** Multi-AZ deployment ensures resilience.
* **Elastic Scalability:** Auto Scaling dynamically adjusts compute capacity.
* **Managed Services:** Reduced operational overhead compared to self-managed infrastructure.
* **Security Best Practices:** Encrypted traffic, controlled network segmentation, and IAM-based access control.
* **Cost Optimization:** Scale-in during low demand minimizes unnecessary resource consumption.

This AWS IaaS deployment demonstrates proficiency in cloud infrastructure design, high-availability architectures, auto-scaling strategies, network security segmentation, and managed service integration within the AWS ecosystem.

