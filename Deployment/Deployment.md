# Deployment Guide 
Please follow the steps below to set up the Azure environment


## Step 1: Download Files
Clone or download this repository and navigate to the project's root directory.

## Step 2: Security Access
### Step 2.1: Add Azure Synapse to Azure Key Vault 
1. Go to the Key Vault that was created in the previous step 
2. Click `Access policies`, click `+ Create`, under "Secret permissions" select `Get` and `List` and click `Next`
3. Search for your Synapse Workspace name to be added to the Key vault and click `Next` 
4. Select "Review + create" and save the changes made. 

### Step 2.2 Add Secrets to Azure Key Vault
1. Go to the Key vault that was creted in the previous step
2. Select `Secrets` on the left side

![Key Vault](./img/KeyVaultSecrets.png)

3. Select `+ Generate/Import` 
4. Proivde a name for your `adls-service-principal` and the Service Principal Id created during [Prerequisites steps](./Prerequisites.md)
5. Select `+ Generate/Import` 
6. Proivde a name for your `mgdc-service-principal` and the Service Principal Id created during [Prerequisites steps](./Prerequisites.md)


### Step 2.3: Add your IP address to Synapse firewall
Before you can upload assests to the Synapse Workspace you will need to add your IP address:
1. Go to the Synapse resouce you created in the previous step. 
2. Navigate to `Networking` under `Security` on the left hand side of the page.
3. At the top of the screen click `+ Add client IP`
    ![Update Firewalls](./img/deploy-firewall.png)  
4. Your IP address should now be visible in the IP list

### Step 2.4: Update storage account permisions 
In order to perform the necessary actions in Synapse workspace, you will need to grant more access.
1. Go to the Azure Data Lake Storage Account for your Synapse Workspace
2. Go to the `Access Control (IAM) > + Add > Add role assignment` 
3. Now search and select the `Storage Blob Data Contributor` role and click "Next" 
4. Click "+ Select members", search and select your username and your adls-spn created in previous steps and click "Select" 
5. Click `Review and assign` at the bottom

