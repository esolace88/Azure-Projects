# Terraform (IaC)

Welcome to the Terraform project! This is a simple project that levarages Terraform within Azure CLI to deploy resource as code. This allows rapid deployment of service while adding an additional layer of version contol. (See Demo folder for a quick view of the end product)

## Getting Started

To get started, you will need an Azure account with CLI access to initiate Terrafom.

-------------

## Step by Step 

1. Either Upload or copy & paste the terraform file into your Azure Home Directory

2. Modify the terraform file
	- replace all entries of <Enter-Your-ResourceGName with your Resource-Group
		- To determine your first resource group enter the following
      ```bash
        az group list --query "[0].name" -o tsv     
      ```
		- Copy and Paste the resource-group name in the following cmd
      ```bash
        sed -i -e 's/<Enter-Your-ResourceGName/<Paste>/' './test.tf'     
      ```
	- replace the following entry <Enter-A-Unique-Name with a unique name, use the following cmd:
     ```bash
        sed -i -e 's/<Enter-A-Unique-Name/<Enter Name>/' './test.tf'     
     ```
3. Initiate Terraform
     ```bash
        terraform init     
     ```
4. Import the scope of the deployment
     ```bash
        terraform plan    
     ```
5. Apply said plan
     ```bash
        terraform apply    
     ```
6. When Prompted type "Yes"
7. Wait until deployment is completed and review results

-------------

If you get the following errror in and your are using Azure CLI web version
	- Simple restart the shell
	- Start again from step 3

```bash                                                    
│ Error: building account: getting authenticated object ID: parsing json result from the Azure CLI: waiting for the Azure CLI: exit status 1: ERROR: Failed to connect to MSI. Please make sure MSI is configured correctly.
│ Get Token request returned: <Response [400]>
│ 
│   with provider["registry.terraform.io/hashicorp/azurerm"],
│   on test.tf line 13, in provider "azurerm":
│   13: provider "azurerm" {
```

