Infrastructure as Code – Azure Resource Model

Paul Smith, August 2016

Introduction

The traditional IT paradigm has a legacy/perception of poor documentation and a release pipeline that is typically slow, inconsistent and lacks good goverence. In-part this is a sympton of the technology and siloing of IT disciplines e.g. networking, storage, compute and at a higher level develpoment and operations. Whilst industry frameworks such as ITIL impose some structure and goverence the traditional IT paradigm requires considerable effort to implement and maintain this structure and as a result is often inconsistent and poses a risk to the business.

Although these issues are mostly understood and managed, moving to a cloud paradigm with the same mindset will create new issues that are less understood, difficult to manage and therefore increase risk to the business.

The cloud paradigm is arguably an evolutionary step in IT with the convergence of the traditional siloed disciplines and at a higher level development and operational tasks (referred to as ‘devops’). Furthermore the cloud is synonymous with inovation and introduces technologies that enable continuous integration and deployment models that lend themselves to automation e.g. resource templates, source control integration and APIs for managing the cloud platform.

The goal of this model is to provide an example of a release pipeline and cloud architecture that leverage these technologies enabling better management of cloud infrastructure and a more agile IT service that reduces business risk and provides a platform for business growth.

Design Goals

-   Resource templates to provide functional documention of infrastructure.

-   Nested resource templates to utilise code reuse.

-   Resource units to describe a collection of resources that collectively interact as a unit with other resource units e.g. a server comprises a NIC, public IP and a dedicated storage account.

-   Source control to manage versioning of resource templates.

-   Source control folder structure to organise resource templates .

-   Resource tags to store metadata to enable reporting and GUI management tools.

-   Servers treated as ‘cattle not pets’ i.e. no manual configuration.

-   Desired State Configuration to control configuration.

-   JAE - just enough adminstration principles to control administration.

-   Azure Key Vault to store credentials used during the build process.

-   Automation to implement and schedule virtual machine shutdown and startup.

-   Automated build test release pipeline – GUI driven tools to assemble builds, validate and deploy.

-   Resource groups (containers) for lifecycle management of resource units.

-   Resource tags for reporting.

-   Automation for dev/test server shutdown and startup – implemented as a resource tag.

-   The resource build container is the functional build document and is held in source control.

Design Constraints

-   Each server has it’s own resource group to make lifecycle easier. This approach requires a separate storage account per server so the overall server count will be limited by the subscription limits for storage accounts. This approach is optional

Resource Templates

A ‘resource template’ describes a set of Azure resources in a format that can be interpreted and provisioned by the Azure fabric. The template is imdepodent meaning that running it more than once will not cause any issues unlike a script that will attempt to recreate a resource that already exists. Another way of saying this is that the template describes the resources and Azure will only create the resource if it doesn’t already exist. In contrast a script is a set of steps to create the resource and has no awareness if the resource already exists unless coded.

Nested templates are resource templates that are called from a parent template. This approach allows for common resources to desribed in a separate template and just called instead of describing them repeatedly.

Resource Units

A ‘resource unit’ describe a collection of resources that collectively interact as a unit with other resource units e.g. a server comprises a NIC, public IP and a dedicated storage account

Resource units can be identified in a configuration management system as a ‘config item’ (CI) that has relationships to other CIs e.g. a server ‘is located in’ a virtual network

These relationships enables change or incident impact to be identified e.g. the server can be deleted with no impact to the network however a network incident will impact the server

Resource Tags

Resource tags are custom ‘name-value’ pairs that can be associated with a resource in Azure. There is no central schema for these tags so they should be set as part of the deployment process to ensure consistency. An example of a resource tag would be ‘Environment-Production’. The purpose of resource tags is to enhance reporting and GUI management tools e.g. report on all production virtual machines belonging to the Finance.

Servers – Cattle not Pets

This term contrasts servers whose configuration state is known and controlled vs servers whose configuration is less known and controlled, usually a result of manual practices. A ‘cattle’ server is easier to manage and presents less risk then a ‘pet’ server.

