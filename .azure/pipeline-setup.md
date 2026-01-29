# Azure DevOps CI/CD Pipeline Setup Guide

## Prerequisites
1. **Azure Subscription**: Create a free Azure subscription at https://azure.microsoft.com/free
2. **Azure DevOps Organization**: Create an organization at https://dev.azure.com
3. **GitHub/Azure DevOps Repo**: Push your code to Azure DevOps Repos

## Step 1: Create Azure DevOps Project
1. Go to https://dev.azure.com
2. Create a new project named `KLJobPortal`
3. Import your repository or create a new one

## Step 2: Create Service Principal for CI/CD
Run these commands in Azure CLI:

```bash
# Set variables
SUBSCRIPTION_ID="<your-subscription-id>"
SERVICE_PRINCIPAL_NAME="KLJobPortal-CI-CD"
RESOURCE_GROUP="klrg"

# Create service principal
az ad sp create-for-rbac --name $SERVICE_PRINCIPAL_NAME \
  --role contributor \
  --scopes /subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP

# Save the output (clientId, clientSecret, tenantId)
```

## Step 3: Configure Azure DevOps Service Connection
1. Go to Project Settings → Service Connections
2. Click "New Service Connection" → "Azure Resource Manager"
3. Choose "Service Principal (manual)"
4. Fill in the credentials from Step 2:
   - Service Principal Id (Client ID)
   - Service Principal Key (Client Secret)
   - Tenant ID
5. Name it `AzureServiceConnection`

## Step 4: Create Infrastructure Deployment Pipeline
1. Go to Pipelines → Create Pipeline
2. Select Azure Repos Git
3. Select your repository
4. Choose "Starter pipeline"
5. Replace with contents of `azure-pipelines-infra.yml`
6. Run the pipeline to create Azure resources (App Service, Container Registry)

## Step 5: Create CI/CD Deployment Pipeline
1. Go to Pipelines → Create Pipeline
2. Follow the same process
3. Replace with contents of `azure-pipelines.yml`
4. Configure pipeline variables in Pipeline settings:
   - `REGISTRY_NAME`: Name of your Container Registry (from infra pipeline output)
   - `REGISTRY_USERNAME`: ACR username
   - `REGISTRY_PASSWORD`: ACR password
   - `APP_NAME_BACKEND`: Backend App Service name
   - `APP_NAME_FRONTEND`: Frontend App Service name
   - `RESOURCE_GROUP`: Resource group name

## Step 6: Set up Environment Approvals (Optional)
1. Go to Pipelines → Environments
2. Create environments: `dev`, `staging`, `production`
3. For staging and production, add approval requirements:
   - Click environment → Approvals and checks
   - Add approval requirement

## Step 7: Trigger Pipelines
- Push to `main` branch to trigger both pipelines
- Pull requests will only run CI (build & test)

## Environment Variables
Configure these in Azure DevOps pipeline:
- `RESOURCE_TOKEN`: Random 5-char lowercase string (auto-generated in infra pipeline)
- `RESOURCE_GROUP`: Resource group name (default: `klrg`)
- `LOCATION`: Azure region (default: `eastus`)

## Next Steps
1. Update `application.properties` with database connection strings
2. Configure environment-specific settings
3. Monitor pipeline runs in Azure DevOps
4. Check deployed apps in Azure Portal

## Troubleshooting
- If service connection fails, verify Service Principal has correct RBAC roles
- If ACR push fails, verify credentials in pipeline variables
- Check Azure DevOps logs for detailed error messages
