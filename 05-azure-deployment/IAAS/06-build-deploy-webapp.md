## Build and Secure Deployment of Application Artifact – Azure (Canada Central)

This document outlines a production-aligned deployment architecture for hosting a Java-based web application on an Azure Virtual Machine running Apache Tomcat. The application artifact is built locally using Maven and securely stored in Azure Blob Storage. The Virtual Machine retrieves the artifact using Managed Identity authentication, eliminating the need for static credentials.

All resources are deployed in the **Canada Central** region to ensure geographic consistency, compliance alignment, and reduced latency.

---

## 1. Resource Organization

## Resource Group

All resources are deployed within:

* **Resource Group:** Webapp01
* **Region:** Canada Central

Centralizing resources within a single resource group enables simplified lifecycle management, cost tracking, and infrastructure governance.

---

## 2. Application Build Process

The application is compiled locally using:

* Java Development Kit (JDK)
* Apache Maven

Build command:

```bash
mvn clean install
```

This command:

* Cleans previous build artifacts
* Compiles source code
* Resolves dependencies
* Executes unit tests (if configured)
* Packages the application into a `.war` file
* Installs the artifact into the local Maven repository

The deployable artifact is generated in:

```
target/vprofile-v2.war
```

Before building, backend configurations are updated to reference internal Azure DNS hostnames rather than static IP addresses.

---

## 3. Azure Storage Account – Secure Artifact Repository

## Storage Account Configuration

* **Name:** webapp01storage (globally unique)
* **Region:** Canada Central
* **Performance Tier:** Standard
* **Redundancy:** LRS (Locally Redundant Storage)
* **Account Type:** StorageV2
* **Access Tier:** Hot
* **Minimum TLS Version:** 1.2
* **Public Blob Access:** Disabled

### Redundancy Considerations

| Option | Description                              | Use Case                       |
| ------ | ---------------------------------------- | ------------------------------ |
| LRS    | Replicates data within single datacenter | Cost-effective lab environment |
| ZRS    | Replicates across availability zones     | High availability production   |
| GRS    | Replicates to secondary region           | Disaster recovery scenario     |

For this environment, **LRS** is sufficient. In production, **ZRS** or **GRS** would be recommended depending on business continuity requirements.

---

## Blob Container

* **Container Name:** artifacts
* **Public Access Level:** Private

This ensures artifacts are accessible only through authenticated Azure identities.

---

## 4. Storage Security Hardening

To align with enterprise security standards, additional configurations are implemented:

### 1. Disable Public Network Access (Optional Production Setting)

Set:

```
Storage Account → Networking → Public network access → Disabled
```

This ensures storage is only accessible through Private Endpoints.

---

### 2. Enable Secure Transfer Required

Ensures HTTPS-only traffic to the storage account.

---

### 3. Private Endpoint Configuration

To prevent storage access over the public internet:

1. Navigate to:

   ```
   Storage Account → Networking → Private Endpoint → Add
   ```

2. Configure:

   * Subnet: Application Subnet
   * Private DNS Integration: Enabled

This assigns a private IP address to the storage account within the Virtual Network, ensuring traffic remains internal to Azure.

---

## 5. Lifecycle Management Policy

Blob lifecycle management reduces long-term storage costs and enforces governance.

policy:

* Move blobs to Cool tier after 30 days
* Move blobs to Archive tier after 90 days
* Delete blobs after 365 days

Configure under:

```
Storage Account → Lifecycle Management → Add Rule
```

This supports operational efficiency and cost optimization.

---

### Security Principles Applied

* Least privilege network access
* SSH restricted to administrator IP only
* No open inbound ports beyond required services

---

## 6. Managed Identity Configuration

To eliminate credential management risks:

## Enable System-Assigned Managed Identity

```
VM → Identity → System Assigned → On
```

Azure automatically creates a service principal linked to the VM.

---

## Assign Role to Storage Account

Navigate to:

```
Storage Account → Access Control (IAM) → Add Role Assignment
```

Assign:

* **Role:** Storage Blob Data Reader
* **Principal:** VM Managed Identity

This grants read-only access to artifacts, following least privilege principles.

---

## 7. Secure Artifact Upload from Local Machine

Authenticate:

```bash
az login
```

Upload artifact:

```bash
az storage blob upload \
  --account-name webapp01storage \
  --container-name artifacts \
  --name vprofile-v2.war \
  --file target/vprofile-v2.war \
  --auth-mode login
```

---

## 8. Deployment on Azure VM

## Step 1 – Connect to VM

```bash
ssh azureuser@<Public-IP>
```

---

## Step 2 – Install Azure CLI

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | bash
```

---

## Step 3 – Authenticate Using Managed Identity

```bash
az login --identity
```

No credentials required.

---

## Step 4 – Download Artifact

```bash
az storage blob download \
  --account-name webapp01storage \
  --container-name artifacts \
  --name vprofile-v2.war \
  --file /tmp/vprofile-v2.war \
  --auth-mode login
```

---

## Step 5 – Deploy to Tomcat

Stop service:

```bash
systemctl stop tomcat10
```

Remove existing deployment:

```bash
rm -rf /var/lib/tomcat10/webapps/ROOT
```

Copy artifact:

```bash
cp /tmp/vprofile-v2.war /var/lib/tomcat10/webapps/ROOT.war
```

Restart service:

```bash
systemctl start tomcat10
```

---

## 9. Optional Production Enhancements

* Enable Azure Monitor and Log Analytics
* Enable Microsoft Defender for Cloud
* Configure Backup for VM
* Use Availability Zones for higher resiliency
* Enable Just-in-Time (JIT) VM Access for SSH security

---

## End-to-End Architecture Overview

1. Artifact built locally using Maven
2. Artifact uploaded to Azure Blob Storage (Canada Central)
3. Storage secured via Private Endpoint and RBAC
4. VM authenticates using Managed Identity
5. Artifact retrieved securely
6. Application deployed and served via Load Balancer or Application Gateway
