# Azure Shared Access Signature (SAS)

Welcome to the Azure Shared Access Signature (SAS) project! This is a simple project that levarages time-limited access tokens to ensure access to your resources is available only for a specified duration. (See Demo folder for a quick view of the end product)

## Getting Started

To get started, you will need an Azure account. In this demo i've configured SAS via CLI, it can also be deployed via GUI with the Container page.

-------------

## INFO 
In the Step by Step below you will see I use certain options. Below is a quick peek into what certain options perform. 
  **For more information regarding SAS tokens and it's options, Review Microsofts' Page (https://learn.microsoft.com/en-us/cli/azure/storage/blob?view=azure-cli-latest)**

**--as-user**
Indicates that this command return the SAS signed with the user delegation key. The expiry parameter and '--auth-mode login' are required if this argument is specified.
default value: False

**--permissions**
The permissions the SAS grants. Allowed values: (a)dd (c)reate (d)elete (e)xecute (i)set_immutability_policy (m)ove (r)ead (t)ag (w)rite (x)delete_previous_version (y)permanent_delete. Do not use if a stored access policy is referenced with --id that specifies this value. Can be combined.

## Step by Step 

1. Login into Azure
2. Open Azure Terminal
3. Create Variables for Storage account and Container
    --Note Create a Container if one isn't present. Also the code below queries the first item in the list output. 
    --Storage Account Variable:
      ```bash
        $SA = az storage account list --query "[0].name"     
      ```
    --Container Variable:
      ```bash
        $CN = az storage container list --account-name $SA --query "[0].name" -o tsv     
      ```
4. Run the following AZ Storage Blob Generate-SAS CLI:
      ```bash
       az storage blob generate-sas --account-name $SA `                           
        --container-name $CN `       
        --name blob.jpeg `
        --permissions acdrw `
        --expiry 2023-07-04 `
        --auth-mode login `
        --as-user `
        --full-uri
      ```
5. Copy and Past the URL printed
   
