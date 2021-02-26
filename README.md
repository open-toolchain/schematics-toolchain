# IBM Schematics Terraform Template

This repo contains an Tekton template for automating workflow with [IBM Cloud Schematics](http://cloud.ibm.com/schematics/), a service for managing infrastructure as code.  This template will configure a DevOps toolchain that will give you the option to invoke a delivery pipeline to update infrastructure automatically using the Terraform template's git repo.  

## Getting Started
1.  First, [create an IBM Cloud Schematics workspace](https://cloud.ibm.com/schematics/workspaces/create)
2. Once the Workspace is created, scroll down and click `Create Toolchain`.
3. Complete the toolchain setup form and click "Create" to create a toolchain.
4. Return to the workspace created, and import newly created Terraform Template git repo. *Might need to create an access token within Gitlab to import Template into Schematics Workspaces*

### To get started, click this button:
[![Deploy To IBM Cloud](https://console.bluemix.net/devops/graphics/create_toolchain_button.png)](https://cloud.ibm.com/devops/setup/deploy?repository=https://github.com/DavidLopezIBM/schematics-tekton&env_id=ibm:yp:us-south)

## Pipeline Details

### Schematics-Deploy Pipeline

The pipeline assumes a Schematics workspace already exists.  Follow the "Getting Started" section above to start by creating a Schematics workspace for you infrastructure as code/Terraform templates.

It implements the following best practices:
- Updates the Schematics workspace with any pipeline environment variables that match Terraform variables.  Any overrides to variables in the Schematics workspace should be done using environment variables on the delivery pipeline.
  - The "UPDATE" task will inspect the terraform template and iterate over template variables. 
  - If there is an environment variable that exactly matches the template variable, the Schematics workspace will be updated with the overriden values from the environment.
- Plan the changes to the infrastructure.  
- If the plan is successful, it will apply changes to the infrastructure.


### Schematics-PR

This pipeline listens for Pull Requests created against the `main` branch. The purpose of this pipeline is to pull source code and scan through the terraform files for compliance.

#### cra-terraform-scan
This task scans ibm-terraform-provider files for compliance issues.

Currently supported compliance checks:
|ID | Rule | 
|---------|---------|
|1| Ensure IAM does not allow too  many admins per account
|2| Ensure https for all inbound traffic via VPC LBaaS
|3| Ensure no VPC security groups allow incoming traffic from 0.0.0.0/0 to port 22 (ssh)
|4| Ensure no security groups have ports open to internet(0.0.0.0/0)
|5| Ensure VPC flowlogs logging is enabled in all VPCs
|6| Ensure no VPC security groups allow incoming traffic  from 0.0.0.0/0 to port RDP
|7| Ensure that COS Storage Encryption is set to On
|8| Ensure COS bucket is linked to LogDNA
|9| Ensure network access for COS is restricted to specific IP range
|10| Ensure certificates are automatically renewed before expiration. (This lifecycle applies to Certificate Manager generated certificates only)
|11| Ensure that Database for ElasticSearch Encryption is set to On
|12| Ensure that Database for ETCD Encryption is set to On
|13| Ensure that Database for MongoDB Encryption is set to On
|14| Ensure that Database for Radis Encryption is set to On
|15| Ensure that Database for PostgreSQL Encryption is set to On
|16| Ensure that Database for RabittMQ Encryption is set to On
|17| Ensure that Web application firewall is set to On in CIS
|18| Ensure ActivityTracker is provisioned
|19| Ensure Activity Tracker is provisioned in multiple regions for that account
|20| Ensure no all-resource IAM service policy
|21| Ensure IAM service policy is restricted using resource groups
|22| Ensure CIS is provisioned
|23| Ensure DDOS protection is set to On in CIS
|24| Ensure IAM does not authorize CIS to read from COS
|25| Ensure SSL is configured properly (using full/strict/origin_pull only)
|26| Ensure TLS 1.2 for all inbound traffic via CIS
|27| Ensure DNS record is protected
|28| Ensure IAM does not allow too many account managers per account
|29| Ensure IAM does not allow too many IAM admins per account
|30| Ensure IAM does not allow too many all resource managers per account
|31| Ensure IAM does not allow too many all resource readers per account
|32| Ensure IAM does not allow too many KMS managers per account
|33| Ensure IAM does not allow too many COS managers per account
|34| Ensure IAM users are attached to access groups
|35| Ensure CIS load balancer is provisioned
|36| Ensure CIS load balancer is  properly configured
|37| Ensure VPC has atleast one security group attached
|38| Ensure that Databases for ElasticSearch encryption is enabled with BYOK
|39| Ensure that Databases for MongoDB encryption is enabled with BYOK
|40| Ensure that Databases for PostgreSQL encryption is enabled with BYOK
|41| Ensure that Databases for RabbitMQ encryption is enabled with BYOK
|42| Ensure that Databases for ETCD encryption is enabled with BYOK
|43| Ensure that Databases for Redis encryption is enabled with BYOK
|44| Ensure that network access is set for ElasticSearch to be exposed on Private end Points only
|45| Ensure that network access is set for MongoDB to be exposed on Private end Points only
|46| Ensure that network access is set for PostgreSQL to be exposed on Private end Points only
|47| Ensure that network access is set for RabbitMQ to be exposed on Private end Points only
|48| Ensure that network access is set for ETCD to be exposed on Private end Points only
|49| Ensure that network access is set for Redis to be exposed on Private end Points only
|50| Ensure network access for Redis is restricted to specific IP range
|51| Ensure network access for ETCD is restricted to specific IP range
|52| Ensure network access for RabitMQ is restricted to specific IP range
|53| Ensure network access for PostgreSQL is restricted to specific IP range
|54| Ensure network access for MongoDB is restricted to specific IP range
|55| Ensure network access for ElasticSearch is restricted to specific IP range
|56| Ensure that COS Storage Encryption is set to On with BYOK
|57| Ensure that Database for ETCD Encryption is set to On


### Inputs

#### Parameters

  - **ibmcloud-api**: (Default: `https://cloud.ibm.com`) The ibmcloud api url
  - **repository**: The full URL path to the repo with the deployment files to be scanned
  - **revision**: (Default: `master`) The branch to scan
  - **tf-dir**: (Default `""`) The directory where the terraform main entry files are found.
  - **ibmcloud-apikey-secret-key**: (Default: apikey) field in the secret that contains the api key used to login to ibmcloud
  - **continuous-delivery-context-secret**: (Default: `secure-properties`) Reference name for the secret resource  
  - **directory-name**: The directory name where the repository is cloned
  - **pipeline-debug**: (Default: `0`) 1 = enable debug, 0 no debug
  - **policy-config-json**: (Default `""`) Configure policies thresholds
  - **pr-url**: The pull request url
  - **project-id**: (Default: `""`) Required id for GitLab repositories
  - **scm-type**: (Default: `github-ent`) Source code type used (github, github-ent, gitlab)
  - **resource-group**: (Default: `""`) target resource group (name or id) for the ibmcloud login operation
  - **git-access-token**: (Default: `""`) (optional) token to access the git repository. If this token is provided, there will not be an attempt to use the git token obtained from the authorization flow when adding the git integration in the toolchain
  - **tf-var-file**: (Default: `""`) Comma seperated list of tf-var files to be passed to terraform




#### Implicit
The following inputs are coming from tekton annotation:
 - **PIPELINE_RUN_ID**: ID of the current pipeline run

### Workspaces

- **artifacts**: The output volume to check out and store task scripts & data between tasks

### Results

- **status**: status of cra terraform scan task, possible value are-success|failure
- **evidence-store**: filepath to store terraform scan task evidence 