Desired State Configuration (DSC)

DSC is a Microsoft technology for applying and monitoring configuration of Window servers. Servers run an agent that configures the server from a centrally managed configuration script applicable to that server. In additiona to an initial configuration the agent can be set to monitor and reapply the configuration if changed locally. DSC provides a known state of the server that can help to reduce incidents and simplfy management.

Just Enough Administration (JEA)

JEA provides a roles based access control (RBAC) platform through PowerShell Remoting to enable administrators to accomplish pre-defined operational tasks on servers without giving them administrator rights. This is achieved through endpoints that are associated with a pre-defined scripts that are executed on behalf of the user with a local administrator account.

Azure Key Vault

Azure Key Vault is a service that enables secrets to be stored in the cloud. This service eliminates the exposure of secrets such as administrator passwords during the build process.

Automation of Server Shutdown and Startup

Azure server costs are calculated on an hourly provisioned basis therefore to reduce costs servers can be shutdown when not in use (deallocated). To automate this task a PowerShell script is scheduled to run every hour using the Azure Automation service. The script checks all servers for a resource tag called ‘autoscheduledshutdown’ and either shuts the server down or restarts it if required. The resource tag has a string value that contains one or more schedules in a valid date string format e.g. 8PM -&gt; 12AM, Saturday, December 25.

*Reference: Technet Script Center – Scheduled Virtual machine Shutdown/Startup*

Automated Build Test Release Pipeline

Provisioning a resource typically includes three main tasks; build, test and release. Automating these tasks provides efficiency, consistency, accountability and transperency.

Resource Groups

In Azure resource groups are logical containers for resources that enable management of thoes resources as a single entity e.g. when a resource group is deleted all resources withing the group are automatically deprovisioed and deleted.

This approach is useful for managing the lifecycle of a ‘resource unit’ such as a server as the entire server can be easily deprovisioned by deleting the resource group. Note that any shared resources such as the virtual network that the NIC connects are not located in separate resource groups. For this reason each server in this model has it’s own storage account.

Source Control

Source control enables automatic versioing of templates providing a record of changes made to the template over time. This provides accountability and transperency and should link back to a change management system.

Source Control Folder Structure

Organising the source repository is necessary for continuous integration and deployment process automation.

Build Templates

A Build template is a pre-assembled collection of the resource templates and scripts required to build a given type of resource e.g. a file server, a IIS server.

New resources can be created simply by copying a build template and modifying the ‘Deploy Parameters’ markdown file.

Alternatively the build template can be used as a starting point for more customised resources.

<img src="media/image1.png" width="228" height="121" />

Creating a New Build Template

A ‘Build Template’ folder contains all the templates and scripts required to create a new resource in Azure.

The folder contains;

-   A resource ‘deploy template’ e.g. “VM\_DeployTemplate\_001.json” (this is the main template that is deployed by ARM).

-   Any component resource (nested) templates called from the ‘deploy template’ e.g. “NIC\_001.json”.

-   A ‘Deploy Parameters’ markdown file that is a functional document for the resource that includes the deployment parameters.

-   A ‘Parameters\_Template’ json file used to create the ‘DeployTemplate.Parameters.json’ file during the build process based on the ‘Deploy Parameters’ markdown file.

-   A ‘readme’ markdown file to describe any metadata related to the build template.

To create a new ‘Build Template’

1.  Create a folder under ‘\\Resources\\Build Templates’ with a name format;

> &lt;resource type&gt;\_Build\_&lt;identifier&gt;
>
> e.g. “VM\_Build\_020”

1.  Copy the following files into the folder;

-   \\Resources\\Scripts\\BuildAndDeploy.ps1

-   \\Resources \\Scripts\\DeployParameters.md

-   \\Resources \\Scripts\\README.md

-   \\Resources \\Resource Templates\\Deploy Templates\\&lt;deploy template&gt;.json’
    e.g. “VM\_DeployTemplate\_001.json”

