#  CICD With Oracle Integration Cloud


## Introduction

This workshop walks you through how to create a Continuous Integration and Continuous Delivery(CICD) Pipeline for Oracle Integration Cloud(OIC) Artifacts including: Integrations, Connectors, Packages, Processes, and Visual Builder Cloud Applications. This allows us to implement DevOps to our integration development. This lab is only meant to be a jumping off point and we will be giving you a step-by-step process for exporting artifacts. The purpose of this lab is to give you the tools to create and edit your own CICD pipeline while providing a backbone for both exporting and importing artifacts.

**How to Get Your Free Cloud Trial Account**

> If you already have an Oracle Cloud account then you can skip this section. If you don't have an Oracle Cloud account then you can quickly and easily sign up for a free trial account that provides:
> $300 of free credits good for up to 3500 hours of Oracle Cloud usage
> Credits can be used on all eligible Cloud Platform and Infrastructure services for the next 30 days
> Your credit card will only be used for verification purposes and will not be charged unless you 'Upgrade to Paid' in My Services.
>
> ![http://www.oracle.com/webfolder/technetwork/tutorials/learning\_path/images/700705-auto-dw-social-bn728\_-152.png](images/pic1.png)
>
> Once your trial account is created, you will receive a Welcome to Oracle Cloud email that contains your cloud account password along with links to useful collateral. Click here to sign into the Oracle Cloud, go to: [*https://cloud.oracle.com*](https://myservices.us.oraclecloud.com/mycloud/signup?language=en&sourceType=:ex:tb:::RC_NAMK181017P00031:ADW_IMHOL&SC=:ex:tb:::RC_NAMK181017P00031:ADW_IMHOL&pcode=NAMK181017P00031)
>

###  **What is DevOps?**
DevOps or Developer Operations is a methodology, mindset and culture that allows companies and projects to be continuously changed, tested and deployed to a live production environment. By employing DevOps practices, companies switch from large releases every couple of months to smaller bi-weekly to weekly releases. This allows for more immediate bug fixes and for new features to be released much more frequently. This in-turn increases the overall quality of your product or process at any given time.

### **Why should I use devops with OIC?**
DevOps can be extremely important for OIC. It allows us to fix issues with our applications having inconsistencies or issues more quickly and more frequently. Additionally, it will allow us to store all versions of our OIC artifacts. If we accidentally allow a bug to be pushed to production we can quickly move back to our last version or even two or three versions back.

### **How will our CICD Pipeline work?**
For the purposes of this lab we will be using just one integration environment, however, in a production environment we recommend using three integration environments. The reason for this is to have a more modular building and production process. The three environments would be development(DEV), testing(TEST), and production(PROD). Our development can happen in our DEV environment.  When completed we move the developed artifacts to TEST for testing before finally moving the artifacts to PROD where our live OIC artifacts live. If testers find an issue they can push issues to the DEV team for the DEV team to try and fix. If no issues are found they can be moved to PROD, where the OIC artifacts can work in our live environments. 

### **DevCS: Configuration File and Build Scripts**
Developer Cloud Service(DevCS) is a tool that we will be using to build out our CICD pipeline. DevCS has many tools within it, but for the purposes of this lab we will be using its Repository and Builds. The repository is a git repository we will be using to store all of our artifacts including our configuration files. The builds uses hudson to run scripts to export and import our OIC artifacts. This config file will allow us to dynamically change our OIC artifacts that we want to move while maintaining a static build script .

###  **About This Lab**
This lab is not meant to give you an end to end full solution to all of your CICD needs. It will, however, give you a jumping off point for doing so. It will give you the tools to understand how to tackle the problem of how to create a CICD pipeline for OIC in general and give you some initial scripts to get you on your way. We will walk you through how to set up one of our two scripts and this should provide the tooling for you to set up the second script on your own. If there are multiple people following this lab follow lab 100 all together. In lab 200 there is a point that specifies when you can split up and do your own repos and builds. 

##  **Navigate to Lab 100**

- _You can see a list of Lab Guides_ by clicking on the **Menu Icon** in the upper left corner of the browser window. You're now ready to continue with [**Lab 100**](https://adam-paz.github.io/IntegrationCICDWorkshop/?page=LabGuide100.md).

Feedback would be much appreciated, if you have any feedback please email me at: adam.paz@oracle.com

  ![](images/LabMenuIcon.png)
