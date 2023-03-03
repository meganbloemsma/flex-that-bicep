# What is Azure Bicep

:muscle:

## Table of Contents

1. :pushpin: [Useful links](https://github.com/meganbloemsma/flex-that-bicep/blob/main/docs/bicep.md#pushpin-useful-links)
2. :paperclip:[General](https://github.com/meganbloemsma/flex-that-bicep/blob/main/docs/bicep.md#general)
3. :grey_question:[When to use Bicep](https://github.com/meganbloemsma/flex-that-bicep/blob/main/docs/bicep.md#question-when-to-use-bicep), including [Bicep vs Terraform](https://github.com/meganbloemsma/flex-that-bicep/blob/main/docs/bicep.md#terraform-vs-bicep)
4. :clipboard:[Bicep keywords](https://github.com/meganbloemsma/flex-that-bicep/blob/main/docs/bicep.md#bicep-keywords)

## :pushpin: Useful links

[Microsoft Learn](https://learn.microsoft.com) has several learning paths to learn Bicep:

1. [Fundamentals of Bicep](https://learn.microsoft.com/en-us/training/paths/fundamentals-bicep/)
2. [Intermediate Bicep](https://learn.microsoft.com/en-us/training/paths/intermediate-bicep/)
3. [Advanced Bicep](https://learn.microsoft.com/en-us/training/paths/advanced-bicep/)

These include code-along exercises and a sandbox environment. The src folder contains my code for these exercises.

:tv: If you're more of a video learning gal, you can take a look at [this Pluralsight course (2021)](https://app.pluralsight.com/library/courses/deploying-azure-resources-using-bicep) or this [youtube video (2023)](https://www.youtube.com/watch?v=VDCAJIGqHZU).

## :paperclip: General

Bicep is a domain-specific language used for deployment and configuration of Azure resources. It's an infrastructure-as-code tool for Azure resources. The most popular alternative is Terraform.

You submit the Bicep template to [Azure Resource Manager](https://learn.microsoft.com/en-us/azure/azure-resource-manager/). This service requires JSON templates but Bicep automagically converts your code into a JSON template when you submit your deployment. This process is called *transpilation*.

Here you can view the difference between the bicep code, and the automagically transpilated JSON [(image source)](https://learn.microsoft.com/en-us/training/modules/introduction-to-infrastructure-as-code-using-bicep/5-how-bicep-works):
![JSON vs Bicep](https://learn.microsoft.com/en-us/training/modules/introduction-to-infrastructure-as-code-using-bicep/media/bicep-json-comparison-inline.png)

## :question: When to use Bicep

The main advantage is that it's **Azure-native**:

- Bicep will support new releases or updates of Azure resources on day 1
- Fully integrated into Azure platform

Furthermore there is **no state management**. You don't need to keep your resource state information: Azure automagically keeps track of this state for you.

If you're already using ARM JSON templates you can use the Bicep CLI to **decompile any ARM template into a Bicep template** using the command

    bicep decompile

## Bicep vs Terraform

The most popular alternative to bicep is Hashicorp's [Terraform](https://developer.hashicorp.com/terraform/docs).

It's also an infrastructure-as-code tool that allows you to configure and deploy Azure resources.

The main difference is that Terraform is not Azure-native. It is able to work with Azure (using, surprise surprise, [Terraform on Azure](https://learn.microsoft.com/en-us/azure/developer/terraform/overview)) and other providers. A full list of these providers can be found [here](https://registry.terraform.io/).

There is no answer to the question 'which is better', because it depends on your needs and existing infrastructure.

There are many sources that compare the differences in more detail, such as:

- [MSFT doc: comparing Terraform and Bicep](https://learn.microsoft.com/en-us/azure/developer/terraform/comparing-terraform-and-bicep?tabs=comparing-bicep-terraform-integration-features)
- [Techtarget: compare ARM template vs Terraform for IaC](https://www.techtarget.com/searchitoperations/tip/Compare-ARM-templates-vs-Terraform-for-infrastructure-as-code)

:tv: A video explanation of the differences in detail can be found in this [Pluralsight video](https://app.pluralsight.com/library/courses/arm-templates-terraform-it-ops-sessions).

Bing away if you want even more resources!

## :clipboard: Bicep keywords

### Parameters

Let you bring in values from outside the template file.

A *parameter file* lists all the parameters and values for the deployment. If the template is deployed from an automated process like a deployment pipeline, the pipeline can provide parameter values - automagically filling it in during deployment.

**Decorators** attach constraints and metadata to a parameter, e.g. a list of allowed values.

Example:

    @minLength(2)
    @maxLength(64)
    description ('The name of vnet.')
    param vnetName string

Details about parameters and decorators can be found [here](https://learn.microsoft.com/en-gb/training/modules/build-reusable-bicep-templates-parameters/2-understand-parameters).

### Expressions

You often don't want to hard-code values, or even ask for them to be specified in a parameter. Instead, you want to discover values when the template runs.

**Example:**

Instead of specifying the resource location individually, you might have a simple business rule that says, "By default, deploy all resources into the same location the resource group was created in."

In Bicep, you can create a parameter called location, then use an expression to set its value:

    var appServicePlanName = 'toy-product-launch-plan'

    param location string = resourceGroup().location

    resource appServiceApp 'Microsoft.Web/sites@2021-03-01' = {
        name: appServiceAppName
        location: location
        properties: {
            serverFarmId: appServicePlan.id
            httpsOnly: true
        }
    }

Another example using an expression is automatically creating an unique name for a resource based on a particular naming strategy your company uses.

    param storageAccountName string = uniqueString(resourceGroup().id)

### Modules

Individual Bicep files for different parts of your deployment. The main Bicep file can reference these modules. You can have a single Bicep module that lots of Bicep templates use.

![Bicep modules](https://learn.microsoft.com/en-gb/training/modules/includes/media/bicep-templates-modules.png)

Behind the scenes, modules are transpiled into a single JSON template for deployment.

When you want the template to include a reference to a module, use the module keyword:

    module myModule 'modules/mymodule.bicep' = {
        name: 'MyModule'
        params: {
            location: location
        }
    }

### Outputs

Outputs can be used when you need to get information from the template deployment.

:exclamation:*Don't create outputs for secret values. Anyone with access to your resource group can read outputs from templates.*

### Conditions

Deploy resources only when specific constraints are in place.

e.g. When you deploy to a production environment, you need to ensure auditing is enable for your SQL server. In the dev environments you don't want to enable auditing. You can configure this in a single template to deploy resources to all your environments.

**Example**, where code deploys a SQL auditing resource only when the environmentName parameter equals 'Production':

    @allowed([
        'Development'
        'Production'
        ])
    param environmentName string

    var auditingEnabled = environmentName == 'Production'

    resource auditingSettings 'Microsoft.Sql/servers/auditingSettings@X' = if (auditingEnabled) {
        parent: server
        name: 'default'
        properties: {
        }
    }

### Loops

Deploy multiple resources that have similar properties.

You can use the *for* keyword to create a loop. Place it in the resource declaration, and then specify how you want Bicep to identify each item in the loop.

Example:
    param storageAccountNames array = [
        'saauditus'
        'saauditeurope'
        'saauditapac'
    ]

    resource storageAccountResources 'Microsoft.Storage/storageAccounts@2021-09-01' = [for storageAccountName in storageAccountNames: {
        name: storageAccountName
        location: resourceGroup().location
        kind: 'StorageV2'
        sku: {
            name: 'Standard_LRS'
        }
    }]

If you want to loop to create a specific number of resources, use the *range()* function. This creates an array of numbers.

Example, where you need to create 4 storage accounts (sa1 to sa4):

    resource storageAccountResources 'Microsoft.Storage/storageAccounts@2021-09-01' = [for i in range(1,4): {
        name: 'sa${i}'
        location: resourceGroup().location
        kind: 'StorageV2'
        sku: {
            name: 'Standard_LRS'
        }
    }]

**Example** using conditions and loops: ['main-logical-exercise.bicep' file](https://github.com/meganbloemsma/flex-that-bicep/blob/main/src/fundamentals/main-logical-exercise.bicep).

### Run your bicep code

By running your main file. This main file should loop through all the modules and subfolders containing bicep code:

    bicep build main.bicep