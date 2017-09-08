Troubleshooting and Damage Control
----------------------------------

Contents
1. [Issues with Builds](#buildIssues)
2. [Issues with Deployments](#deploymentIssues)
	1. [Missing database migration](#missingMigration)
	2. [Image Pull Problems](#imagePull)

## Issues with Builds  (Tools Project)<a name="buildIssues"></a>
------

| Area | Problem | Solution |
| ---- | ------- | -------- |
| Jenkins | Slaves do not start, following text: `Using 64 bit Java since OPENSHIFT_JENKINS_JVM_ARCH is not set | Set Jenkins config setting - Maven Kubernetes option, new environment variable OPENSHIFT_JENKINS_JVM_ARCH with value of x86_64 |
| Jenkins | Slaves start but cannot communicate with the master | Set the Kubernetes Jenkins URL and JNLP fields to an appropriate value. If the values are appropriate, then check that the platform is not having DNS issues - it can get into a state where local services doe not resolve to valid IP addresses|
| OpenShift / Jenkins | OpenShift pipeline builds appear "stuck" - they start by may be at the running stage for hours. Viewing the console logs for the jenkins pod shows that there are problems deleting a Jenkins job. | Login to Jenkins and manually delete the job.  If you are presented with an error, rsh to the Jenkins pod and use the rm command to delete the folder in question, and retry the delete from the Jenkins UI. |
| Jenkins | Jenkins marks a build as failed after 15 minutes, but the build succeeds in OpenShift | By default the limit on a build is 15 minutes.  Adjust the OpenShift build timeout in the Jenkins configuration to allow for a longer build. |

##Issues with Deployments (Dev, Test, Prod...)<a name="deploymentIssues"></a>
---------------------------------------------

## Symptom ##<a name="missingMigration"></a>

Deployment fails.

Deployment log has text:
`error: timed out waiting for any update progress to be made`

Pod created during deployment fails.  The failed pod contains the following as the last line:
`django.db.utils.ProgrammingError: relation "server_audit" does not exist`

## Root Cause ##
Application is refusing to startup due to a missing database table.

## Solution ##
1. Setup a test environment
2. Reproduce the problem
3. Develop a fix
4. Deploy the fix

--------------------------------------

## Symptom ##<a name="imagePull"></a>

Deployment fails due to image pull problem.

## Root Cause ##
Varies, but may be caused by pruning or missing permissions.

## Solution ##
1. Ensure that appropriate image puller permissions exist.
`oc policy add-role-to-user system:image-puller system:serviceaccount:moe-land-designations-dev:default -n moe-land-designations-tools`
2. You may have to re-tag the image (or redo the build).