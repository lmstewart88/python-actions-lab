# Python Web App DevOps Lab
This workshop is a step-by-step guide for exploring how adopting DevOps can automate the deployment of a Python Web App to Azure.

This workshop demonstrates how to:
1. [Use Bicep to create a web app in the Azure](#Create-App-Service-app-using-Bicep)
2. [Configure GitHub actions to deploy Azure App Service using Infrastructure as code](#A-tour-of-app-service-features)
3. [Create a sample app code on your machine (in Node.js)](#Creating-and-building-the-demo-app)
4. [Deploy to your web app on Azure](#Deploy-the-Express-application-to-an-Azure-Web-App)
5. [Validate it is working and inspect the deployed web app](#Check-that-your-application-has-deployed-correctly)
6. [Demonstrate how to use deployment slots for blue/green deployments](#Blue/Green-Deployments-using-Deployment-Slots)
7. [How to inject variables or secrets into your web app](#Injecting-variables-and-secrets-into-a-web-app)
8. [How to use key vault to store a secret that the web app then uses](#Using-Azure-Key-Vault-to-hold-secrets)

## Prerequisites
1. Access to an Azure subscription or resource group with contributor rights. Will be provided by your coach.
2. Visual Studio Code with the *Azure Tools* extension installed (this extension is published by Microsoft)

If you're in a hurry the completed resources for this lab can be found in the 'completed' folder within this repo.

## Create App Service app using Bicep
Bicep is a domain-specific language (DSL) that uses declarative syntax to deploy Azure resources. It provides concise syntax, reliable type safety, and support for code reuse. You can use Bicep instead of JSON to develop your Azure Resource Manager templates (ARM templates). Bicep syntax reduces that complexity and improves the development experience.

Copy template snippets bit by bit to guide user through each section.

### Task 1
First we need to create the bicep file that will define our infrastructure. Go ahead and create a file called 'main.bicep' in a new folder called 'bicep' 