-   \\Resources \\Resource Templates\\Deploy Templates\\
    &lt;deploy template&gt;.parameters\_template.json’
    e.g. “VM\_DeployTemplate\_001. parameters\_template.json”

-   ‘\\Resources \\Resource Templates\\&lt;component resource folder&gt;\\&lt;component resource&gt;.json’
    e.g “\\Resources \\Resource Templates\\NIC\\NIC\_001.json’

1.  Update the ‘README’ markdown file in the root of ‘\\Resources\\Build templates’ folder to document the new built template metadata.

Creating a New Resource from a Build Template

1.  Copy a ‘build template’ e.g. ‘VM\_Build\_001’ from the ‘\\Resources\\Build Templates’ folder to the ‘\\TEST’ folder.

2.  Rename the build template folder to the new resource name e.g. ‘tvm00010’.

3.  Edit the ‘Deploy Parameters.md’ file.

4.  Open the ‘BuildAndDeploy.ps1’ script.

5.  Run the script – this will only execute Step 1 in the ‘Main’ code block of the script. This step calls the ‘BuildParameterFile’, ‘ConfirmParameterFile’ and ‘UpdateReadmeMarkdown’ functions to create and validate the ‘&lt;DeployTemplate&gt;.parameter.json’ file.

6.  Review the summary output from step 1 to confirm the ‘&lt;DeployTemplate&gt;.parameter.json’ file is correct.

7.  Upload the new resource folder to the Azure storage account under the ‘servers’ container.

8.  Step 2 – login to Azure.

9.  Step 3 – create the resource group.

10. Step 4 – deploy template with parameters file.

Azure Resource Template ‘parameters.json’ Abstraction PowerShell Module

The purpose of this PowerShell module is to provide a set of functions that enable an Azure Resource Template ‘parameters.json’ file to be described more simply and legibly in a *markdown* file. The benefit of a markdown file is that it can be documented and formatted as a business document whilst serving as a functional document for provisioning infrastructure in Azure.

The module includes a function to parse the markdown file and extract parameters that can be used to populate the ‘parameters.json’ file. This function can also be run in ‘test’ mode to compare the parameters found in the markdown vs the target ‘parameters.json’ file. There is also a function to validate the ‘parameters.json’ file once it has been populated.

The following three patterns are parsed in the markdown file;

| 1.  **Standard parameter** 
                             
 Parameter Name = value      | 1.  **Hash table**   
                                                    
                              > Parameter Name = {  
                              >                     
                              > Name = value        
                              >                     
                              > …                   
                              >                     
                              > }                   | 1.  **Array**         
                                                                            
                                                     > Parameter Name = \[  
                                                     >                      
                                                     > Name                 
                                                     >                      
                                                     > …                    
                                                     >                      
                                                     > \]                   |
|----------------------------|----------------------|-----------------------|

Action taken;

Pattern 1 - **Standard parameter **

*Direct mapping of name/value*

