# Deployment Guide 
Please follow the steps below to set up a demo Office 365 Developer Program account and an Azure environment
> * **Note**: The Office 365 Developer Program is intended to only be used with the sample dataset provided in this solution accelerator. If you are working with your own dataset, please skip this section and move to `Step 3: Set up Microsoft Graph Data Connect`

## Step 1: Set up Office 365 Developer Program account environment 
1. Navigate to  [Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) 
2. Click `Join now` 


## Step 2: Set up Azure environment 
In order to use this solution accelerator with the Office 365 Developer Program, your Azure subscription needs to be on the same tenant. In this step you will create a free Azure subscription. 
### Step 2.1: Azure Subsription
1. Navigate to the [Azure Portal](https://portal.azure.com/)
2. Sign in on with the account you created in Step 1. 
### Step 2.2: Set up Azure Environment 
1. Navigate to the [Azure Portal](https://portal.azure.com/)
2. Go to the subscription you are using for this solution, under "Settings" click `Resource providers`
3. Search for `Microsoft.Synapse` and click "Register" at the top 
4. Navigate to the [Deployment Guide](./Deployment.md) and follow the steps to set up the required resources for this solution 

## Step 3: Set up Microsoft Graph Data Connect
> **Note**: take note of the service principle Ids and Security Group Ids as you will need them in later steps 
1. Follow Exercise 1 of the [Microsoft Graph Data Connect tutorial](https://github.com/microsoftgraph/msgraph-training-dataconnect/blob/master/Lab.md#exercise-1-setup-office-365-tenant-and-enable-microsoft-graph-data-connect)
    * Grand Azure AD users global administrator role
    * Configure Microsoft Graph data connect consent request approver group
    * Enable Microsoft Graph data connect in your Office 365 tenant
        * Repeat these steps to [create a second security group](https://github.com/microsoftgraph/msgraph-training-dataconnect/blob/master/Lab.md#configure-microsoft-graph-data-connect-consent-request-approver-group) for the users you will pull Office 365 data for. 
2. Follow Exercise 2 [Create Azure AD Application](https://github.com/microsoftgraph/msgraph-training-dataconnect/blob/master/Lab.md#create-azure-ad-application)
    * Create Azure AD Application for `Microsoft Graph data connect Data Transfer`
    * Repeat these steps to create a second Azure AD Application for `ADLS connection`
3. Follow the Step in [Deployment.md](./Deployment.md) 

