# Infrastructure as Code – Azure Resource Model 
Paul Smith, August 2016
------------

### Introduction
The traditional IT paradigm has a legacy/perception of poor documentation and a release pipeline that is typically slow, inconsistent and lacks good goverence. In-part this is a sympton of the technology and siloing of IT disciplines e.g. networking, storage, compute and at a higher level develpoment and operations. Whilst industry frameworks such as ITIL impose some structure and goverence the traditional IT paradigm requires considerable effort to implement and maintain this structure and as a result is often inconsistent and poses a risk to the business.
Although these issues are mostly understood and managed, moving to a cloud  paradigm with the same mindset will create new issues that are less understood, difficult to manage and therefore increase risk to the business. 
The cloud paradigm is arguably an evolutionary step in IT with the convergence of the traditional siloed disciplines and at a higher level development and operational tasks (referred to as ‘devops’). Furthermore the cloud is synonymous with inovation and introduces technologies that enable continuous integration and deployment models that lend themselves to automation e.g. resource templates, source control integration and APIs for managing the cloud platform. 
The goal of this model is to provide an example of a release pipeline and cloud architecture that leverage these technologies enabling better management of cloud infrastructure and a more agile IT service that reduces business risk and provides a platform for business growth.


### Design Goals
* Resource templates to provide functional documention of infrastructure.
* Nested resource templates to utilise code reuse.
* Resource units to describe a collection of resources that  collectively interact as a unit with other resource units e.g. a server comprises a NIC, public IP and a dedicated storage account.
* Source control to manage versioning of resource templates.
* Source control folder structure to organise resource templates .
* Resource tags to store metadata to enable reporting and GUI management tools.
* Servers treated as ‘cattle not pets’ i.e. no manual configuration.
* Desired State Configuration to control configuration.
* JAE - just enough adminstration principles to control administration.
* Azure Key Vault to store credentials used during the build process.
* Automation to implement and schedule virtual machine shutdown and startup.
* Automated build test release pipeline – GUI driven tools to assemble builds, validate and deploy.
* Resource groups (containers) for lifecycle management of resource units.
* Resource tags for reporting.
* Automation for dev/test server shutdown and startup – implemented as a resource tag.
* The resource build container is the functional build document and is held in source control.

### Design Constraints
* Each server has it’s own resource group to make lifecycle easier. This approach requires a separate storage account per server so the overall server count will be limited by the subscription limits for storage accounts. This approach is optional

### Resource Templates
A ‘resource template’ describes a set of Azure resources in a format that can be interpreted and provisioned by the Azure  fabric. The template is imdepodent meaning that running it more than once will not cause any issues unlike a script that will attempt to recreate a resource that already exists. Another way of saying this is that the template describes the resources and Azure will only create the resource if it doesn’t already exist. In contrast a script is a set of steps to create the resource and has no awareness if the resource already exists unless coded.
Nested templates are resource templates that are called from a parent template. This approach allows for common resources to desribed in a separate template and just called instead of describing them repeatedly.

### Resource Templates
A ‘resource template’ describes a set of Azure resources in a format that can be interpreted and provisioned by the Azure  fabric. The template is imdepodent meaning that running it more than once will not cause any issues unlike a script that will attempt to recreate a resource that already exists. Another way of saying this is that the template describes the resources and Azure will only create the resource if it doesn’t already exist. In contrast a script is a set of steps to create the resource and has no awareness if the resource already exists unless coded.
Nested templates are resource templates that are called from a parent template. This approach allows for common resources to desribed in a separate template and just called instead of describing them repeatedly.

### Resource Units
A ‘resource unit’ describe a collection of resources that  collectively interact as a unit with other resource units e.g. a server comprises a NIC, public IP and a dedicated storage account
Resource units can be identified in a configuration management system as a ‘config item’ (CI) that has relationships to other CIs e.g. a server ‘is located in’ a virtual network
These relationships enables change or incident impact to be identified e.g. the server can be deleted with no impact to the network however a network incident will impact the server

### Resource Tags
Resource tags are custom ‘name-value’ pairs that can be associated with a resource in Azure. There is no central schema for these tags so they should be set as part of the deployment process to ensure consistency. An example of a resource tag would be ‘Environment-Production’. The purpose of resource tags is to enhance reporting and  GUI management tools e.g. report on all production virtual machines belonging to the Finance.
Servers – Cattle not Pets
This term contrasts servers whose configuration state is known and controlled vs servers whose configuration is less known and controlled, usually a result of manual practices. A ‘cattle’ server is easier to manage and presents less risk then a ‘pet’ server. 

### Desired State Configuration (DSC)
DSC is a Microsoft technology for applying and monitoring configuration of Window servers. Servers run an agent that configures the server from a centrally managed configuration script applicable to that server. In additiona to an initial configuration the agent can be set to monitor and reapply the configuration if changed locally. DSC provides a known state of the server that can help to reduce incidents and simplfy management.

### Just Enough Administration (JEA)
JEA provides a roles based access control (RBAC) platform through PowerShell Remoting to enable administrators to accomplish pre-defined operational tasks on servers without giving them administrator rights. This is achieved through endpoints that are associated with a pre-defined scripts that are executed on behalf of the user with a local administrator account. 

### Azure Key Vault
Azure Key Vault is a service that enables secrets to be stored in the cloud. This service eliminates the exposure of secrets such as administrator passwords during the build process.

