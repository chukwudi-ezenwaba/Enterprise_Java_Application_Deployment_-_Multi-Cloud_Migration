
The artifact for the web app would be built from the souce code in a local pc and the artifact pushed to S3 bucket. From the webapp we would fecth the artifact and deploy it to rhe tomcat service. The artifact would be built from the source code using mavin and JDK. Communication from the local PC to S3 bucket would be done using AWS CLI. 

An S3 bucket is created for this purpose. 

For authentication from local PC to S3 bucket IAM keys are setup from IAM user and the keys stored on local PC. 

From the server running tomcat, AWS CLI is also used to communicate with the S3 bucket. IAM role is also created for the tomcat server with permission to access the S3 bucket. 

## S3 Bucket Configuration
* Bucket type - **General purpose**
* Bucket name - **webapp01-s3**

## IAM Keys Configuration 
This is configured from the IAM blade on the portal using the following parameters. 
An IAM user is firstr created with the details below and then keys created. 
* username - **webapp01-admin**
* Permission option - **Attach policies directly**
* Permission policies - **AmazonS3FullAccess**

From the created user, in the **Security credentials** tab, in the **Access keys** section the access keys are created with **use case** as **CLI** and download csv files with access keys

## IAM Role for `webapp01` Configuration
The IAM role is created from the IAM blade in the **roles** tab using the **create role** button.

* Trusted entity type - **AWS service** 
* Use case - **EC2**
* Permissions policies - **AmazonS3FullAccess**
* name - **S3-admin** 

## Apply Role to `webapp01` 
on the `webapp01` in the instances blade right-click and navigate to **actions** >> **security** >> **Modify IAM role** >> select `S3-admin` role >> **Update IAM role**
