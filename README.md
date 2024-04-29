# Edge Infrastructure QuickStart Template (Preview)

## Overview

This Quick-Start template simplifies your Infrastructure as Code journey with Azure edge products througout their lifecycle. It includes a few Terraform modules (**AKS Arc**, **Azure Stack HCI**, **Arc Site Manager** and **Arc extensions**), a scalable hands-on repository structure and the automation tools to streamline the setup the infrastructure configurations.

### What You'll Get

By using this template, you can get all of the followings inside a single PR under your Github account

* a scalable and extendible repository structure following the DevOps best practice
* organized variables with the recommended values to boost your initial setup
* a customizable CD pipeline with the automations.

### Is This Right for You?

**Yes** if you want to:

* Create a site containing AKS Arc, HCI23H2 along with Arc extensions using Terraform
* Replicate the above site's settings hundreds or thoudsands of times
* Integrate the above settings with CI/CD pipeline using Github Actions
* Automate all of the above scenarios

**No** if you want to:

* Create single AKS Arc or HCI instance using Terraform. Altough this template contains the Terraform module for each of them, we are still waiting to officially publish them into public Terraform registry. You are welcome to use this repository for testing and exploration. For production usage, please contact arcIaCSupport@microsoft.com for each module's status.
* Use any other IaC tool such as Bicep or ARM to provision your Azure resouces. We are working on our roadmap. Please stay tuned for future releases. 

<details>

<summary><b>Repo Structure</b></summary>

<img src="doc/img/repoStructure.png" alt="repoStructure" width="800"/>

```PROJECT_ROOT
│
├───.azure
│   │   backendTemplate.tf              // Backend storage account config file
│   │
│   └───hooks
│           pre-commit                  // Git hook to generate deployment workflow and set backend
│
├───.github
│   └───workflows
│           site-cd-workflow.yml        // Set up CD pipeline
|           terraform-plan.yml
│
├───dev                                 // The first stage to deploy
│   └───sample
│           backend.tf
│           main.tf                     // Main configuration file for the site
│           provider.tf
│           terraform.tf
│           variables.tf
│
├───modules
│   ├───base                            // Base module of all sites
│   │       main.tf
│   │       variables.tf
│   │
│   ├───hci                             // Module to manage HCI clusters
│   │
│   ├───hci-extensions                  // Module to manage HCI extensions                                                                     
│   ├───hci-provisioners                // Module to connect servers to Arc
│   │───aks-arc                         // Module to manage AKS Arc clusters
│   └───hci-vm                          // Module to manage HCI VMs
│   └───site-manager                    // Module to manage site-manager
│
├───prod                                // prod stage sites are deployed after qa stage
│   │
│   └───prod1
│
└───qa                                  // qa stage sites are deployed after dev stage
    │
    └───qa1
```

Base module contains the global variables across all sites. Each stage and each site folder contains the local variables specific to the stage/site. Local settings can override the global settings.

</details>

<details>

<summary><b>Variables Structure</b></summary>

| Variable Type           | Description                                                                                                     | Example             | Where to set value                                                                                       | Override Priority |
| ----------------------- | --------------------------------------------------------------------------------------------------------------- | ------------------- | -------------------------------------------------------------------------------------------------------- | :---------------: |
| Global Variables        | The values of the global variables typically are consistent across the whole fleet but specific for one product | `domainFqdn` in HCI | Set in `modules/base/<product>.hci.global.tf`. Add default value for variables.                          |        low        |
| Site specific variables | The values of these variables are unique in each site                                                           | `siteId`            | These variables must be set in the site `main.tf` file under each site folder                            |       high        |
| Pass through variables  | The values of these variables are inherited from GitHub secrets                                                 | `subscriptionId`    | `modules/base/<product>.hci.misc.tf`                                                                     |                   |
| Reference variables     | These variables are shared by 2 or more products                                                                | `location`          | Its definition can be found in `variables.<product>.*.tf` if its link is `ref/<product>/<variable_name>` |                   |

</details>

### Supported Azure edge resource types

<details>

<summary><b> Supported Azureedge resource types</b></summary>

