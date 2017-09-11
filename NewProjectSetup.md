# BCDevOps-Guide

New Project Setup
-----------------

1. Setup the OpenShift project spaces.  At minimum there must be a Tools and Dev OpenShift projects.

2. Setup a Github repositories to store code for the project.

   1. A small project that has a single application deployment can have just one repository
   2. Medium sized projects can use one repository with sub-directories, and then use the context directory
   3. It is recommended that as soon as you have frequent updates occurring to multiple areas   of the repository that affect multiple deployments, that you split the repository up.  This simplifies the structure of the repository and avoids the need for complex continuous integration setup.

3. Create and OpenShift / Templates folder in the repository
4. Create a build template containing the following:
   1. Parameters needed by other sections of the template:
```json
"parameters": [
    {
      "name": "APPLICATION_NAME",
      "displayName": "Application Name",
      "description": "The name given to the application",
      "required": true,
      "value": "nmp"
    },
    {
      "name": "BUILD_PROJECT",
      "displayName": "Build Project",
      "description": "The openshift project where builds and target images are stored.",
      "required": true,
      "value": "agri-nmp-tools"
    },    
    {
      "name": "BACKEND_NAME",
      "displayName": "Backend Name",
      "description": "The name assigned to all of the backend objects defined in this template.",
      "required": true,
      "value": "nmp"
    },
    {
      "name": "RPROXY_NAME",
      "displayName": "Reverse Proxy Name (SiteMinder)",
      "description": "The name assigned to the objects used as a SiteMinder entry point (or Reverse Proxy).  Typcially this is an NGINX instance.",
      "required": true,
      "value": "cerberus"
    },
    {
      "name": "DEPLOYMENT_TYPE",
      "displayName": "Deployment Type",
      "description": "The name assigned to the imagestreams defined in this template.",
      "required": true,
      "value": "latest"
    },
    {
      "name": "SOURCE_REPOSITORY_URL",
      "displayName": "Source Repository",
      "description": "The source repository to use for the builds.",
      "required": true,
      "value": "https://github.com/bcgov/agri-nmp.git"
    },	
    {
      "name": "GIT_REFERENCE",
      "displayName": "Git Reference",
      "description": "Optional branch, tag, or commit.",
      "required": true,
      "value": "master"
    },
    {
      "name": "EDITOR_NAME",
      "displayName": "Swagger Editor Name",
      "description": "The name assigned to all of the swagger editor objects defined in this template.",
      "required": true,
      "value": "editor"
    },
    {
      "name": "MOCKSERVER_NAME",
      "displayName": "Mock Server Name",
      "description": "The name assigned to all of the mock server objects defined in this template.",
      "required": true,
      "value": "mock"
    },
    {
      "name": "SCHEMASPY_NAME",
      "displayName": "Schema Spy Name",
      "description": "The name assigned to all of the schema spy objects defined in this template.",
      "required": true,
      "value": "schema-spy"
    },
	{
      "name": "SONAR_DATABASE_USER",
      "displayName": "DB Username",
      "description": "Username for Sonar Database user that will be used for accessing the database.",
      "generate": "expression",
      "from": "user[A-Z0-9]{3}"
    },
    {
      "name": "SONAR_DATABASE_PASSWORD",
      "displayName": "Database Password",
      "description": "Password for the Sonar Database user.",
      "generate": "expression",
      "from": "[a-zA-Z0-9]{16}"
    },
	{
      "name": "SONAR_DATABASE_ADMIN_PASSWORD",
      "displayName": "Database Admin Password",
      "description": "Password for the Admin Database user.",
      "generate": "expression",
      "from": "[a-zA-Z0-9]{16}"
    },
	{
      "name": "SONAR_ADMIN_PASSWORD",
      "displayName": "Sonar admin password",
      "description": "Password for the Sonar admin user.",
      "generate": "expression",
      "from": "[a-zA-Z0-9]{16}"
    },
	{
      "name": "DATABASE_VOLUME_CAPACITY",
      "displayName": "Database volume capacity",
      "description": "Size of the database used for SonarQube",
      "value": "5Gi"
    },
	{
      "name": "DATABASE_SERVICE_NAME",
      "displayName": "Database service name",
      "description": "Name of the database service",
      "value": "postgresql-sonarqube"
    }
```
   3. SonarQube
