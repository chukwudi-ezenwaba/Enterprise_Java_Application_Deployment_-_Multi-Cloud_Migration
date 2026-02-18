## EC2 Instance Setup

Four EC2 instances is setup to host the following services:
* **Tomcat service** 
* **MySQL service**
* **Rabbit MQ**
* **Memcache**

As these instances are deployed, custom sripts to setup the required services each instance would host is added to the deployment setup. These scripts are located in the **automation** section.

### MYSQL Ec2 Instance.

 The **mysql** script contains querries to srtup all that is required for the sql server, create database. create admin user and grant it privileges on the account database and finally deploy the db schema file.

* Instance name: **db01**
* Names and tags: Key **Project**, Value **webapp01**, Resources **Instances** **Volumes**
* Application and OS images: **Amazon Linux 2023AMI**
* Instance type: **T3 micro** (Free tier)
* Key pair: **web-app-key1**
* Network settings: Select existing security group - **BCKNDSG**
* Advanced details: **User data - optional** **add mysql deployment script here**