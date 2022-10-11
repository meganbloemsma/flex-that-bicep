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

### When to use Bicep
The main advantage is that it's **Azure-native**:
- Bicep will support new releases or updates of Azure resources on day 1
- Fully integrated into Azure platform

Furthermore there is **no state management**. You don't need to keep your resource state information: Azure automagically keeps track of this state for you.

If you're already using ARM JSON templates you can use the Bicep CLI to **decompile any ARM template into a Bicep template** using the command

    bicep decompile


#### Terraform vs Bicep
f