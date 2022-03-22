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

* **Azure_StorageAccount_Name** in the value field ensure you use an all lowercase value to confirm with Azure Storage Account naming. 

Now head over to Projects > Azure Bicep templates > Runbooks