```json
{
      "kind": "Secret",
      "apiVersion": "v1",
      "metadata": {
        "name": "sonarqube-secrets"
      },
      "stringData" : {
        "database-user" : "${SONAR_DATABASE_USER}",
        "database-password" : "${SONAR_DATABASE_PASSWORD}",
		"database-admin-password" : "${SONAR_DATABASE_ADMIN_PASSWORD}",
		"sonar-admin-password" : "${SONAR_ADMIN_PASSWORD}"
      }
    },
{
    "kind": "DeploymentConfig",
    "apiVersion": "v1",
    "metadata": {
        "name": "sonarqube",
        "generation": 1,
        "creationTimestamp": null,
        "labels": {
            "app": "sonarqube"
        }
    },
    "spec": {
        "strategy": {
            "type": "Rolling",
            "rollingParams": {
                "updatePeriodSeconds": 1,
                "intervalSeconds": 1,
                "timeoutSeconds": 600,
                "maxUnavailable": "25%",
                "maxSurge": "25%"
            },
            "resources": {},
            "activeDeadlineSeconds": 21600
        },
        "triggers": [
            {
                "type": "ConfigChange"
            },
            {
                "type": "ImageChange",
                "imageChangeParams": {
                    "automatic": true,
                    "containerNames": [
                        "sonarqube"
                    ],
                    "from": {
                        "kind": "ImageStreamTag",
                        "namespace": "openshift",
                        "name": "sonarqube:6.0"
                    }
                }
            }
        ],
        "replicas": 1,
        "test": false,
        "selector": {
            "app": "sonarqube",
            "deploymentconfig": "sonarqube"
        },
        "template": {
            "metadata": {
                "creationTimestamp": null,
                "labels": {
                    "app": "sonarqube",
                    "deploymentconfig": "sonarqube"
                },
                "annotations": {
                    "openshift.io/container.sonarqube.image.entrypoint": "[\"./bin/run.sh\"]"
                }
            },
            "spec": {
                "volumes": [
                    {
                        "name": "sonarqube-extensions",
                        "persistentVolumeClaim": {
                            "claimName": "sonarqube-data"
                        }
                    }
                ],
                "containers": [
                    {
                        "name": "sonarqube",
                        "image": "openshiftdemos/sonarqube@sha256:90bc4c270d3a9f9923ef0b38f7904cfb9c00e4307d4d853e9341a334e8f29cf0",
                        "ports": [
                            {
                                "containerPort": 9000,
                                "protocol": "TCP"
                            }
                        ],
                        "env": [
                            {
                                "name": "SONARQUBE_JDBC_USERNAME",
                                "valueFrom": {
                                      "secretKeyRef" : {
                                        "name" : "sonarqube-secrets",
                                        "key" : "database-user"
                                    }
                                }
                            },
                            {
                                "name": "SONARQUBE_JDBC_URL",
                                "value": "jdbc:postgresql://${DATABASE_SERVICE_NAME}/sonarqube"
                            },
                            {
                                "name": "SONARQUBE_JDBC_PASSWORD",
                                "valueFrom": {
                                      "secretKeyRef" : {
                                        "name" : "sonarqube-secrets",
                                        "key" : "database-password"
                                    }
                                }
                            },
                            {
                                "name": "SONARQUBE_ADMINPW",
                                "valueFrom": {
                                      "secretKeyRef" : {
                                        "name" : "sonarqube-secrets",
                                        "key" : "sonar-admin-password"
                                    }
                                }    
                            }
                        ],
                        "resources": {
                            "limits": {
                                "cpu": "1",
                                "memory": "4Gi"
                            }
                        },
                        "volumeMounts": [
                            {
                                "name": "sonarqube-extensions",
                                "mountPath": "/opt/sonarqube/extensions"
                            }
                        ],
                        "readinessProbe": {
                            "httpGet": {
                                "path": "/",
                                "port": 9000,
                                "scheme": "HTTP"
                            },
                            "timeoutSeconds": 1,
                            "periodSeconds": 10,
                            "successThreshold": 1,
                            "failureThreshold": 3
                        },
                        "terminationMessagePath": "/dev/termination-log",
                        "imagePullPolicy": "IfNotPresent"
                    }
                ],
                "restartPolicy": "Always",
                "terminationGracePeriodSeconds": 30,
                "dnsPolicy": "ClusterFirst",
                "securityContext": {}
            }
        }
    },
    "status": {
        "latestVersion": 0,
        "observedGeneration": 0,
        "replicas": 0,
        "updatedReplicas": 0,
        "availableReplicas": 0,
        "unavailableReplicas": 0
    }
},
{
    "kind": "Service",
    "apiVersion": "v1",
    "metadata": {
        "name": "sonarqube",
        "creationTimestamp": null,
        "labels": {
            "app": "sonarqube"
        }
    },
    "spec": {
        "ports": [
            {
                "name": "9000-tcp",
                "protocol": "TCP",
                "port": 9000,
                "targetPort": 9000
            }
        ],
        "selector": {
            "app": "sonarqube",
            "deploymentconfig": "sonarqube"
        },
        "type": "ClusterIP",
        "sessionAffinity": "None"
    },
    "status": {
        "loadBalancer": {}
    }
},
```  
   2. SonarQube Postgresql instance
     
