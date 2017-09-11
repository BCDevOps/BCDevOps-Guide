# BCDevOps-Guide

Introduction
------------

OpenShift is a Linux based platform as a service hosting environment. 
It is capable of hosting applications or services that are accessible via the web, in a fault tolerant, scalable fashion.

## OpenShift Tips
* Scaling
* Secrets Mgmt
* Vanilla vs Custom images
* Pipelines - Jenkins
* S2I Images
* Data Container Networking
* Advanced OpenShift template for reuse
* Docker Basics
* Restoring

OpenShift Network Description
----------
There are three types of networks available to OpenShift:
1.	Internet / Cloud – these are public facing locations on the Internet.  TLS can be used to provide some security over the Internet.
2.	DMZ – the Government DMZ
3.	Other Zones – it is possible to network to other zones by special request.



## Getting Setup in Your New Project
* Environments / Projects
* App Stack chat
* HealthChecks
* Jobs/Schedules
* Backup Procedures
* Troubleshooting Builds
* Automated testing
* Storage Chat
* Auth and SiteMinder

## GitHub and Git Intro
* Github Forking and Pull Request Best Practices
* Git Basics: conflict resolution

## On boarding new Dev's
* Roles
* Tools


Future Considerations
--------------------
stack share
starter apps


New Project Setup
----------------
[New Project Setup Sub Page](NewProjectSetup.md)

Jenkins
-------

Jenkins is automatically created by OpenShift when the build pipeline is started.

It is recommended you start the pipeline once to create it, make the configuration settings mentioned below, and then setup the hook to automatically build on pull requests.

**Important Notes**

As of 2017-07-11 there is a known issue with the default Jenkins configuration - the Jenkins URL is not set correctly.  To correct this:
- Login to Jenkins as Administrator
- Go to Manage Jenkins
- Set the Jenkins URL to the URL of the OpenShift Route for Jenkins

The default Jenkins configuration also has a health check that only waits for 3 seconds before running.  This is not enough time for the Jenkins pod to start running.  Increase the initial check delay to 30 seconds to allow it to start running before the health check runs.

By default the Kubernetes plugin does not override the OpenShift defaults for memory consumption.  In order for the build pipeline to work you will need to allocate **4Gi** of RAM to the **maven** node template.  (More may be required as the volume of code increases).  

To increase the amount of RAM available in the maven node template, do the following:
- login to Jenkins as administrator
- Click on Manage Jenkins
- Scroll down until you are at the maven kubernetes template area
- Click the advanced button to expose the memory limit fields
- Increase the amount of ram to a suitable amount.  Do a test build to verify you have enough.
- Click save

Note:  On the OpenShift Dashboard you will see two deployment objects under the group Jenkins Persistent.  This is because the Jenkins server has two services.  One service is the main web based user interface, and the other is a JNLP interface that the slaves use to communicate with the main server.

Fault Tolerance & Resiliency
----------------------------

A key requirement for any application deployed to OpenShift is that it must implement Health Checks.  Health checks ensure that the application has been properly deployed, and that it has the necessary resources to service customers.
Specific areas that must be monitored by the health check include:
•	Access to any persistent volumes used by the deployment
•	Access to any databases used by the deployment


Designing a Pipeline
-------------------

Terms:
- Node - an OpenShift pod that will be used to execute pipeline commands.   Supported types:
	- master is the Jenkins server itself
	- maven is a JDK equipped node, capable of running mvn
	- gradle is a JDK equipped node capable of running gradle
- Stage - a block of code within the Pipeline that has a clear pass or fail.  Each stage is shown on the user interface.
- Jenkinsfile - the code file that contains the pipeline definition.  Written in Groovy
- Build Configuration - the OpenShift object that refers to the Jenkinsfile.  Build configurations that refer to Jenkinsfiles will show on the Pipeline view.
- Rules:
1. You may have stages inside or outside of a Node block
2. Commands that interact with the shell or a build environment must be inside a node block
3. Do not put the input statement or other blocking statements inside a node block
4. The BuildConfiguration that refers to the Jenkinsfile should 

Testing a Pipeline
------------------
Temporarily point the pipeline build configuration at your own repository.  This will allow you to test the pipeline before you do a pull request to move the code to the main project repository.
1. Edit the Build Configuration, change the Repository details to your details
2. Manually start the pipeline
3. Observe the results
4. Change the pipeline build configuration back to point to the main repository

TROUBLESHOOTING AND DAMAGE CONTROL
---------------

[Troubleshooting and Damage Control](Troubleshooting.md) have been moved to separate document.


Reference Architectures
-----------------------
### Language / Stack Selection ###

| Name | UI Code | Server Code | Suggested Database	| Pros | Cons |
| ---- | ------- | ----------- | ------------------ | ---- | ---- |
| MEAN	| JavaScript (Angular) |	Node.js	| Mongo	| Relatively easy to onboard new developers | Slow builds |
| .NET Core | JavaScript | .Net Core C# MVC | Postgresql | Similar to Windows .NET, Efficient | More complex |
| Django | Python or Javascript	| Python | Mongo | Fastest development, great for prototypes | Python language limitations |
| Ruby | Javascript | Ruby | Varies	| High level language | |	
| Java	| JavaScript | Java |	Postgresql, MySQL	| Allows use of enterprise code; ideal for enterprise level software | Requires greater amount of development experience / time |

# Proxies #

NGINX is the most commonly used proxy in the BC Gov repository.


## License
<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons Licence" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/80x15.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">BC DevOps Guide by the Province of British Columbia</span> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.

