# :closed_lock_with_key: Security
is important.

# Table of Contents

1. :pushpin: [Useful links]()
2. 

# :pushpin: Useful links
f

# Securing your parameters
Protecting sensitive values, like passwords and API keys, in your deployments is important. *The best approach is to avoid credentials entirely with [managed identities](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview).*

If you don't want to use managed identities, you can use the **@secure decorator**. This can be applied to string and object parameters and Azure won't make the parameter values available in the deployment logs.

Example:

    @secure()
    param sqlServerAdministratorLogin string

    @secure()
    param sqlServerAdministratorPassword string

Also consider integrating with a key vault service like [Azure Key Vault](https://learn.microsoft.com/en-us/azure/key-vault/).