```json
,
	{
      "kind": "PersistentVolumeClaim",
      "apiVersion": "v1",
      "metadata": {
        "name": "postgresql-sonarqube-pvc"
      },
      "spec": {
        "accessModes": [
          "ReadWriteOnce"
        ],
        "resources": {
          "requests": {
            "storage": "${DATABASE_VOLUME_CAPACITY}"
          }
        }
      }
    },
	{
      "kind": "PersistentVolumeClaim",
      "apiVersion": "v1",
      "metadata": {
        "name": "sonarqube-data"
      },
      "spec": {
        "accessModes": [
          "ReadWriteOnce"
        ],
        "resources": {
          "requests": {
            "storage": "2Gi"
          }
        }
      }
    },
	{
      "kind": "Service",
      "apiVersion": "v1",
      "metadata": {
        "name": "${DATABASE_SERVICE_NAME}",
        "annotations": {
          "description": "Exposes the database server"
        }
      },
      "spec": {
        "ports": [
          {
            "name": "postgres",
			"protocol": "TCP",
            "port": 5432,
            "targetPort": 5432,
			"nodePort": 0
          }
        ],
        "selector": {
          "app": "${DATABASE_SERVICE_NAME}"
        },
        "type": "ClusterIP",
        "sessionAffinity": "None"
      },
	   "status": {
        "loadBalancer": {}
      }
    },	
	{
		"kind": "DeploymentConfig",
		"apiVersion": "v1",
		"metadata": {
			"name": "${DATABASE_SERVICE_NAME}",
			"generation": 1,
			"creationTimestamp": null,
			"labels": {
						"app": "${DATABASE_SERVICE_NAME}"
			},			
			"annotations": {
				"description": "Defines how to deploy the database",
				"openshift.io/container.postgresql.image.entrypoint": "[\"container-entrypoint\",\"run-postgresql\"]"
			}
		},
		"spec": {
			"strategy": {
				"type": "Recreate"				
			},
			"triggers": [
				{
					"type": "ConfigChange"
				},
				{
					"type": "ImageChange",
					"imageChangeParams": {
						"automatic": true,
						"containerNames": [
							"postgres"
						],
						"from": {
							"kind": "ImageStreamTag",
							"namespace":  "openshift",
							"name": "postgres:9.4"
						}						
					}
				}
			],
			"replicas": 1,
			"test": false,
			"selector": {
				"app": "${DATABASE_SERVICE_NAME}",
				"deploymentconfig": "${DATABASE_SERVICE_NAME}"
			},
				
			"template": {
				"metadata": {
					"creationTimestamp": null,
					"labels": {
						"app": "${DATABASE_SERVICE_NAME}",
						"deploymentconfig": "${DATABASE_SERVICE_NAME}"
					},
					"annotations": {
						"openshift.io/container.postgresql.image.entrypoint": "[\"container-entrypoint\",\"run-postgresql\"]"
					}
				},
				"spec": {
					"volumes": [
						{
							"name": "${DATABASE_SERVICE_NAME}-data",
							"persistentVolumeClaim": {
								"claimName": "${DATABASE_SERVICE_NAME}-pvc"
							}
						}
					],
					"containers": [
						{
							"name": "postgres",
							"image": "registry.access.redhat.com/rhscl/postgresql-94-rhel7",
							"ports": [
								{
									"containerPort": 5432,
									"protocol": "TCP"
								}
							],
							"env": [
								{
									"name": "POSTGRESQL_DATABASE",
									"value": "sonarqube"
								},
								{
									"name": "POSTGRESQL_PASSWORD",
									"valueFrom": {
                                      "secretKeyRef" : {
                                        "name" : "sonarqube-secrets",
                                        "key" : "database-password"
										}
                                    }                        
								},
								{
									"name": "POSTGRESQL_ADMIN_PASSWORD",
									"valueFrom": {
                                      "secretKeyRef" : {
                                        "name" : "sonarqube-secrets",
                                        "key" : "database-admin-password"
										}
                                    }                        
								},
								{
									"name": "POSTGRESQL_USER",
									"valueFrom": {
                                      "secretKeyRef" : {
                                        "name" : "sonarqube-secrets",
                                        "key" : "database-user"
										}
                                    }                        
								}
							],
							"readinessProbe": {
								"timeoutSeconds": 1,
							  "initialDelaySeconds": 15,
							  "exec": {
								"command": [ "/bin/sh", "-i", "-c", "psql -h 127.0.0.1 -U $POSTGRESQL_USER -q -d $POSTGRESQL_DATABASE -c 'SELECT 1'"]                    
							  }
							},
							"livenessProbe": {
							  "timeoutSeconds": 1,
							  "initialDelaySeconds": 30,
							  "tcpSocket": {
								"port": 5432
							  }
							},
							"resources": {},
							"volumeMounts": [
								{
									"name": "${DATABASE_SERVICE_NAME}-data",
									"mountPath": "/var/lib/pgsql/data"
								}
							],
							"terminationMessagePath": "/dev/termination-log",
							"imagePullPolicy": "Always"
						}
					],
					"restartPolicy": "Always",
					"terminationGracePeriodSeconds": 30,
					"dnsPolicy": "ClusterFirst",
					"securityContext": {
                  "capabilities": {},
                  "privileged": false
                }
            },
			"restartPolicy": "Always",
            "dnsPolicy": "ClusterFirst"			
		},		
		"status": {}
	    }
    },
```

  3. Do not create a deployment for the Jenkins Pipeline, allow the platform to create that.
  4. Image streams that will be used to store the application's images
	
