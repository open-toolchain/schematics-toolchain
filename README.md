# IBM Schematics Terraform Template

This repo contains an Tekton template for automating workflow with [IBM Cloud Schematics](http://cloud.ibm.com/schematics/), a service for managing infrastructure as code.  This template will configure a DevOps toolchain that will give you the option to invoke a delivery pipeline to update infrastructure automatically using the Terraform template's git repo.  

## Getting Started
1.  First, [create an IBM Cloud Schematics workspace](https://cloud.ibm.com/schematics/workspaces/create)
2. Once the Workspace is created, scroll down and click `Create Toolchain`.
3. Complete the toolchain setup form and click "Create" to create a toolchain.
4. Return to the workspace created, and import newly created Terraform Template git repo. *Might need to create an access token within Gitlab to import Template into Schematics Workspaces*

### To get started, click this button:
[![Deploy To IBM Cloud](https://console.bluemix.net/devops/graphics/create_toolchain_button.png)](https://cloud.ibm.com/devops/setup/deploy?repository=https://github.com/open-toolchain/schematics-toolchain&env_id=ibm:yp:us-south)

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

#### terraform-v2

This task uses `ibmcloud cli` and the `cra` plugin to scan ibm-terraform-provider files for compliance issues.
##### Inputs

##### Parameters

  - **continuous-delivery-context-secret**: (Default: `secure-properties`) Reference name for the secret resource
  - **ibmcloud-api**: (Default: `https://cloud.ibm.com`) The ibmcloud api url
  - **ibmcloud-apikey-secret-key**: (Default: `apikey`) field in the secret that contains the api key used to login to ibmcloud
  - **ibmcloud-region**: (Optional) The ibmcloud target region
  - **pipeline-debug**: (Default: `0`) 1 = enable debug, 0 no debug
  - **resource-group**: (Optional) Target resource group (name or id) for the ibmcloud login operation
  - **custom-script**: (Optional) A custom script to be ran prior to CRA scanning
  - **ibmcloud-trace**: (Default: `false`) Enables IBMCLOUD_TRACE for ibmcloud cli logging
  - **output**: (Default: `false`) Prints command result to console
  - **path**: (Default: `/artifacts`) Directory where the repository is cloned
  - **strict**: (Optional) Enables strict mode for scanning
  - **toolchainid**: The ibmcloud target toolchain to be used
  - **verbose**: (Optional) Enable verbose log messages
  - **terraform-report**: (Default: `./terraform.json`) Filepath to store generated Terraform report
  - **tf-dir**: (Default `""`) The directory where the terraform main entry files are found.
  - **tf-plan**: (Optional) Filepath to Terraform Plan file.
  - **tf-var-file**: (Optional) Filepath to the Terraform var-file
  - **tf-version**: (Default: `0.15.5`)  The terraform version to use to create Terraform plan
  - **tf-policy-file**: (Optional) Filepath to policy profile. This flag can accept an SCC V2 profile or a custom json file with a set of SCC rules. 
  - **tf-attachment-file**: (Optional) Path of SCC V2 attachment file.

##### Implicit
The following inputs are coming from tekton annotation:
 - **PIPELINE_RUN_ID**: ID of the current pipeline run

#### Workspaces

- **artifacts**: The output volume to check out and store task scripts & data between tasks

##### Results

- **status**: Status of cra terraform task, possible value are - success|failure

##### Usage

Example usage in a pipeline.
``` yaml
    - name: cra-terraform-scan
      taskRef:
        name: cra-terraform-scan-v2
      workspaces:
        - name: artifacts
          workspace: artifacts
      params:
        - name: ibmcloud-api
          value: $(params.ibmcloud-api)
        - name: ibmcloud-region
          value: $(params.ibmcloud-region)
        - name: pipeline-debug
          value: $(params.pipeline-debug)
        - name: resource-group
          value: $(params.resource-group)
        - name: custom-script
          value: $(params.custom-script)
        - name: ibmcloud-trace
          value: $(params.ibmcloud-trace)
        - name: output
          value: $(params.output)
        - name: path
          value: $(params.path)
        - name: strict
          value: $(params.strict)
        - name: toolchainid
          value: $(params.toolchainid)
        - name: verbose
          value: $(params.verbose)
        - name: terraform-report
          value: $(params.terraform-report)
        - name: tf-dir
          value: $(params.tf-dir)
        - name: tf-plan
          value: $(params.tf-plan)
        - name: tf-var-file
          value: $(params.tf-var-file)
        - name: tf-policy-file
          value: $(params.tf-policy-file)
        - name: tf-version
          value: $(params.tf-version)
        - name: tf-attachment-file
          value: $(params.tf-attachment-file)
     
```