[Learn more](https://docs.microsoft.com/azure/synapse-analytics/security/how-to-set-up-access-control)


## Optional Step 3: Set up Pipelines
> This step is an optional step to connect to your Office 365 and/or Salesforce account. If you are using the sample dataset from this repository, skip to `Step 4`
### Step 3.1: Create Linked Service 
1. Launch the Synapse workspace [Synapse Workspace](https://ms.web.azuresynapse.net/)
2. Select the `subscription` and `workspace` name you are using for this solution accelerator
3. Navigate to the `Manage` Hub, under "External connection" click `Linked services`
4. Click `+ New`, select `Azure Key Vault`, select the subscription you are using for this solution from the "Azure Subscription" dropdown, and select your Azure Key Vault name from the "Azure key vault name" dropdown
5. Click "Test connection" and click "Save"
6. Click `+ New`, select `Office 365`, provide your `Microsoft Graph data connect Data Transfer` Service principal Id and select `Azure Key Vault`
7. Under "AKV Linked Service" select your Azure Key Vault Linked Service from the dropdown, under "Secret name" select your `Microsoft Graph Data Connect secret`, and under "Secret version" select `Latest version`
8. Click "Test connection" and click "Save" 
9. Click `+ New`, select `Azure Data Lake Storage Gen2`, under "Authentication type" select `Service Princlipal`, provide your `ADLS connection` Service principal Id and select `Azure Key Vault`
10. Under "AKV Linked Service" select your Azure Key Vault Linked Service from the dropdown, under "Secret name" select your `ADLS connection secret`, and under "Secret version" select `Latest version`
11. Select the subscription you are using for this solution from the "Azure Subscription" dropdown and select your storage account name from the "Storage account name" dropdown. 
12. Click "Test connection" and click "Save" 
13. Click `+ New`, select `Salesforce`, provide your `username`, `password` and `Security token` for your Salesforce account
16. Click "Test connection" and click "Save" 
17. Publish all changes


### Step 3.2: Update O365 Data Load Messages Pipeline Parameters 

1. Go to the [Azure Portal](portal.azure.com), select the cloud shell in the top right and run the following command to clone the repository 

```
git clone https://github.com/microsoft/Relationship-Mesh-Solution-Accelerator-with-MGDC-and-Azure-Synapse-Analytics.git
```
2. In the following command replace `<Synapse-workspace-name>` with the name of your Synapse workspace you are using for this solution and run the command
```
synapse_name=<Synapse-workspace-name>

cd Relationship-Mesh-Solution-Accelerator-with-MGDC-and-Azure-Synapse-Analytics
```
3. The following commands will create the dataset connections and pipeline for your Salesforce account in your Synapse workspace. Run each command in the cloud shell 
> **Note**: if you are not using a Salesforce account, you can skip this step and move to the Office 365 pipelines

```
sed -i "s/<synapseworkspacename>/$synapse_name/g" Code/Pipelines/SalesforceDataPipeline/dataset/SalesforceAccountADLS.json 

sed -i "s/<synapseworkspacename>/$synapse_name/g" Code/Pipelines/SalesforceDataPipeline/dataset/SalesforceContactADLS.json 

az synapse dataset create --workspace-name $synapse_name --name SalesforceAccount --file @Code/Pipelines/SalesforceDataPipeline/dataset/SalesforceAccount.json

az synapse dataset create --workspace-name $synapse_name --name SalesforceAccountADLS --file @Code/Pipelines/SalesforceDataPipeline/dataset/SalesforceAccountADLS.json

az synapse dataset create --workspace-name $synapse_name --name SalesforceContact --file @Code/Pipelines/SalesforceDataPipeline/dataset/SalesforceContact.json

az synapse dataset create --workspace-name $synapse_name --name SalesforceContactADLS --file @Code/Pipelines/SalesforceDataPipeline/dataset/SalesforceContactADLS.json

az synapse pipeline create --workspace-name $synapse_name --name SalesforceDataPipeline --file @Code/Pipelines/SalesforceDataPipeline/pipeline/SalesforceDataPipeline.json
```
4. In the following command replace `<allowed-group-security-group-id>` with the id of your security group for the users you will pull Office 365 data for that was created in [Step 3 of the Prerequisites](./Prerequisites.md) and run the command 
    > Go to Microsoft Entra ID in the azure portal and search for the security group name for the allowed users group and copy the Object Id
```
allowed_group=<allowed-group-security-group-id>
```
5. The following commands will create the dataset connections and pipeline for your Office 365 account in your Synapse workspace. Run each command in the cloud shell 


```
sed -i "s/<allowed-group-security-group-id>/$allowed_group/g" Code/Pipelines/o365dataloadmessages/pipeline/o365dataloadmessages.json 

az synapse dataset create --workspace-name $synapse_name --name ADLSMessages --file @Code/Pipelines/o365dataloadmessages/dataset/ADLSMessages.json

az synapse dataset create --workspace-name $synapse_name --name Office365Messages --file @Code/Pipelines/o365dataloadmessages/dataset/Office365Messages.json

az synapse pipeline create --workspace-name $synapse_name --name o365dataloadmessages --file @Code/Pipelines/o365dataloadmessages/pipeline/o365dataloadmessages.json

sed -i "s/<allowed-group-security-group-id>/$allowed_group/g" Code/Pipelines/o365dataloadevents/pipeline/o365dataloadevents.json 

az synapse dataset create --workspace-name $synapse_name --name ADLSEvents --file @Code/Pipelines/o365dataloadevents/dataset/ADLSEvents.json

az synapse dataset create --workspace-name $synapse_name --name Office365Events --file @Code/Pipelines/o365dataloadevents/dataset/Office365Events.json

az synapse pipeline create --workspace-name $synapse_name --name o365dataloadevents --file @Code/Pipelines/o365dataloadevents/pipeline/o365dataloadevents.json
```


### Step 3.3: Run the Pipelines 
1. Launch the Synapse workspace [Synapse Workspace](https://ms.web.azuresynapse.net/)
2. Select the `subscription` and `workspace` name you are using for this solution accelerator
3. Select the `Integrate` hub and click on the `SalesforceDataPipeline` pipeline
4. Select `Add trigger` and click `Trigger now`
5. Trigger the `o365dataloadmessages` pipeline and the `o365dataloadevents` pipeline
6. Once the pipeline runs have started, ask one of your admins to approve the requests from the [Microsoft 365 Admin Portal](https://portal.office.com/adminportal/home#/Settings/PrivilegedAccess) Privileged access requests, select the request, and click "Approve" 



# Step 4: Upload Sample Dataset
> **Note**: If you are using your own Office 365 and/or Salesforce account, you can skip this step and move to `step 5`

1. Launch the Synapse workspace [Synapse Workspace](https://ms.web.azuresynapse.net/)
2. Select the `subscription` and `workspace` name you are using for this solution accelerator
3. In Synapse Studio, navigate to the `Data` Hub
4. Select `Linked`
5. Under the category `Azure Data Lake Storage Gen2` you'll see an item with a name like `xxxxx(xxxxx- Primary)`
6. Select the container named `relmeshadlsfs (Primary)`, select "New folder", enter `salesforcedata` and select "Create"
    * In the `salesforcedata` folder, select "New folder", enter `account` and select "Create"
        * In the `account` folder, select `Upload` to upload the following sample datasets in the [account folder](https://github.com/microsoft/Relationship-Mesh-Solution-Accelerator-with-MGDC-and-Azure-Synapse-Analytics/main/Data/account) of this repository to the `account` folder you created: 
            * `accounts.csv`
   * In the `salesforcedata` folder, select "New folder", enter `contact` and select "Create"
        * In the `contact` folder, select `Upload` to upload the following sample datasets in the [contact folder](https://github.com/microsoft/Relationship-Mesh-Solution-Accelerator-with-MGDC-and-Azure-Synapse-Analytics/main/Data/contact) of this repository to the `contact` folder you created: 
            * `contacts.csv`
7. Select the container named `relmeshadlsfs (Primary)`, select "New folder", enter `o365data` and select "Create"
    * In the `o365data` folder, select "New folder", enter `events` and select "Create"
        * In the `events` folder, select `Upload` to upload the following sample datasets in the [events folder](https://github.com/microsoft/Relationship-Mesh-Solution-Accelerator-with-MGDC-and-Azure-Synapse-Analytics/main/Data/o365data/events) of this repository to the `events` folder you created: 
            * `events.json`
            * `other_events.json`
   * In the `o365data` folder, select "New folder", enter `messages` and select "Create"
        * In the `messages` folder, select `Upload` to upload the following sample datasets in the [messages folder](https://github.com/microsoft/Relationship-Mesh-Solution-Accelerator-with-MGDC-and-Azure-Synapse-Analytics/main/Data/o365data/messages) of this repository to the `messages` folder you created: 
            * `messages.json`
            * `other_messages.json`
8. Select the container named `relmeshadlsfs (Primary)`, select "New folder", enter `column_mappings` and select "Create"
    * In the `column_mappings` folder select `Upload` to upload the following sample datasets in the [column_mappings folder](https://github.com/microsoft/Relationship-Mesh-Solution-Accelerator-with-MGDC-and-Azure-Synapse-Analytics/main/Data/column_mappings) of this repository to the `column_mappings` folder you created: 
        * `Account_Column_Mapping_SF.csv`
        * `Contact_Column_Mapping_SF.csv`



# Step 5: Upload Assets and Run Noteboks
1. Launch the Synapse workspace [Synapse Workspace](https://ms.web.azuresynapse.net/)
2. Go to `Develop`, click the `+`, and click `Import` to select all notebooks from this repository's [folder](https://github.com/microsoft/Relationship-Mesh-Solution-Accelerator-with-MGDC-and-Azure-Synapse-Analytics/main/Code/Notebooks)
3. For each of the notebooks, select `Attach to > spark1` in the top dropdown
4. Configure the parameters in the following 6 notebooks and publish the changes
    * `ProcessAllData.ipynb`
    * `ProcessCalendarData.ipynb`
    * `ProcessNetgraphData.ipynb`
    * `ProcessO365Data_Events.ipynb`
    * `ProcessO365Data_Messages.ipynb`
    * `ProcessSalesforcedata.ipynb`

5. Update the following parameters in `ProcessAllData.ipynb`, attach a Spark pool to the notebook and publish the changes 
    ```
    data_lake_account_name = '' # Synapse Workspace ADLS
    ```
6. Run `ProcessAllData.ipynb`

## Step 6: Power BI Set Up 
1. Open the [Power BI report](https://github.com/microsoft/Relationship-Mesh-Solution-Accelerator-with-MGDC-and-Azure-Synapse-Analytics/main/Deployment/PowerBI/RelationshipMeshSA-Dashboard.pbix) in this repository

2. Click the Transform data dropdown and click Data source settings 

![Power BI](./img/PowerBIDataSource.png)

3. Select the Azure Synapse Workspace connection, select `Change Source...` and provide your SQL Server Database name under Server and click `OK`
    * Navigate to the Synapse Workspace overview page in the Azure Portal, copy the Serverless SQL endpoint
4. Select `Edit Permissions`, under Credentials select `Edit`, sign in to your Microsoft Account, click "OK" and click "Close"
5. Select `Refresh`