```json
{
      "kind": "ImageStream",
      "apiVersion": "v1",
      "metadata":
      {
        "name": "<NAME>",
        "generation": 1,
        "creationTimestamp": null
      },
      "spec":
      {
        "tags": [
          {
            "name": "latest",
            "annotations": null,
            "generation": null,
            "importPolicy": {}
          }

        ]
      },
      "status": {}
    }   	
```

  5. Build configurations that will create the images used for deployment of the app.

```json
{
      "kind": "BuildConfig",
      "apiVersion": "v1",
      "metadata":
      {
        "name": "${BACKEND_NAME}",
        "creationTimestamp": null,
        "labels":
        {
          "app": "${BACKEND_NAME}"
        }
      },
      "spec":
      {
        "triggers": [
          {
            "type": "ImageChange",
            "imageChange": {}
          },
          {
            "type": "ConfigChange"
          }
        ],
        "runPolicy": "Serial",
        "source":
        {
          "type": "Git",
          "git":
          {
            "uri": "${SOURCE_REPOSITORY_URL}",
			"ref": "${GIT_REFERENCE}"
          },
          "contextDir": ""
        },
        "strategy": {
					"type": "Source",
					"sourceStrategy": {
						"from": {
							"kind": "ImageStreamTag",
							"namespace": "${BUILD_PROJECT}",
							"name": "s2i-dotnetcore:latest"
						},
						"env": [
							{
								"name": "BUILD_LOGLEVEL",
								"value": "5"
							}
						],
						"incremental": false
					}
				},
        "output":
        {
          "to":
          {
            "kind": "ImageStreamTag",
            "namespace": "${BUILD_PROJECT}",
            "name": "${BACKEND_NAME}:${DEPLOYMENT_TYPE}"
          }
        },
        "resources": {},
        "postCommit": {},
        "nodeSelector": null
      },
      "status":
      {
        "lastVersion": 0
      }
    },   
```

  6. Any secrets needed for the build
  7. A pipeline definition for each significant build configuration in the project.
		1. The pipeline definition should exist as a single Jenkinsfile
		2. You may specify a specific name for the Jenkinsfile if you need to store multiple pipelines in a single repository
		3. The Jenkinsfile should be located in the same repository as the code it supports
		4. Do not have blocking statements such as "input" within a node block; this will cause the slave to exist forever, resulting in resources being consumed. 

