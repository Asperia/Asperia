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
•	Nested resource templates to utilise code reuse.
•	Resource units to describe a collection of resources that  collectively interact as a unit with other resource units e.g. a server comprises a NIC, public IP and a dedicated storage account.
•	Source control to manage versioning of resource templates.
•	Source control folder structure to organise resource templates .
•	Resource tags to store metadata to enable reporting and GUI management tools.
•	Servers treated as ‘cattle not pets’ i.e. no manual configuration.
•	Desired State Configuration to control configuration.
•	JAE - just enough adminstration principles to control administration.
•	Azure Key Vault to store credentials used during the build process.
•	Automation to implement and schedule virtual machine shutdown and startup.
•	Automated build test release pipeline – GUI driven tools to assemble builds, validate and deploy.
•	Resource groups (containers) for lifecycle management of resource units.
•	Resource tags for reporting.
•	Automation for dev/test server shutdown and startup – implemented as a resource tag.
•	The resource build container is the functional build document and is held in source control.