| **Markdown file example**           
                                      
 *Virtual Machine Spec*               
                                      
 *(refer project document \#892837)*  
                                      
 vmSize = Standard\_D1                | **‘parameters.json’ file BEFORE** 
                                                                          
                                       > "vmSize": {                      
                                       >                                  
                                       > "value": "\_vmSize"              
                                       >                                  
                                       > },                               | **‘parameters.json’ file AFTER** 
                                                                                                             
                                                                           "vmSize": {                       
                                                                                                             
                                                                           "value": "Standard\_D1"           
                                                                                                             
                                                                           },                                |
|-------------------------------------|-----------------------------------|----------------------------------|

Pattern 2 - **Hash table**

*Name/value pairs split into a ‘name’ array and a ‘value’ array.*

| **Markdown file example** 
                            
 *Virtual Machine Tags*     
                            
 CommonTags = {             
                            
 Organisation = Asperia     
                            
 Environment = Production   
                            
 Purpose = SQL              
                            
 }                          | **‘parameters.json’ file BEFORE**  
                                                                 
                             “commonTagsNames”: {                
                                                                 
                             “values”: \[“\_commonTagsNames”\]   
                                                                 
                             },                                  
                                                                 
                             “commonTagsValues”: {               
                                                                 
                             “values”: \[“\_commonTagsValues”\]  
                                                                 
                             },                                  | **‘parameters.json’ file AFTER**                     
                                                                                                                        
                                                                  “commonTagsNames”: {                                  
                                                                                                                        
                                                                  “values”: \[“Organisation”,”Environment”,”Purpose”\]  
                                                                                                                        
                                                                  },                                                    
                                                                                                                        
                                                                  “commonTagsValues”: {                                 
                                                                                                                        
                                                                  “values”: \[“Asperia”,”Production”,”SQL”\]            
                                                                                                                        
                                                                  },                                                    |
|---------------------------|------------------------------------|------------------------------------------------------|

| Example code in a resource template for a hash table use;                          
                                                                                     
 "tags": {                                                                           
                                                                                     
 "\[parameters('commonTagNames')\[0\]\]": "\[parameters('commonTagValues')\[0\]\]",  
                                                                                     
 },                                                                                  |
|------------------------------------------------------------------------------------|

Pattern 3 – **Array**

*Name/value pairs split into two arrays*

| **Markdown file example**                
                                           
 *VM Storage Accounts*                     
                                           
 storageAccounts = saaue00023, saaue00005  | **‘parameters.json’ file BEFORE** 
                                                                               
                                            “storageAccounts”: {               
                                                                               
                                            “values”: \[“\_storageAccounts”\]  
                                                                               
                                            },                                 | **‘parameters.json’ file AFTER**        
                                                                                                                         
                                                                                “storageAccounts”: {                     
                                                                                                                         
                                                                                “values”: \[“saaue00023”,”saaue00005”\]  
                                                                                                                         
                                                                                },                                       |
|------------------------------------------|-----------------------------------|-----------------------------------------|

Example Deploy Parameters Markdown

&lt;\#

Resource deployed by Paul Smith 08/08/16

Coomera Sport Complex project – PRJ00453

Requested W.Gouldie - SRQ010023

\#&gt;

vmName = tvm00010

\# tags to be assigned to all resources in the build

commonTags = {

Organisation = Asperia Digital

Environment = Test

Purpose = IT Infrastructure

Owner = Paul Smith

Description = Corporate File Server - AAuto DSC deployed

Deployed By = Paul Smith

Date Deployed = 09/08/16

ServerDesk Reference = SRQ010023

} max = 10

\# server specs

vmWindowsOSVersion = 2012-R2-Datacenter

vmSize = Standard\_D1

vNetName = NetworkFBAUE

vNetResourceGroup = AsperiaNetworkFBAUE

subnetName = FrontEnd

storageType = Standard\_LRS

\# auto shutdown schedule

autoShutdownScheduleTag = 10PM -&gt; 6AM

\# nested templates to use

sharedTemplateNameVM = VM\_001.json

sharedTemplateNameStorage = StorageAccount\_001.json

sharedTemplateNameNIC = NIC\_001.json

sharedTemplateNamePublicIp = PublicIP\_001.json

\# build template

VMBuildTemplate = VM\_DeployTemplate\_001

\# these parameters don't normally require updating

\#-------------------------------------------------

\# local admin details from key vault

vmAdminUserNameSecretID = /subscriptions/9e21eb5f-a0e8-4223-bc78-a3cb9865324c/resourceGroups/AsperiaKeyVault/providers/Microsoft.KeyVault/vaults/AsperiaKeyVault

vmAdminUserNameSecretName = vmAdminName2

vmAdminPasswordSecretID = /subscriptions/9e21eb5f-a0e8-4223-bc78-a3cb9865324c/resourceGroups/AsperiaKeyVault/providers/Microsoft.KeyVault/vaults/AsperiaKeyVault

vmAdminPasswordSecretName = vmAdminPassword2

\# azure resource names

nicSuffix = -nic-0

publicIpName = PublicIP

\# nested resource deployment names

nestedDeploymentNameVM = deploy-virtual-machine

nestedDeploymentNameStorage = deploy-storage-account

nestedDeploymentNameNic = deploy-nic

nestedDeploymentNamePublicIp = deploy-public-ip

\# dsc

nodeConfigurationName = MyFeatureInstall

modulesURL = https://asperiastoragelrsaue1.blob.core.windows.net/resources/UpdateLCMforAAPull.zip

configurationFunction = UpdateLCMforAAPull.ps1\\\\ConfigureLCMforAAPull

registrationKey = 354aOl8jfKxljQHU1L4O/Dd7ec199XqrTm78LLB9L5xDHC83lTL64bEey9hK+IY4dI7w6ekFZR0Isu8fKCzNhQ==

registrationUrl = https://ase-agentservice-prod-1.azure-automation.net/accounts/1b3bfe6f-e327-41a2-9100-07efdcc728b5

configurationMode = ApplyAndMonitor

configModeFrequencyMins = 15

refreshFrequencyMins = 30

rebootNodeIfNeeded = true

actionAfterReboot = ContinueConfiguration

allowModuleOverwrite = false

timestamp = MM/dd/yyyy H:mm:ss tt

\# location of template on Azure blob storage

baseTemplateUri = https://asperiastoragelrsaue1.blob.core.windows.net/servers/%\#vmName\#%

BuildAndDeploy.ps1

\#region Initialse

\# build variables

$templateFile = (get-childitem "$PSScriptRoot" -Filter "\*.parameters\_template.json").Name

$deployFile = $templateFile.TrimEnd(".parameter\_template.json")

$global:parameterTemplateFile = Get-Content "$PSScriptRoot\\$templateFile"

$global:parameterInputFile = Get-Content "$PSScriptRoot\\parameters.txt"

$global:parameterOutputFile = "$PSScriptRoot\\$deployFile.parameters.json"

$global:ParameterFileConfirmed = $false

\# deploy variables (used in steps 2 to 4 i.e. not used in any functions)

$resourceGroupName = 'tvm00100'

$location = 'Australia East'

$resourceDeploymentName = "$resourceGroupName-vm-deployment"

$baseTemplateUri = "https://asperiastoragelrsaue1.blob.core.windows.net/servers/$resourceGroupName"

$template = "$baseTemplateUri/$deployFile.json"

$templateParameters = "$baseTemplateUri/$deployFile.parameters.json"

\#endregion

\#region Functions

function BuildParameterFile() {

Param (\[switch\]$verbose)

if($verbose){$VerbosePreference="continue"}

\# initialise

Write-Verbose "initialising"

$processingMulitlineParameter = $false

$maxCommonTags = 10

$commentBlock = $false

$dictParamNames = @{}

\# process input file

foreach($line in $global:parameterInputFile){

\# ignore any empty lines, comment lines or blocks

if($line.Length -le 1){continue}

if($line.Substring(0,2) -eq '\#&gt;'){$commentBlock=$false;continue}

if($line.Substring(0,1) -eq '\#' -or $commentBlock){continue}

if($line.Substring(0,2)-eq'&lt;\#'){$commentBlock=$true;continue}

\# check if the line is the end of a 'multiline parameter block' such as a hash table or array

if($line.IndexOf('}') -eq -1 -and $line.IndexOf('\]') -eq -1)

{ \# process the 'name=value' line

$equalSignIndex = $line.IndexOf('=')

if($equalSignIndex -eq -1 -and $processingMulitlineParameter)

{ \# process an array value

$paramName = $line.TrimStart().TrimEnd()

} else

{ \# process a 'name=value' (includes hash table lines)

$paramName = $line.Substring(0, $equalSignIndex).trimend()

}

if($line.IndexOf('{') -ge 0 -or $line.IndexOf('\[') -ge 0)

{ \# start of a hash table or an array

Write-Verbose "Processing hash table - $paramName"

$processingMulitlineParameter=$true;

$multilineParamName = $paramName

$hashNameArray = $null

$hashValueArray = $null

$hashArrayCount = 0

$hashTableParameter=$false

if($line.IndexOf('{') -ge 0){$hashTableParameter=$true}

} else

{ \# process single-line name:value

$paramValue = $line.Substring($equalSignIndex+1).trimstart()

\# look for a reference to a parameter defined in this parameter file i.e. format $!&lt;paramater name&gt;!$

if($paramValue.IndexOf('%\#') -ge 0)

{

$parameterReferenceName = $line.Substring($line.IndexOf('%\#')+2,($line.IndexOf('\#%')-$line.IndexOf('%\#')-2))

\# check if referenced parameter is in the dictionary of parameters already processed

if ($dictParamNames.ContainsKey($parameterReferenceName))

{

$parameterReferenceValue = $dictParamNames.Get\_Item($parameterReferenceName)

$replaceStr = '%\#'+$parameterReferenceName+'\#%'

$paramValue = $paramValue -replace $replaceStr, $parameterReferenceValue

}

}

if($processingMulitlineParameter)

{

$hashNameArray += '"'+$paramName+'",'

$hashValueArray += '"'+$paramValue+'",'

$hashArrayCount++

} else

{

\# add the name:value to the dictionary in case it is referenced elsewhere in the parameter file

$dictParamNames.Add($paramName,$paramValue)

$global:parameterTemplateFile = $global:parameterTemplateFile -replace "\_$paramName", $paramValue

}

}

} else

{ \# end of hash table - convert to two string arrays

Write-Verbose "End Processing hash table - $paramName"

$hashArrayString = $hashNameArray.Substring(1,$hashNameArray.Length-3)

For($i=$hashArrayCount+1;$i-le10;$i++){$hashArrayString+='","Tag '+$i}

$replaceStr = "\_$multilineParamName"+'Names'

$global:parameterTemplateFile = $global:parameterTemplateFile -replace $replaceStr, $hashArrayString

Write-Verbose "$replaceStr = $hashArrayString"

$hashArrayString = $hashValueArray.Substring(1,$hashValueArray.Length-3)

For($i=$hashArrayCount+1;$i-le10;$i++){$hashArrayString+='","empty'}

$replaceStr = "\_$multilineParamName"+'Values'

$global:parameterTemplateFile = $global:parameterTemplateFile -replace $replaceStr, $hashArrayString

Write-Verbose "$replaceStr = $hashArrayString"

$processingMulitlineParameter = $false

}

}

Write-Verbose "Created $global:parameterOutputFile"

$global:parameterTemplateFile | Out-File $global:parameterOutputFile

}

function ConfirmParameterFile() {

Param (\[switch\]$verbose)

if($verbose){$VerbosePreference="continue"}

Write-Verbose "Checking $parameterOutputFile"

Write-Verbose "Failed - $parameterOutputFile"

$global:ParameterFileConfirmed = $false

}

\#endregion

\#region Main

\# Step 1 - only this step is executed when the script is run

BuildParameterFile -verbose

ConfirmParameterFile -verbose

if($global:ParameterFileFailedCheck){return}

\# Step 2 - login to Azure

{

Add-AzureRMAccount

}

\# Step 3 - Create the resource group (variables located at top of script and initialised when script is run)

{

New-AzureRmResourceGroup \`

-Name $resourceGroupName \`

-Location $Location \`

-Verbose -Force

}

\# Step 4 - Deploy template with parameters file

{

New-AzureRmResourceGroupDeployment \`

-Name $resourceDeploymentName \`

-ResourceGroupName $resourceGroupName \`

-TemplateUri $template \`

-TemplateParameterUri $templateParameters \`

-Verbose -Force

}

\#endregion
