# Demos instructions

The following document describe how to do all the demos presented during the session. 

## Suggested setup to present

> You should have a few things open and ready:

- PowerPoint at the title slide
- Browser Tab: Portal Azure Dashboard/ home
- Browser Tab: Octopus Deploy
- Visual Studio Code, with Azure PowerShell and Azure Bicep installed

---

## Demo 1 - Azure Bicep Storage template

This first demo is introducing the audience to an Azure Bicep template.  In this case we are showing a simple template that will deploy an Azure Storage Account. 

You should walk them through the various lines of code and explain them as you go.  Highlight to them the fact that the template is only 21 lines long. 

Now open up a Terminal inside your Visual Studio Code instance and convert that Bicep template to an Azure ARM template. 

```bash
az bicep build --file .\1storage.bicep
```

This will generate a JSON file, if you right click on that file within Visual Studio Code and select "Open to the side" it will open with half the screen showing the Bicep template and the other the ARM.  

Walk the audience through the amount of lines in each and how one appears to be easier to understand than the other. 

We now want to deploy that Azure Bicep template. 

Within your terminal type: 

```powershell

New-AzResourceGroup -Name bicepdemo -Location "North Europe"

New-AzResourceGroupDeployment -Name StorageDeployment -resourceGroupName bicepdemo -TemplateFile 1storage.bicep -storageAccountName "octobicep"

```

The above command will create an Azure resource group and then deploy the Azure Bicep template. 

You can now switch to your ADemo 2 - Azure Bicep Storage template with a failurezure Portal and show them the new storage account resource. 

## Demo 2 - Azure Bicep Storage template with a failure

Within this demo we want to build on the first template that we showed the audience, but add a bit more complexity and real world steps to it. 

You should walk them through the various lines of code and explain them as you go.  Highlight the fact we are now introducing allowed values and also tagging to the resource deployment. 

Now open up a Terminal inside your Visual Studio Code instance type the following command: 

```powershell
New-AzResourceGroupDeployment -Name StorageDeploymentv2 -resourceGroupName bicepdemo -TemplateFile 2storage.bicep -storageAccountType "Standrd_Local"
```

This will try to deploy the template but will fail and show the allowed inputs hasn't been met. 

## Demo 3 - Deploy Azure Bicep template from Octopus Deploy

We want to start to introduce to the audience that yes it's great to be able to create and deploy templates from your local machine, however what happens when you want to do that at scale or run at certain times during the day or not.  Or let others in your team do the deployments?

Start to talk about how Octopus Deploy Runbooks can be used to deploy infrastructure.

First explain you can't store Bicep templates into Octopus natively however if you package or zip them then you can.  So we will zip our bicep templates and then upload them to our Octopus server. 

```powershell
## you might want to hide your octopus information from the audience
$OCTOPUSSERVERURL = "https://octopus.example.com"
$OCTOPUSSERVERAPIKEY = "API-INSERTYOURAPIKEYHERE"
$OCTOPUSSPACENAME = "yourspacename"

octo pack --id="StorageTemplate" --format="zip" --version="0.0.1" --basePath="Templates" --overwrite

octo push --package="StorageTemplate.0.0.1.zip" --server="$OCTOPUSSERVERURL" --apiKey="$OCTOPUSSERVERAPIKEY" --space="$OCTOPUSSPACENAME"
```

You can now switch over to Octopus Deploy.  

Take a few minutes to walk people through the portal, show them things like Infrastructure > Accounts and explain how you've connected Octopus and Azure together so Octopus can create/delete/read resources in your Azure environment. 

> 💡 Be mindful not everyone in the audience will be familiar with Octopus Deploy. 

Head over to Library > Packages and show the audience the zip file you uploaded. 

Now head to Library > Variable Sets.  Talk them through variables and add in a new variable: 

* **Azure_StorageAccount_Name** in the value field ensure you use an all lowercase value to conform with Azure Storage Account naming. 

Now head over to Projects > Azure Bicep templates > Runbooks

Create a new Runbook. 

Your first step will be a "Run Azure Script" step.  Within this step walk through the configuration process, and ensuring you select the correct worker pool. 

Within the source code section enter: 

```powershell
New-AzResourceGroup -Name $OctopusParameters["Azure_Environment_ResourceGroup_Name"] -Location $OctopusParameters["Azure_Location_Abbr"]
```

Now create a second step, also a "Run Azure Script" step. Within this step walk through the configuration process, and ensuring you select the correct worker pool. 

Within the source code section enter: 

```powershell
#### Reference the package with the Bicep files
$filePath = $OctopusParameters["Octopus.Action.Package[StorageTemplate].ExtractedPath"]

#### Change Directory to extracted package
cd $filePath

#### Set the deployment name
$today=Get-Date -Format "dd-MM-yyyy"
$deploymentName="StorageTemplate"+"$today"

### Deploy the Bicep template files
New-AzResourceGroupDeployment -Name $deploymentName -ResourceGroupName $OctopusParameters["Azure_Environment_ResourceGroup_Name"] -TemplateFile "2storage.bicep" -storageAccountName $OctopusParameters["Azure_StorageAccount_Name"]
```

