## Build and Deployment of Application Artifact – AWS Implementation

This section documents the build and deployment lifecycle of the web application within the AWS environment. The workflow follows production-aligned DevOps practices, emphasizing artifact integrity, secure credential management, separation of responsibilities, and repeatable deployment processes.

The application artifact is compiled locally using Maven and then stored centrally in Amazon S3. The Tomcat EC2 instance retrieves the artifact securely using an IAM Role, ensuring that no static AWS credentials are embedded on the server.

This design reflects enterprise-grade deployment architecture and provides a foundation for CI/CD automation.

---

## 1. Application Build Process

The application source code is compiled and packaged locally using:

* **Java Development Kit (JDK)**
* **Apache Maven** (Build automation and dependency management tool)

The build process is executed using:

```bash
mvn clean install
```

## What This Command Performs

* `clean` removes previous build artifacts to ensure a fresh and consistent build.
* `install` compiles the source code, resolves project dependencies, runs unit tests (if configured), packages the application into a `.war` file, and installs the artifact into the local Maven repository.

Unlike `mvn package`, the `install` phase ensures the artifact is also stored locally in the Maven repository, making it reusable for dependent modules or future builds.

The resulting deployable artifact `vprofile-v2.war` is generated inside the:

```
target/
```

directory.

Before executing the build, the application configuration file is updated to reference internal DNS hostnames (e.g., `webappdb01.webapp.in`) rather than static private IP addresses. This ensures proper internal name resolution within the VPC and avoids service disruption if backend instances are replaced.

---

## 2. Centralized Artifact Storage – Amazon S3

To decouple the build process from the runtime environment, an Amazon S3 bucket is provisioned to act as centralized artifact storage.

This architectural decision enables:

* Version-controlled artifact storage
* Re-deployment without rebuilding
* Simplified rollback procedures
* Separation between application build and infrastructure layers

## S3 Bucket Configuration

* **Bucket Type:** General Purpose
* **Bucket Name:** `webapp01-s3`
* **Region:** us-east-1

Amazon S3 provides 99.999999999% (11 nines) durability, ensuring high availability and resilience for stored artifacts.

---

## 3. Secure Local Access – IAM User Configuration

To upload artifacts from the local workstation to S3, a dedicated IAM user is created.

## IAM User Details

* **Username:** `webapp01-admin`
* **Permission Model:** Attach policies directly
* **Policy Attached:** `AmazonS3FullAccess`

Access keys are generated under:

```
IAM → Users → Security Credentials → Create Access Key (CLI Use Case)
```

The Access Key ID and Secret Access Key are securely downloaded and stored locally.

> In a production-grade environment, a least-privilege bucket-specific policy would be implemented instead of full S3 access.

---

## 4. AWS CLI Configuration on Local Machine

The AWS CLI is configured to enable authenticated communication with AWS services.

```bash
aws configure
```

Input parameters:

* Access Key ID
* Secret Access Key
* Default Region: `us-east-1`
* Default Output Format: `json`

This configuration establishes secure programmatic access for artifact upload.

---

## 5. Uploading the Artifact to Amazon S3

Once the artifact has been successfully built, it is uploaded to the S3 bucket using:

```bash
aws s3 cp target/vprofile-v2.war s3://webapp01-s3
```

Verification command:

```bash
aws s3 ls s3://webapp01-s3/
```

Successful verification confirms:

* Proper CLI authentication
* Correct S3 permissions
* Artifact availability for runtime retrieval

---

## 6. IAM Role for Tomcat EC2 Instance

To avoid embedding credentials within the EC2 instance, an IAM Role is created and attached directly to the Tomcat server.

## IAM Role Configuration

* **Trusted Entity:** AWS Service
* **Use Case:** EC2
* **Policy Attached:** `AmazonS3FullAccess`
* **Role Name:** `S3-admin`

This enables AWS to automatically provide temporary security credentials to the instance via the Instance Metadata Service (IMDS).

## Attach Role to EC2 Instance

From the EC2 Console:

```
Actions → Security → Modify IAM Role → Select S3-admin → Update
```

This design ensures:

* No hardcoded credentials on the server
* Automatic credential rotation
* Centralized access control management
* Alignment with AWS security best practices

---

## 7. Deployment on Tomcat EC2 Instance

## Step 1: Connect to EC2 Instance

```bash
ssh -i path-to-key/web-app-key1.pem ec2-user@<Public-IP>
```

Elevate privileges:

```bash
sudo -i
```

---

## Step 2: Install AWS CLI on Server

```bash
snap install aws-cli --classic
```

Verify installation:

```bash
aws --version
```

Since the instance uses an IAM Role, no manual AWS credential configuration is required.

---

## Step 3: Retrieve Artifact from S3

```bash
aws s3 cp s3://webapp01-s3/vprofile-v2.war /tmp/
```

The artifact is now locally available on the EC2 instance.

---

## Step 4: Stop Tomcat Service

```bash
systemctl stop tomcat10
```

Stopping the service prevents file-locking conflicts during deployment.

---

## Step 5: Remove Existing Deployment

```bash
rm -rf /var/lib/tomcat10/webapps/ROOT
```

This ensures a clean deployment state before introducing the updated artifact.

---

## Step 6: Deploy Updated Artifact

```bash
cp /tmp/vprofile-v2.war /var/lib/tomcat10/webapps/ROOT.war
```

Renaming the file to `ROOT.war` ensures the application loads at the base context path (`/`).

---

## Step 7: Restart Tomcat Service

```bash
systemctl start tomcat10
```

Tomcat automatically extracts the WAR file and deploys the application.

Verification:

```bash
ls /var/lib/tomcat10/webapps/
```

---

## End-to-End Deployment Workflow Summary

1. Application is built locally using `mvn clean install`.
2. Artifact is uploaded to Amazon S3.
3. EC2 instance retrieves artifact using IAM Role authentication.
4. Tomcat service is restarted with updated deployment.
5. Application becomes accessible via the configured Application Load Balancer.

---

The solution aligns with the AWS Well-Architected Framework principles, particularly in the **Security**, **Operational Excellence**, and **Reliability** pillars.
