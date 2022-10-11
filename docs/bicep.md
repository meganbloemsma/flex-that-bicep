# Bicep
Learning about Azure bicep :muscle: .

# :pushpin: Useful links
[Microsoft Learn](https://learn.microsoft.com) has several learning paths to learn Bicep:

1. [Fundamentals of Bicep](https://learn.microsoft.com/en-us/training/paths/fundamentals-bicep/)
2. [Intermediate Bicep](https://learn.microsoft.com/en-us/training/paths/intermediate-bicep/)
3. [Advanced Bicep](https://learn.microsoft.com/en-us/training/paths/advanced-bicep/)

These include code-along exercises and a sandbox environment.

## General
Bicep is a domain-specific language and built to easy deployment and configuration of Azure resources.

You submit the Bicep template to [Azure Resource Manager](https://learn.microsoft.com/en-us/azure/azure-resource-manager/). This service requires JSON templates but Bicep automagically converts your template into a JSON template when you submit your deployment. This process is called *transpilation*.

Comparing JSON and Bicep [image source](https://learn.microsoft.com/en-us/training/modules/introduction-to-infrastructure-as-code-using-bicep/5-how-bicep-works):
![JSON vs Bicep](https://learn.microsoft.com/en-us/training/modules/introduction-to-infrastructure-as-code-using-bicep/media/bicep-json-comparison-inline.png)

## Bicep keywords
**Parameters**
Lets you bring in values from outside the template file.

A *parameter file* lists all the parameters and values for the deployment. If the template is deployed from an automated process like a deployment pipeline, the pipeline can provide parameter values - automagically filling it in during deployment.

**Variables**
Are defined and set within the template. It lets you store information in one place and refer to it throughout the template.

**Expressions**
You often don't want to hard-code values, or even ask for them to be specified in a parameter. Instead, you want to discover values when the template runs.

***Example:*** 
When you're writing and deploying a template, you often don't want to have to specify the location of every resource individually. Instead, you might have a simple business rule that says, "By default, deploy all resources into the same location the resource group was created in."

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

**Modules**
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

**Outputs**
Outputs can be used when you need to get information from the template deployment.

*Don't create outputs for secret values. Anyone with access to your resource group can read outputs from templates.*

## When to use Bicep
The main advantage is that it's **Azure-native**:
- Bicep will support new releases or updates of Azure resources on day 1
- Fully integrated into Azure platform

Furthermore there is **no state management**. You don't need to keep your resource state information: Azure automagically keeps track of this state for you.

If you're already using ARM JSON templates you can use the Bicep CLI to **decompile any ARM template into a Bicep template** using the command

    bicep decompile


### Terraform vs Bicep
f