# Python Web App and GitHub Actions Lab
This lab is a step-by-step beginners guide for exploring how adopting GithHub Actions can automate the deployment of a Python web app to Azure.

This lab demonstrates how to:
1. [Use Bicep to define the required Azure infrastructure](#use-bicep-to-define-the-required-azure-infrastructure)
2. [Deploy the infrastructure using GitHub Actions](#deploy-the-infrastructure-using-github-actions)
3. [Check workflow and deployment status](#check-workflow-and-deployment-status)
4. [Deploy a sample Python app to Azure](#deploy-a-sample-python-app-to-azure)
5. [Check the application has deployed correctly](#check-the-application-has-deployed-correctly)

## Prerequisites

1. A GitHub account - Sign up here: https://github.com/join

1. Service principal granting contributor access to an Azure subscription or resource group. You will be provided with these details if this lab is part of a coached session. 

2. A fork of this repository - Use the fork button located in the top right of this page. Ensure the owner is set to your GitHub username, the repository name can be left as is. 

![alt text](/images/fork_button.jpg "Fork Button")

This lab is designed for beginners and simplicity. All tasks can be carried out online at https://github.com/

## Use Bicep to define the required Azure infrastructure
Bicep is a domain-specific language (DSL) that uses declarative syntax to deploy Azure resources. It provides concise syntax, reliable type safety, and support for code reuse. You can use Bicep instead of JSON to develop your Azure Resource Manager templates (ARM templates). Bicep syntax reduces that complexity and improves the development experience.

### Task 1 - Create the Bicep file
First task is to create a bicep file to contain our code.

Above the list of files on this page, use the Add file drop-down and click Create new file.

![alt text](/images/create_new_file.png "Create new file")

In the file name field, type the name and extension for the file, in this case 'bicep/main.bicep'.

![alt text](/images/bicep_path.png "file Path")

Leave the page open ready for the next task.

### Task 2 - Define the infrastructure using Bicep
Next you need to declare the required infrastructure. Copy and paste the code below into the body of the 'main.bicep' file you just created.

```
param webAppName string = uniqueString('YOURNAME') // Generate unique String for web app name
param sku string = 'S1' // The SKU of App Service Plan
param linuxFxVersion string = 'PYTHON|3.9' // The runtime stack of web app
param location string = resourceGroup().location // Location for all resources

var appServicePlanName = toLower('asp-${webAppName}')
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

output hostname string = appService.name

```

The code above declares two resources:
1. An App Service plan to define a set of compute resources for a web app to run. This is identified by the 'resource appServicePlan' block.
2. An App Service which is a HTTP-based service for hosting web applications, REST APIs, and mobile back ends. This is where code will be deployed and can be identified by the 'resource appService' block.

You'll notice that parameters are used to define some properties, some of which are generated from resource group values. It's generally a good idea to generate unique names for resources too avoid conflict with other deployments. Locate the line shown below and replace YOURNAME to generate a unique string based on your name.

```
param webAppName string = uniqueString('YOURNAME')
```

Question: During that change, did you also stumble across where the runtime stack could be changed?

### Task 3 - Commit the Bicep file
When ready, commit the new file to main, adding a comment to describe your changes.

## Deploy the infrastructure using GitHub Actions
In this section we will use GitHub Actions to automate the deployment of the Azure infrastructure defined in the previous section.

GitHub actions uses a definition file containing the various steps and parameters needed to automate your workflow.

### Task 1 - Generate credentials
First step is to generate credentials to allow GitHub actions  access to your Azure Subscription. In this lab your coach will provide a service principal with the required access. In the real world access may have to be requested via the security or platform team.

Please now ensure you have the service principal details from your coach. You will need these for the following tasks.

### Task 2 - Add secrets to GitHub
In GitHub, browse your repository, select Settings > Secrets > Actions > New repository secret.

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
name: Python application

on:
  [push]

env:
  DEPLOYMENT_NAME: ${{ github.run_id }}-${{ github.run_number }}

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:

      # Checkout code
    - uses: actions/checkout@v2
    
      # Log into Azure
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

      # Deploy Bicep file
    - name: Deploy Bicep
      uses: azure/arm-deploy@v1
      id: deploy
      with:
        subscriptionId: ${{ secrets.AZURE_SUBSCRIPTION }}
        resourceGroupName: ${{ secrets.AZURE_RG }}
        template: ./bicep/main.bicep
        failOnStdErr: false
        deploymentName: ${{ env.DEPLOYMENT_NAME }}
    - run: echo ${{ steps.deploy.outputs.hostName }}
```

The workflow file includes 'on: [Push]' which means the workflow is triggered when there's a push event on the main branch. Below this 'jobs' are defined for each required task.

To finalise your changes complete the following:

6. Scroll to the bottom and under 'Commit Changes' ensure 'Commit directly to the main branch' is selected.

7. Select Commit new file (or Commit changes) adding a commit to describe the change.

Note: Updating either the workflow file or Bicep file triggers the workflow. The workflow starts right after you commit the changes.

## Check workflow and deployment status
By viewing the status of our workflow you can identify success or failure and debug any issues should they occur.

To do this select the 'Actions' tab in your repository. 

![alt text](/images/actions-tab.png "Actions Tab")

Under 'All Workflows' you'll see the current status of the workflow we just created. Something like the below screenshot (Your names will differ):

![alt text](/images/runs.png "Actions Tab")

From the list of workflow runs, click the name of the most recent run to see the workflow run summary.

Once the status is green, click 'build-and-deploy' to list the jobs and then expand the 'Run echo' job as below.

![alt text](/images/hostname.png "Jobs")

In this job we run the echo command to display the output defined in our Bicep file called 'hostName'. This output shows the autogenerated name of the App Service so we can check the live status of the web app.

To get the URL you need replace YOUR_WEB_APP_NAME with the value after the highlighted echo command.
```
https://YOUR_WEB_APP_NAME.azurewebsites.net
```
For example, in the screenshot above the value is wapp-vuqhwdvcgkkk2, which means the URL would be:

https://wapp-vuqhwdvcgkkk2.azurewebsites.net

Copy and paste your completed URL into your browser to view your web app. Ensure you also save the URL somewhere, as you will need it later.

If all the steps have been completed correctly, you'll notice a default landing page confirming the app is up and running but missing your content.

![alt text](/images/default_landing_page.png "Jobs")

You have successfully deployed app service using GitHub Actions! but there's more to do... 

## Deploy a sample Python app to Azure
Now our infrastructure is in place we can proceed to deploy our application code. In this lab we will use a simple python web app for demonstration purposes. However, in a microservices architecture, this could be one of many APIs interconnecting to form an application.

To deploy our Python application we need to add a couple of additional steps to the workflow.

The first is to build the web app. The process of building a web app and deploying to Azure App Service changes depending on the language. In Python, the file 'Requirements.txt' is used to declare the required packages needed for the app to run successfully, we then use a job within our workflow to install these packages. This is shown below:

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r ./app/requirements.txt

Second is to deploy the code to the app service we deployed earlier. For this we need the name of the app service and the path to the Python application project.

    - name: Deploy web app using GH action azure/webapps-deploy
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ steps.deploy.outputs.hostName }}
        package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}

We fetch the app service name from the deploy Bicep job output (the same output we used to construct our URL) and then set the package path using an environment variable.

### Task 1 - Expand workflow
In GitHub browse to '.github/workflows' and open 'main.yml'. Replace all the code with the contents below.

```
name: Python application

on:
  [push]

env:
  AZURE_WEBAPP_PACKAGE_PATH: './app' # set this to the path to your web app project, defaults to the repository root
  DEPLOYMENT_NAME: ${{ github.run_id }}-${{ github.run_number }}

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:

      # Checkout code
    - uses: actions/checkout@v2
    
      # Log into Azure
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    
      # Build the Python app
    - name: Set up Python 3.x
      uses: actions/setup-python@v2
      with:
        python-version: 3.x
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r ./app/requirements.txt

      # Deploy Bicep file
    - name: Deploy Bicep
      uses: azure/arm-deploy@v1
      id: deploy
      with:
        subscriptionId: ${{ secrets.AZURE_SUBSCRIPTION }}
        resourceGroupName: ${{ secrets.AZURE_RG }}
        template: ./bicep/main.bicep
        failOnStdErr: false
        deploymentName: ${{ env.DEPLOYMENT_NAME }}
    - run: echo ${{ steps.deploy.outputs.hostName }}

      # Deploy Web App
    - name: Deploy web app using GH action azure/webapps-deploy
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ steps.deploy.outputs.hostName }}
        package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}
    - name: logout
      run: |
        az logout
```

Question: Where would you enter the path to the application should it change?

When you're ready commit your changes just as before. This will trigger a new run where your Python app will be deployed.

## Check the application has deployed correctly

Just as we did in the 'Check Workflow and Web App Status' section, you should now check the status of your run and review any errors. Feel free to do that now and see if you can also identify the extra jobs we added.

Once the run has completed and has a green tick, browse to your web app URL (Remember the one you constructed earlier?). You should be greeted by the page shown below:

![alt text](/images/hello_page.png "Jobs")

If you see the page above you have successfully deployed your Python web application to Azure! Well done!

Everytime you make a change to your web application, those changes will be automatically deployed using GitHub Actions!

## Summary
In this lab we created a simple GitHub Actions workflow to automate the deployment of Azure App Service and a sample Python web app.

The aim was to quickly demonstrate how automated deployments can deliver speed and reliability, allowing developers to focus on product development.

In more advanced scenarios workflows may include increased governance such as pull request triggers, automated testing and release approvals to protect the integrity of the application.

## Resources

[Quickstart: Deploy Bicep files by using GitHub Actions](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/deploy-github-actions?tabs=CLI)

[Deploy to App Service using GitHub Actions](https://docs.microsoft.com/en-us/azure/app-service/deploy-github-actions?tabs=applevel)

[Quickstart: Deploy a Python (Django or Flask) web app to Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/quickstart-python?tabs=flask%2Cwindows%2Cazure-portal%2Cterminal-bash%2Cvscode-deploy%2Cdeploy-instructions-azportal%2Cdeploy-instructions-zip-azcli)

[Deploying Python to Azure App Service](https://docs.github.com/en/actions/deployment/deploying-to-your-cloud-provider/deploying-to-azure/deploying-python-to-azure-app-service)