Explain to the audience that this is instructing Octopus to extract the zip file, move into the directory where the extracted package is and then execute the deployment of the Bicep template. 

Ensure you associate your zip file package with this step and then click Save. 

You are now ready to run this Runbook.   It should take a few minutes to run.  

> 💡 This might be a good time to ask the audience to ask any questions or to check if any questions are available to answer.  Or it could be a good time for you as a presenter to grab some water and catch your breath. 

Once the Runbook has completed and is successful you can switch over to the Azure Portal and show the created resource for the audience. 

## Demo 4 - Nested templates

We now want to show the audience how you can start to use multiple templates and build up a larger deployment.  We want to show them that Bicep is powerful and easy to understand. 

The nested template we'll be showing is one that deploys three Azure Web Apps, an Azure App Service plan and SQL Server and database. 

Walk the user through through how you can call other templates within a Bicep template by using **module** and referencing the file location. Mention that you could use module repositories to do this or reference local templates. 

Within Visual Studio Code also show the audience the Bicep visualiser tool that shows gives you a diagram of what the template will deploy. 

If you want to deploy this template from your local machine you can issue the following command: 

```powershell
New-AzResourceGroupDeployment -Name OctoPetShop -ResourceGroupName SarahBicepDemos -TemplateFile octopetshop.bicep -planName octoPetASP -planSku S1 -productwebSiteName octopetproduct -shoppingwebSiteName octopetshopping -frontwebSiteName octopetfront -startFWIpAddress 0.0.0.0 -endFWIpAddress 0.0.0.0 -databaseName octopetdb -sqlServerName octopetsql -sqlAdministratorLogin octopetadmin -sqlAdministratorLoginPassword $sqlpassword
```

> 💡 This deployment can take between 2-10minutes so be aware of that. 

Alternatively you can skip running it locally and start to talk to the user about getting the template ready for deployment with Octopus Deploy. 

Once again pack up the templates into a zip file and upload them to Octopus Deploy. 

```powershell
octo pack --id="OctoBicepFiles" --format="zip" --version="0.0.1" --basePath="Nested-OctoPetShop" --overwrite

octo push --package="OctoBicepFiles.0.0.1.zip" --server="$OCTOPUSSERVERURL" --apiKey="$OCTOPUSSERVERAPIKEY" --space="$OCTOPUSSPACENAME"
```

Switch over to the Octopus Deploy instance. 

Head over to Library > Packages and show the audience the zip file you uploaded. 

Now head to Library > Variable Sets. Show them the pre-created library that can help to deploy these resources. 

Now head over to Projects > Azure Bicep templates > Runbooks

Create a new Runbook. 

Your first step will be a "Run Azure Script" step.  Within this step walk through the configuration process, and ensuring you select the correct worker pool. 

Within the source code section enter: 

```powershell
New-AzResourceGroup -Name $OctopusParameters["Azure_Environment_ResourceGroup_Name"] -Location $OctopusParameters["Azure_Location_Abbr"]
```

Now create a second step, also a "Run Azure Script" step. Within this step walk through the configuration process, and ensuring you select the correct worker pool. 

Within the source code section enter: 

```powershell
# Reference the package with the Bicep files
$filePath = $OctopusParameters["Octopus.Action.Package[OctoBicepFiles].ExtractedPath"]

# Change Directory to extracted package
cd $filePath

# Set the deployment name
$today=Get-Date -Format "dd-MM-yyyy"
$deploymentName="OctoPetShopInfra"+"$today"

# Deploy the Bicep template files

New-AzResourceGroupDeployment -Name $deploymentName -ResourceGroupName $OctopusParameters["Azure_Environment_ResourceGroup_Name"] -TemplateFile octopetshop.bicep -planName $planName -planSku $planSku -productwebSiteName $OctopusParameters["ProductService.Name"] -shoppingwebSiteName $OctopusParameters["ShoppingCartService.Name"] -frontwebSiteName $OctopusParameters["WebApp.Name"] -startFWIpAddress $startFWIpAddress -endFWIpAddress $endFWIpAddress -databaseName $OctopusParameters["Database.Name"] -sqlServerName $OctopusParameters["Database.Server"] -sqlAdministratorLogin $OctopusParameters["Database.Admin.Username"] -sqlAdministratorLoginPassword $OctopusParameters["Database.Admin.Password"]
```

> 💡 Explain to the audience that even though you are deploying a more complex template you aren't doing anything different inside Octopus, the process is the same. 

Kick start the Runbook, again this may take 2-10minutes to run so bare that in mind. 