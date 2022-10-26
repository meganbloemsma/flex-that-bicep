# :bell: Testing

Your pipeline should enforce quality checks before, during and after each deployment. 

Here I will focus on how to extend your Azure DevOps (ADO) pipeline. I also assume you're using [CI/CD](https://microsoft.github.io/code-with-engineering-playbook/continuous-integration/CICD/).

# Table of Contents

1. :pushpin: [Useful links](https://github.com/meganbloemsma/flex-that-bicep/blob/main/docs/testing.md#pushpin-useful-links)
2. :clapper: [Using stages](https://github.com/meganbloemsma/flex-that-bicep/blob/main/docs/testing.md#using-stages)
3. :ribbon: [Using Bicep linter](https://github.com/meganbloemsma/flex-that-bicep/blob/main/docs/testing.md#using-bicep-linter)
4. :last_quarter_moon: [What-if operation](https://github.com/meganbloemsma/flex-that-bicep/blob/main/docs/testing.md#what-if-operation)

## :clapper: Using 'stages'

By using stages you can verify the quality of your code before you deploy it. You can view stages as 'steps' your deployment takes. With each step something is validated, checked, tested, etc.

You can define these stages yourself in ADO's Pipelines. There are [templates](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/env-templates?view=azure-devops) available but you can also manually define a stage through YAML or the ADO portal. Here I will elaborate on how to do it through YAML, but for all of these steps you can check the [documentation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/add-template-to-azure-pipelines?tabs=CLI) on how to do it in the portal.

When you've manually defined a stage you can [save it as a template](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/env-templates?view=azure-devops#save-a-stage-template) so you can re-use it later.

You can control the sequence of stages and add dependencies, e.g. that a stage runs only if the previous stage was successful. You can specify the dependencies by using the 'dependsOn' keyword.

In a YAML file it looks like this:

  stages:

    - stage: Test
    jobs:
    - job: Test

    - stage: DeployUS
    dependsOn: Test
    jobs: 
    - job: DeployUS

    - stage: DeployEurope
    jobs: 
    - job: DeployEurope

## :ribbon: Using Bicep linter

Linting is the process of checking your code against a set of recommendations. The Bicep linter runs automatically when you use the Bicep tooling, and every time you build a Bicep file the linter runs along with it.

You want your Bicep templates to be linted each time anyone checks in code to your repo. Add a lint stage and job to your pipeline to ensure this':

    stages:

    - stage: Lint
    jobs: 
    - job: Lint
        steps:
        - script: |
            az bicep build --file deploy/main.bicep

If you want to if your Bicep template is likely to deploy in your Azure environment successfully (without deploying any resources), you can do a [*preflight validation*](https://learn.microsoft.com/en-us/training/modules/test-bicep-code-using-azure-pipelines/3-lint-validate-bicep-code?pivots=powershell).
You use the AzureResourceManagerTemplateDeployment task to submit a Bicep file for preflight validation, and configure the deploymentMode to Validation:

    - stage: Validate
    jobs: 
    - job: Validate
        steps:
        - task: AzureResourceManagerTemplateDeployment@3
            inputs:
            connectedServiceName: 'MyServiceConnection'
            location: $(deploymentDefaultLocation)
            deploymentMode: Validation
            resourceGroupName: $(ResourceGroupName)
            csmFile: deploy/main.bicep

## :last_quarter_moon: what-if operation

