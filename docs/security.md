# :closed_lock_with_key: Security
is important.

# Table of Contents

1. :pushpin: [Useful links](https://github.com/meganbloemsma/flex-that-bicep/blob/main/docs/security.md#pushpin-useful-links)
2. :key: [Managed identity in Bicep](https://github.com/meganbloemsma/flex-that-bicep/blob/main/docs/security.md#managed-identity-in-bicep)
3. :scroll: [Securing your parameters without managed identity](https://github.com/meganbloemsma/flex-that-bicep/blob/main/docs/security.md#securing-your-parameters-without-managed-identity)

# :pushpin: Useful links

- [Securing parameters with Azure KeyVault](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/key-vault-parameter?tabs=azure-cli)
- [Quickstart Azure KeyVault with Bicep](https://learn.microsoft.com/en-us/azure/key-vault/secrets/quick-create-bicep?tabs=CLI)
- [How to set up your key vault and best practices.](https://learn.microsoft.com/en-us/azure/key-vault/secrets/secrets-best-practices?source=recommendations)
- [Basic template for setting up Azure Key Vault and a secret.](https://azure.microsoft.com/en-gb/resources/templates/key-vault-create/)

# :key: Managed identity in bicep

Instead of putting secret values (like passwords) in your Bicep file or parameter file, you can retrieve the values from Key Vault during a deployment.

If you're new to managed identities, I'd recommend [reading this first.](https://learn.microsoft.com/en-us/training/modules/authenticate-apps-with-managed-identities/) 
A shorter read can be found [here.](https://learn.microsoft.com/en-us/training/modules/implement-managed-identities/)

I will be digging into [Azure KeyVault](https://learn.microsoft.com/en-us/azure/key-vault/) (AKV).

## Azure Key Vault
*Start with [setting up your key vault and taking a look at best practices.](https://learn.microsoft.com/en-us/azure/key-vault/secrets/secrets-best-practices?source=recommendations)*

When using a key vault with the Bicep file for a [managed application](https://learn.microsoft.com/en-us/azure/azure-resource-manager/managed-applications/overview), you must grant access to the Appliance Resource Provider service principal. For more information, see [Access Key Vault secret when deploying Azure Managed Applications.](https://learn.microsoft.com/en-us/azure/azure-resource-manager/managed-applications/key-vault-access)

### getSecret function
When a module expects a 'string' parameter with 'secure:true' modifier, you can use the 'getSecret' function to obtain a key vault secret. The value is never exposed because you only reference its key vault ID.

Below example comes from [MSFT docs](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/key-vault-parameter?tabs=azure-cli#use-getsecret-function). First we create a 'sql.bicep' file where we create a SQL server:

    param sqlServerName string
    param adminLogin string

    @secure()
    param adminPassword string

    resource sqlServer 'Microsoft.Sql/servers@2020-11-01-preview' = {
        name: sqlServerName
        location: resourceGroup().location
        properties: {
            administratorLogin: adminLogin
            administratorLoginPassword: adminPassword
            version: '12.0'
        }
    }

The following Bicep file consumes sql.bicep as module. It will:

1. references an existing key vault,
2. calls the 'getSecret' function to retrieve the key vault secret
3. passes the value as parameter to the module.

    param sqlServerName string
    param adminLogin string

    param subscriptionId string
    param kvResourceGroup string
    param kvName string

    resource kv 'Microsoft.KeyVault/vaults@2019-09-01' existing = {
        name: kvName
        scope: resourceGroup(subscriptionId, kvResourceGroup )
    }

    module sql './sql.bicep' = {
        name: 'deploySQL'
        params: {
            sqlServerName: sqlServerName
            adminLogin: adminLogin
            adminPassword: kv.getSecret('vmAdminPassword')
        }
    }

### Reference to key vault directly
**Alternatively,** if you don't want to use a module you can reference the key vault directly in the parameter file.




### Folder structure
Multiple bicep files should be created when using AKV.

f

# :scroll: Securing your parameters without managed identity
Protecting sensitive values, like passwords and API keys, in your deployments is important. *The best approach is to avoid credentials entirely with [managed identities](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview).*

If you don't want to use managed identities, you can use the **@secure decorator**. This can be applied to string and object parameters and Azure won't make the parameter values available in the deployment logs.

Example:

    @secure()
    param sqlServerAdministratorLogin string

    @secure()
    param sqlServerAdministratorPassword string

Also consider integrating with a key vault service like [Azure Key Vault](https://learn.microsoft.com/en-us/azure/key-vault/).