* [Azure Stack HCI, version 23H2](https://learn.microsoft.com/en-us/azure-stack/hci/whats-new)
* [Azure Stack HCI extensions](https://learn.microsoft.com/en-us/azure-stack/hci/manage/arc-extension-management?tabs=azureportal)
* [Azure Kubernetes Service (AKS) enabled by Azure Arc](https://learn.microsoft.com/en-us/azure/aks/hybrid/)
* [Arc Site Manager](https://review.learn.microsoft.com/en-us/azure/azure-arc/site-manager/overview?branch=release-preview-site-manager)
* [Azure resource group](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/overview)

</details>

## Getting started

This repository implements AD preparation and Arc connection. Follow the instructions below to set up the rest of the components.

* [Getting-Started](./doc/Getting-Started.md)

Otherwise, you need to finish AD preparation and connect servers to Arc by yourself for all sites. Then, HCI and AKS provisioning can follow [Getting-Started-Self-Connect](./doc/Getting-Started-Self-Connect.md)

## Scenario 1: Create your fist site (for Greenfiled customers)

**Overview**: If you don't have any existing edge resources set up yet, this scenario can help you quickly set uo a fresh new site using a pre-defined infrastructure template. Otherwise, feel free to skip this scenario if you already have a testing resource group with AKS on HCI.

**Steps**: [Create your first site](./doc/Add-first-Site.md)

**Expected outcome**:

* A PR containing Terraform code set up for AKS Arc, HCI, Arc extensions under a single resource group
* A PR containign the pre-defined CI/CD pipeline containing 3 stages: Dev, QA, Prod
* Provisioning action will happen in your side (*merge the PR to `main`*)

## Scenario 2: Scale more sites (Private Preview)

This repository will help you set up the scaling configuration code quickly and get ready for scaling the fleet. You have the two options: setting up the scaling configurations by yourself or you can use our automation pipeline to accelerate the repository set up along with custom settings.

### User workflow diagram

<img src="doc/img/IaCGithubAction.png" alt="IaCGithubAction"/>

### Prerequisites

This feature is currently in **Private Preview**. Before you begin, ensure you have the access to our automation tools. Please fill in this [form](https://github.com/Azure/Edge-infrastructure-quickstart-template/issues/new?assignees=xwen11&labels=customers+onboarding&projects=&template=preview-sign-up-form.md&title=%5BOnboarding%5D) to sign up for Private Preview.
After you get the SAS tokens, following the steps below:

1. Add `PRIVATE_PREVIEW_SAS` to your GitHub repo secrets.
2. Download the binaries to run locally
    - Windows: Run `./private_preview.ps1 "<PRIVATE_PREVIEW_SAS>"`
    - Linux: Run `./private_preview.sh "<PRIVATE_PREVIEW_SAS>"`
3. Verify your downloads
    - Run `./az-edge-module-export -v` & `az-edge-site-scale -v`.
    ```
    2024/04/29 10:37:54 telemetry.go:110: InstallationId: ***, SessionId: ***
    az-edge-module-export version main(20240426.2)
    ```

### Option 1: Set up the scaling configurations with the static templates

**Overview**: This option allows for the creation of the code configurations for multiple resource groups, each containing 1 HCI cluster, 1 AKS Arc cluster and the optional monitoring Arc extenion and Arc site manager.

**Steps**:  [Add New Sites with the static templates](./doc/Add-New-Sites-with-static.md)

**Expected outcome**:

* A PR containing Terraform code for # of sites, each containing 1 HCI cluster, 1 AKS Arc cluster and the optional monitoring extension and Arc site manager extension.
* A PR containing a pre-defined CI/CD pipeline containing 3 stages: Dev, QA, Prod
* Provisioning action will happen in your side (*merge the PR to `main`*)

### Option 2: Set up the scaling configurations with the custom templates

**Overview**: If you already have a sample Site modeld with a resource group. This option will codify the existing resources and translate them into Terraform modules, then using automations to replicate the custom modules for multiple sites.

**Steps**: [Add-New-Sites-with-automation](./doc/Add-New-Sites-with-automation.md)

**Expected outcome**:

* A Github repository with Terraform code for # of sites, each containing custom settings for HCI clusters, AKS Arc clusters
* A pre-defined CI/CD pipeline containing 3 stages: Dev, QA, Prod
* Provisioning action will happen in your side (*merge the PR to `main`*)

## Enable Arc extensions for all sites

Any change merged into `main` branch will trigger the update pipeline. If the change fails in early stages, the deployment will be blocked so that this failure will not affect the production sites.

Following tutorials help you to turn on opt-in features:

* [Add HCI Insights](./doc/Add-HCI-Insights.md)
* [Add New Sites with Arc Site Manager](./doc/Add-Site-Manager.md)

## Advanced topics

- [Customize Stages](./doc/Customize-Stages.md)
- [Update global parameters](./doc/Edit-Global-Parameters.md)
- [Disable Telemetry](./doc/Disable-Telemetry.md)
- [Untrack Resources from The Repository](./doc/Untrack-Resources.md)
- [View your CI/CD pipeline running status](./doc/View-pipeline.md)
- [TroubleShoot](./doc/TroubleShooting.md)

## Ask for support

[Open issue](https://github.com/Azure/Edge-infrastructure-quickstart-template/issues/new) or contact arcIaCSupport@microsoft.com for any issue or support

## License  
  
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more information.  
  
## Disclaimer  

'Preview Terms'. This repository (the "Preview") is subject to the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/en-us/support/legal/preview-supplemental-terms/). Unless otherwise noted, Customer should not use the Preview to process Personal Data or other Data that is subject to legal or regulatory compliance requirements.

'Confidentiality'.The Preview and any associated materials or documentation are confidential information and subject to obligations in your Non-Disclosure Agreement with Microsoft.

This repository is provided "as-is" without any warranties or support. Use at your own risk. Always test in a non-production environment before deploying to production.  

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit [Microsoft opensource](https://cla.opensource.microsoft.com).

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft
trademarks or logos is subject to and must follow
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
