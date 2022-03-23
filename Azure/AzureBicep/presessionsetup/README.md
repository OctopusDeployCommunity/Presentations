# Pre Session Setup

## Required Software

* Visual Studio Code
* Azure PowerShell
* Azure Bicep CLI
* Azure Bicep Visual Studio Code Extension
* Octopus CLI

## Required Tools

* Octopus Deploy server
* Azure subscription

## Visual Studio Code Setup

Ensure that your Visual Studio Code installation has the Azure Bicep extension installed and that you have authenticated to your Azure subscription through the terminal.  As you will be deploying resources through the terminal within Visual Studio Code. 

## Octopus Deploy Setup

* Create an Octopus Deploy Space. 
* Create an Octopus Deploy Project called "Bicep Templates". 
* Create Test and Production environments. 
* Create a link from your Azure subscription to Octopus Deploy. 

### Create Library Variable Set - Azure

Create a new library variable set called Azure. 

Within there create the following variables: 

| Name 	                                    | Value | Scope
--------------------------------------------|---------|-----------------
|  Azure_Environment_ResourceGroup_Name       | #{Octopus.Space.Name}-#{Octopus.Environment.Name}-rg    | 
|  Azure_Location_Abbr | westeurope     | 

### Create Library Variable Set - OctoPetShop Infrastructure

Create a new library variable set called OctoPetShop Infrastructure. 

Within there create the following variables: 

| Name 	                                    | Value | Scope
--------------------------------------------|---------|-----------------
|  Database.Admin.Password       |    | 
|  Database.Admin.Username | octopetshopdba    | 
|  Database.Server | #{Octopus.Space.Name | ToLower | Replace " "}-#{Octopus.Project.Name | ToLower | Replace " "}-#{Octopus.Environment.Name | ToLower | Replace " "}   | 
|  endFWIpAddress | 0.0.0.0   | 
|  planName | octoaspprod   | Production
|  planName | octoasptest   | Test
|  planSku | S1   | 
|  ProductService.Name | #{Global.Environment.Suffix}-#{Octopus.Space.Name | ToLower | Replace " "}-#{Octopus.Project.Name | ToLower | Replace " "}-productservice   | 
|  ShoppingCartService.Name | #{Global.Environment.Suffix}-#{Octopus.Space.Name | ToLower | Replace " "}-#{Octopus.Project.Name | ToLower | Replace " "}-shoppingcartservice   | Prduction
|  startFWIpAddress | 0.0.0.0   | 
|  WebApp.Name | #{Global.Environment.Suffix}-#{Octopus.Space.Name | ToLower | Replace " "}-#{Octopus.Project.Name | ToLower | Replace " "}-web  | 


### Deploy a worker

The default workers don't have the necessary tools installed on them to be able to deploy Azure Bicep templates so you will need to spin up a worker yourself and then link that to the Octopus Deploy space. 

My recommendation would be to deploy a Windows Server with the following tools installed: 

* Azure PowerShell
* Azure CLI
* Azure Bicep CLI

