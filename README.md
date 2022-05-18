# Python Web App DevOps Lab
This workshop is a step-by-step guide for exploring how adopting DevOps can automate the deployment of a Python Web App to Azure.

This workshop demonstrates how to:
1. [Use Bicep to create a web app in the Azure](#Create-App-Service-app-using-Bicep)
2. [Configure GitHub actions to deploy Azure App Service using Infrastructure as code](#A-tour-of-app-service-features)
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

## Create App Service app using Bicep
Bicep is a domain-specific language (DSL) that uses declarative syntax to deploy Azure resources. It provides concise syntax, reliable type safety, and support for code reuse. You can use Bicep instead of JSON to develop your Azure Resource Manager templates (ARM templates). Bicep syntax reduces that complexity and improves the development experience.

Copy template snippets bit by bit to guide user through each section.

### Task 1
First we need to create the bicep file that will define our infrastructure.

Above the list of files, using the Add file drop-down, click Create new file.

![alt text](/images/create_new_file.png "Create new file")

In the file name field, type the name and extension for the file, in this case 'bicep/main.bicep'.

![alt text](/images/bicep_path.png "file Path")

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

What other properties of the bicep code can you recognise?

When ready commit the new file to main, adding a comment to describe your changes.