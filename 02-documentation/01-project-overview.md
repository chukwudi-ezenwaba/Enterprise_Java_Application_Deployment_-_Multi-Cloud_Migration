### Project Overview
This project simulates an enterprise Java application lifecycle from on-premises deployment to multi-cloud modernization. It demonstrates how traditional workloads can be migrated to Azure and AWS while applying cloud-native design principles to improve reliability, scalability, and security.

### on-prem-deployment.

**VProfile** is a multi-tier Java-based web application designed and deployed using a distributed microservices-oriented architecture. The application stack consists of five independent services, each deployed on its own dedicated virtual machine (VM) to simulate a production-like environment. This separation of concerns improves scalability, fault isolation, and service-level management while enabling realistic testing prior to cloud deployment.

The architecture is divided into frontend and backend service tiers:

#### Frontend Tier

1. **Nginx** – Functions as a reverse proxy and load balancer.
2. **Apache Tomcat** – Hosts and runs the Java web application (WAR file).

#### Backend Tier

3. RabbitMQ – Message broker for asynchronous communication.
4. Memcached – Distributed in-memory caching layer.
5. MySQL – Relational database management system (RDBMS).

Each service is provisioned on an independent virtual machine using Infrastructure as Code (IaC) principles. The virtualization platform used is Oracle VM VirtualBox, while environment orchestration and provisioning are handled through Vagrant. All provisioning logic is implemented using shell scripts defined within the `Vagrantfile`, ensuring repeatability, automation, and consistency across deployments.

This local virtualized environment provides a controlled platform for development, testing, and research (R&D). It allows for configuration validation, performance testing, and service integration verification before migrating the workload to a cloud environment such as AWS or Azure. By replicating a multi-tier production architecture locally, the deployment reduces configuration drift and increases deployment confidence.

---

### ARCHITECTURE AND TRAFFIC FLOW

The application follows a standard three-tier architecture with additional messaging and caching layers to optimize performance and scalability.

1. #### Client Request Initiation
   A user enters the application’s URL or IP address into a web browser. The DNS resolution (or direct IP routing in a lab setup) directs traffic to the frontend load balancer.

2. #### Reverse Proxy and Load Balancing (Nginx)
   The request is received by Nginx, which operates as a reverse proxy and load balancer. Nginx forwards HTTP requests to the Apache Tomcat application server based on defined upstream configurations.

3. #### Application Processing (Apache Tomcat)
   Apache Tomcat hosts the deployed Java web application (VProfile). Upon receiving the request, the application server:

    Renders the login interface.
    Accepts user credentials (username and password).
    Performs authentication and application logic processing.

4. #### Database Interaction (MySQL)
   On initial login or registration, user credentials are securely stored in the MySQL relational database. MySQL serves as the system of record, ensuring data persistence and transactional integrity.

5. #### Caching Layer (Memcached)
   Frequently accessed data, such as authenticated session information or user profile data, is cached in Memcached. This reduces repeated database queries, lowers latency, and improves overall application performance.

6. #### Asynchronous Messaging (RabbitMQ)
   RabbitMQ facilitates asynchronous communication between the web application and backend processing components. It manages message queues that decouple frontend request handling from backend task execution. This improves scalability and fault tolerance by ensuring tasks are processed reliably without blocking user-facing operations.

#### Summary of Request Flow:

Client → Nginx (Reverse Proxy) → Apache Tomcat (Java App) → MySQL (Persistent Storage) → Memcached (Caching) → RabbitMQ (Message Queue Processing)

This layered approach enhances performance, scalability, reliability, and modularity.

---

### INFRASTRUCTURE PROVISIONING AND EXECUTION

The entire environment is provisioned using Vagrant, which automates virtual machine lifecycle management on VirtualBox. Each service is defined within the `Vagrantfile`, including:

* VM hostname
* Network configuration (private IP addressing)
* Resource allocation (CPU, RAM)
* Provisioning scripts

All configuration and installation steps are automated using shell provisioning scripts. These scripts install required packages, configure services, and deploy the application components.

#### Deployment Command

To provision and start the complete environment, the following command is executed:

```bash
vagrant up
```

This command performs the following actions:

1. Creates all defined virtual machines.
2. Allocates system resources.
3. Executes provisioning scripts.
4. Installs and configures each service.
5. Deploys the Java application on Apache Tomcat.
6. Establishes service communication between all nodes.

The result is a fully functional, production-like multi-tier application environment running locally.

---

### DESIGN BENEFITS

* **Isolation:** Each service runs independently, reducing cross-service interference.
* **Scalability Testing:** Services can be horizontally scaled in future iterations.
* **Automation:** Infrastructure as Code ensures consistent deployments.
* **Production Simulation:** Mirrors real-world cloud architecture patterns.
* **Reduced Deployment Risk:** Enables thorough validation before cloud migration.

This implementation demonstrates competencies in virtualization, service orchestration, Linux administration, reverse proxy configuration, Java application deployment, messaging systems, caching strategies, and infrastructure automation.

### AWS-deployment.