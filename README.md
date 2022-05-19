# Python Web App DevOps Lab
This workshop is a step-by-step guide for exploring how adopting GithHub Actions can automate the deployment of a Python web app to Azure.

This workshop demonstrates how to:
1. [Use Bicep to create a web app in the Azure](#use-bicep-to-create-a-web-app-in-the-azure)
2. [Deploy App Service using GitHub Actions](#deploy-app-service-using-github-actions)
3. [Check Workflow and Web App Status](#check-workflow-and-web-app-status)
4. [Deploy a Python App to Azure](#deploy-python-app-to-azure)
5. [Check that your application has deployed correctly](#check-that-your-application-has-deployed-correctly)

## Prerequisites

1. A GitHub account - Sign up here! https://github.com/join

1. Service principal granting contributor access to an Azure subscription or resource group. You will be provided with these details if this lab is part of a coached training session. 

2. A fork of this repository - Use the fork button located in the top right of this page. Ensure the owner is set to your GitHub username, the repository name can be left as is. 

![alt text](/images/fork_button.jpg "Fork Button")

This lab is designed for beginners and simplicity. All tasks can be carried out online at https://github.com/

## Use Bicep to create a web app in the Azure
Bicep is a domain-specific language (DSL) that uses declarative syntax to deploy Azure resources. It provides concise syntax, reliable type safety, and support for code reuse. You can use Bicep instead of JSON to develop your Azure Resource Manager templates (ARM templates). Bicep syntax reduces that complexity and improves the development experience.

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

output hostname string = appService.name

```

The code above declares two resources:
1. An App Service plan to define a set of compute resources for a web app to run. This is identified by the 'resource appServicePlan' block.
2. An App Service which is a HTTP-based service for hosting web applications, REST APIs, and mobile back ends. This is where code will be deployed and can be identified by the 'resource appService' block.

You'll notice that parameters are used to define some properties, some of which are generated from the resource group values.

Can you identify where the runtime stack could be changed?

### Task 3 - Commit the Bicep file
When ready commit the new file to main, adding a comment to describe your changes.

## Deploy Azure App Service using GitHub Actions
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
      id: deploy
      with:
        subscriptionId: ${{ secrets.AZURE_SUBSCRIPTION }}
        resourceGroupName: ${{ secrets.AZURE_RG }}
        template: ./bicep/main.bicep
        failOnStdErr: false
    - run: echo ${{ steps.deploy.outputs.hostName }}
```

The workflow file includes 'on: [Push]' which means the workflow is triggered when there's a push event on the main branch. Below this 'jobs' are defined for each required task.

To finalise your changes complete the following:

6. Scroll to the bottom and under 'Commit Changes' ensure 'Commit directly to the main branch' is selected.

7. Select Commit new file (or Commit changes) adding a commit to describe the change.

Note: Updating either the workflow file or Bicep file triggers the workflow. The workflow starts right after you commit the changes.

## Check workflow and web app status
By viewing the status of our workflow we can identify success or failure and debug any issues should they occur.

To do this select the Actions tab in your repository. 

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

You have successfully deployed app service using GitHub Actions!

## Deploy Python app to Azure
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

We fetch the app service name from the deploy Bicep job output (the same output we used to contruct our URL) and then set the package path using an environment variable.

### Task 1 - Expand workflow
In GitHub browse to '.github/workflows' and open 'main.yml'. Replace all the code with the contents below.

```
name: Python application

on:
  [push]

env:
  AZURE_WEBAPP_PACKAGE_PATH: './app' # set this to the path to your web app project, defaults to the repository root

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

## Check that your application has deployed correctly

Just as we did in the 'Check Workflow and Web App Status' section, you should now check the status of your run and review any errors. Feel free to do that now and see if you can also identify the extra jobs we added.

Once the run has completed and has a green tick, browse to your web app URL (Remember the one you constructed earlier?). You should be greeted by the page shown below:

![alt text](/images/hello_page.png "Jobs")

If you see the page above you have successfully deployed your Python web application to Azure!

