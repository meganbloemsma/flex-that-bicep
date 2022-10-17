**Set up Azure environment:**

    Connect-AzAccount

    $context = Get-AzSubscription -SubscriptionName 'Concierge Subscription'
    Set-AzContext $context

    Get-AzSubscription

    $context = Get-AzSubscription -SubscriptionId {Your subscription ID}
    Set-AzContext $context

    Set-AzDefault -ResourceGroupName learn-7b3e1bd6-b3c9-4dec-bc3e-c83df1ee3a94

    New-AzResourceGroupDeployment -TemplateFile main.bicep

**Verify deployment:**

    Get-AzResourceGroupDeployment -ResourceGroupName learn-7b3e1bd6-b3c9-4dec-bc3e-c83df1ee3a94 | Format-Table

**Deploy bicep template in environment 'nonprod':**
New-AzResourceGroupDeployment `
  -TemplateFile main.bicep `
  -environmentType nonprod