```groovy
node('maven') {
    stage('build') {
        echo "Building..."
        openshiftBuild bldCfg: 'gwells', showBuildLogs: 'true'
        openshiftTag destStream: 'gwells', verbose: 'true', destTag: '$BUILD_ID', srcStream: 'gwells', srcTag: 'latest'
    }
    
    stage('deploy-dev') {
        echo "Deploying to dev..."
        openshiftTag destStream: 'gwells', verbose: 'true', destTag: 'dev', srcStream: 'gwells', srcTag: 'latest'
    }
    
    stage('checkout for static code analysis') {
        echo "checking out source"
        echo "Build: ${BUILD_ID}"
        checkout scm
    }

    stage('code quality check') {
        SONARQUBE_PWD = sh (
            script: 'oc env dc/sonarqube --list | awk  -F  "=" \'/SONARQUBE_ADMINPW/{print $2}\'',
            returnStdout: true
        ).trim()
        echo "SONARQUBE_PWD: ${SONARQUBE_PWD}"

        SONARQUBE_URL = sh (
            script: 'oc get routes -o wide --no-headers | awk \'/sonarqube/{ print match($0,/edge/) ?  "https://"$2 : "http://"$2 }\'',
            returnStdout: true
        ).trim()
        echo "SONARQUBE_URL: ${SONARQUBE_URL}"

        dir('sonar-runner') {
            sh returnStdout: true, script: "./gradlew sonarqube -Dsonar.host.url=${SONARQUBE_URL} -Dsonar.verbose=true --stacktrace --info  -Dsonar.sources=.."
        }
    }
	
	stage('validation') {
        dir('navunit') {
            sh './gradlew --debug --stacktrace phantomJsTest'
        }
    }
}

stage('deploy-test') {
    input "Deploy to test?"
  
    node('maven') {
        openshiftTag destStream: 'gwells', verbose: 'true', destTag: 'test', srcStream: 'gwells', srcTag: '$BUILD_ID'
    }
}

stage('deploy-prod') {
    input "Deploy to prod?"
    
    node('maven') {
        openshiftTag destStream: 'gwells', verbose: 'true', destTag: 'prod', srcStream: 'gwells', srcTag: '$BUILD_ID'
    }
}

``` 
  8. A Pipeline Build Configuration that will run the pipeline
 
```json
{
    "kind": "BuildConfig",
    "apiVersion": "v1",
    "metadata": {
        "name": "${APPLICATION_NAME}-pipeline",
        "creationTimestamp": null,
        "labels": {
            "app": "${APPLICATION_NAME}-pipeline",
            "name": "${APPLICATION_NAME}-pipeline",
            "template": "${APPLICATION_NAME}-pipeline"
        }
    },
    "spec": {        
        "runPolicy": "Parallel",
        "source": {
            "type": "Git",
            "git": {
                "uri": "${SOURCE_REPOSITORY_URL}",
                "ref": "master"
            }
        },
        "strategy": {
            "type": "JenkinsPipeline",
            "jenkinsPipelineStrategy": {
                "jenkinsfilePath": "pdf.Jenkinsfile"
            }
        },
        "output": {},
        "resources": {},
        "postCommit": {},
        "nodeSelector": null
    },
    "status": {
        "lastVersion": 0
    }
}
```
4. Add the appropriate starter code for the tech stack selected.
5. Verify that the build configurations work

  `oc process -f build-template.json`  (fix any errors)
  `oc process -f build-template.json | oc create -f -`

6. Verify that the pipeline works.  In the web UI, click the start pipeline button.

7. Create a deployment template to deploy the image or images created by the build

The deployment template should only contain Deployment Configuration, Service and Route objects.  If you need to make custom images, do that in the Tools project. 

8. Implement basic functional tests as part of the pipeline to verify the application is online
9. Document any special steps within the OpenShift/Templates/Readme.md file.  You may wish to use a tool such as [MarkdownPad](http://markdownpad.com) to edit the markdown files.
