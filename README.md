# Python Web App DevOps Lab
This workshop is a step-by-step guide for exploring how adopting DevOps can automate the deployment of a Python Web App to Azure.

This workshop demonstrates how to:
1. [Use Bicep to create a web app in the Azure](#use-bicep-to-create-a-web-app-in-the-azure)
2. [Deploy to App Service using GitHub Actions](#deploy-to-app-service-using-github-actions)
3. [Create a sample app code (in Python)](#Creating-and-building-the-demo-app)
4. [Deploy to your web app on Azure](#Deploy-the-Express-application-to-an-Azure-Web-App)
5. [Validate it is working and inspect the deployed web app](#Check-that-your-application-has-deployed-correctly)
6. [How to inject variables or secrets into your web app](#Injecting-variables-and-secrets-into-a-web-app)
7. [How to use key vault to store a secret that the web app then uses](#Using-Azure-Key-Vault-to-hold-secrets)

## Prerequisites
1. Access to an Azure subscription or resource group with contributor rights. Will be provided by your coach.
2. Clone of this repository
2. Visual Studio Code with the *Azure Tools* extension installed (this extension is published by Microsoft)

If you're in a hurry the completed resources for this lab can be found in the 'completed' folder within this repo.

## Use Bicep to create a web app in the Azure
Bicep is a domain-specific language (DSL) that uses declarative syntax to deploy Azure resources. It provides concise syntax, reliable type safety, and support for code reuse. You can use Bicep instead of JSON to develop your Azure Resource Manager templates (ARM templates). Bicep syntax reduces that complexity and improves the development experience.

Copy template snippets bit by bit to guide user through each section.

### Task 1 - Create the Bicep file
First we need to create the bicep file that will define our infrastructure.

Above the list of files, using the Add file drop-down, click Create new file.

![alt text](/images/create_new_file.png "Create new file")

In the file name field, type the name and extension for the file, in this case 'bicep/main.bicep'.

![alt text](/images/bicep_path.png "file Path")

### Task 2 - Define the infrastructure using Bicep
Leaving the window open copy the following bicep code into the body of the file.

```
param webAppName string = uniqueString(resourceGroup().id) // Generate unique String for web app name
param sku string = 'S1' // The SKU of App Service Plan
param linuxFxVersion string = 'PYTHON|3.9' // The runtime stack of web app
param location string = resourceGroup().location // Location for all resources

var appServicePlanName = toLower('AppServicePlan-${webAppName}')
var webSiteName = toLower('wapp-${webAppName}')

resource appServicePlan 'Microsoft.Web/serverfarms@2020-06-01' = {
  name: appServicePlanName
  location: location
  sku: {
    name: sku
  }
  kind: 'linux'
  properties: {
    reserved: true
  }
}

resource appService 'Microsoft.Web/sites@2020-06-01' = {
  name: webSiteName
  location: location
  kind: 'app'
  properties: {
    serverFarmId: appServicePlan.id
    siteConfig: {
      linuxFxVersion: linuxFxVersion
    }
  }
}
```

The code above declares two resources:
1. An App Service plan to define a set of compute resources for a web app to run. This is identified by the 'resource appServicePlan' block.
2. An App Service which is a HTTP-based service for hosting web applications, REST APIs, and mobile back ends. This is where code will be deployed and can be identified by the 'resource appService' block.

You'll notice that parameters are used to define some properties, some of which are generated from the resource group values.

Can you identify where the runtime stack could be changed?

### Task 3 - Commit the Bicep file
When ready commit the new file to main, adding a comment to describe your changes.

## Deploy to App Service using GitHub Actions
In this section we will use GitHub Actions and the bicep code to automate the deployment of the Azure infrastructure we need to run our python application.

GitHub actions use a workflow file which is defined in YAML (.yml) and stored within the /.github/workflows/ path in your repository. This definition contains the various steps and parameters that make up the workflow.

The file has three sections:

Authentication
1. Define a service principal or publish profile.
2. Create a GitHub secret.

Build
1. Set up the environment.
2. Build the web app.

Deploy
1. Deploy the web app.

### Task 1 - Generate credentials
First step is to generate credentials to allow GitHub actions  access to your Azure Subsription. In this lab your coach will provide a service principal with the required access. In the real world access may have to be requested via the security or platform team.

Please now ensure you have the service principal details from your coach. You will need these for the following tasks.

### Task 2 - Add secrets to GitHub
In GitHub, browse your repository, select Settings > Secrets > Actions > New respository secret.

Paste the entire JSON output from the Azure CLI command (provided by your coach) into the secret's value field. Give the secret the name AZURE_CREDENTIALS.

Create another secret named AZURE_RG. Add the name of your resource group to the secret's value field. Your coach will provide the value to use here.

Create another secret named AZURE_SUBSCRIPTION. Add your subscription ID to the secret's value field. Again, use the value provided by your coach.

These values will be used later in the workflow to gain access to Azure.

### Task 3 - Create a workflow

Next task is to create a workflow to deploy the Azure infrastructure.

A workflow defines the steps to execute when triggered. It's a YAML (.yml) file in the .github/workflows/ path of your repository. The workflow file extension can be either .yml or .yaml.

To create a workflow, take the following steps:

1. From your GitHub repository, select Actions from the top menu.

2. Select "Skip this and set up a workflow yourself" You will notice Github provides a basic workflow to help you out. 

3. Replace the content of the yml file with the following code:

```
on: [push]
name: Azure ARM
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:

      # Checkout code
    - uses: actions/checkout@main

      # Log into Azure
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

      # Deploy Bicep file
    - name: deploy
      uses: azure/arm-deploy@v1
      with:
        subscriptionId: ${{ secrets.AZURE_SUBSCRIPTION }}
        resourceGroupName: ${{ secrets.AZURE_RG }}
        template: ./bicep/main.bicep
        failOnStdErr: false
```

The first section of the workflow file includes:
- on: The name of the GitHub events that triggers the workflow. The workflow is triggered when there's a push event on the main branch.
    Select Start commit.
- name: The name of the workflow.

To finalise your changes complete the following:

6. Select Commit directly to the main branch.

7. Select Commit new file (or Commit changes).

Note: Updating either the workflow file or Bicep file triggers the workflow. The workflow starts right after you commit the changes.