### Automation of Server Shutdown and Startup
Azure server costs are calculated on an hourly provisioned basis therefore to reduce costs servers can be shutdown when not in use (deallocated). To automate this task a PowerShell script is scheduled to run every hour using the Azure Automation service. The script checks all servers for a resource tag called ‘autoscheduledshutdown’ and either shuts the server down or restarts it if required. The resource tag has a string value that contains one or more schedules in a valid date string format e.g. 8PM -> 12AM, Saturday, December 25.
*Reference: Technet Script Center  – Scheduled Virtual machine Shutdown/Startup*

### Automated Build Test Release Pipeline
Provisioning a resource typically includes three main tasks; build, test and release. Automating these tasks provides efficiency, consistency, accountability and transperency. 

### Resource Groups
In Azure resource groups are logical containers for resources that enable management of  thoes resources as a single entity e.g. when a resource group is deleted all resources withing the group are automatically deprovisioed and deleted.
This approach is useful for managing the lifecycle of a ‘resource unit’ such as a server as the entire server can be easily deprovisioned by deleting the resource group. Note that any shared resources such as the virtual network that the NIC connects are not located in separate resource groups. For this reason each server in this model has it’s own storage account.
 
### Source Control
Source control enables automatic versioing of templates providing a record of changes made to the template over time.  This provides accountability and transperency and should link back to a change management system.

### Source Control Folder Structure
Organising the source repository is necessary for continuous integration and deployment process automation.

![Source Control Folder Structure](https://github.com/Asperia/Asperia/blob/master/Source_Control_Folder_Structure.PNG "Source Control Folder Structure")

### Build Templates
A Build template is a pre-assembled collection of the resource templates and scripts required to build a given type of resource e.g. a file server, a IIS server.
New resources can be created simply by copying a build template and modifying the ‘Deploy Parameters’ markdown file. 
Alternatively the build template can be used as a starting point for more customised resources.

![Build Template](https://github.com/Asperia/Asperia/blob/master/Build_Template.PNG "Build Template")

### Creating a New Build Template
A ‘Build Template’ folder contains all the templates and scripts required to create a new resource in Azure.
The folder contains;
-	A resource ‘deploy template’ e.g. “VM_DeployTemplate_001.json” (this is the main template that is deployed by ARM).
-	Any component resource (nested) templates called from the ‘deploy template’ e.g. “NIC_001.json”.
-	A ‘Deploy Parameters’ markdown file that is a functional document for the resource that includes the deployment parameters.
-	 A ‘Parameters_Template’ json file used to create the ‘DeployTemplate.Parameters.json’ file during the build process based on the ‘Deploy Parameters’ markdown file.
-	A ‘readme’ markdown file to describe any metadata related to the build template.

#### To create a new ‘Build Template’

1.	Create a folder under ‘\Resources\Build Templates’ with a name format;
<resource type>_Build_<identifier>
e.g. “VM_Build_020”

2.	Copy the following files into the folder;
*	\Resources\Scripts\BuildAndDeploy.ps1
*	\Resources \Scripts\DeployParameters.md
*	\Resources \Scripts\README.md
*	\Resources \Resource Templates\Deploy Templates\<deploy template>.json’
       e.g. “VM_DeployTemplate_001.json”
*	\Resources \Resource Templates\Deploy Templates\
<deploy template>.parameters_template.json’
        e.g. “VM_DeployTemplate_001. parameters_template.json”
*	‘\Resources \Resource Templates\<component resource folder>\<component resource>.json’
        e.g “\Resources \Resource Templates\NIC\NIC_001.json’

3.	Update the ‘README’ markdown file in the root of ‘\Resources\Build templates’ folder to document the new built template metadata.


### Creating a New Resource from a Build Template
1.	Copy a ‘build template’ e.g. ‘VM_Build_001’ from the ‘\Resources\Build Templates’ folder to the ‘\TEST’ folder.
2.	Rename the build template folder to the new resource name e.g. ‘tvm00010’.
3.	Edit the ‘Deploy Parameters.md’ file.
4.	Open the ‘BuildAndDeploy.ps1’ script.
5.	Run the script – this will only execute Step 1 in the ‘Main’ code block of the script. This step calls the ‘BuildParameterFile’, ‘ConfirmParameterFile’ and ‘UpdateReadmeMarkdown’ functions to create and validate the ‘<DeployTemplate>.parameter.json’ file.
6.	Review the summary output from step 1 to confirm the ‘<DeployTemplate>.parameter.json’ file is correct.
7.	Upload the new resource folder to the Azure storage account under the ‘servers’ container.
8.	Step 2 – login to Azure.
9.	Step 3 – create the resource group.
10.	Step 4 – deploy template with parameters file.

Azure Resource Template ‘parameters.json’ Abstraction PowerShell Module
The purpose of this PowerShell module is to provide a set of functions that enable an Azure Resource Template ‘parameters.json’ file to be described more simply and legibly in a markdown file. The benefit of a markdown file is that it can be documented and formatted as a business document whilst serving as a functional document for provisioning infrastructure in Azure.
The module includes a function to parse the markdown file and extract parameters that can be used to populate the ‘parameters.json’ file. This function can also be run in ‘test’ mode to compare the parameters found in the markdown vs the target ‘parameters.json’ file. There is also a function to validate the ‘parameters.json’ file once it has been populated.
The following three patterns are parsed in the markdown file;

| 1.	**Standard parameter**  | 2.	**Hash table**     |  3.	**Array**         |
|:--------------------------|:---------------------|:---------------------|
|                           |                      |                      |
| Parameter Name = value    | Parameter Name = {   | Parameter Name = [   |
|                           | Name = value         | Name                 |
|                           | ...                  | ...                  |
|                           | }                    | }                    |

Action taken;
Pattern 1 - **Standard parameter **
*Direct mapping of name/value*






