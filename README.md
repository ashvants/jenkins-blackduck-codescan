# Jenkins Automated Pipeline for Blackduck Code scan
This is a Jenkins Pipeline script for automated blackduck code scan and notification.


## Overview

[Jenkins](https://www.jenkins.io/) is a an open source automation server mainly used to implement Continuous Integration (CI) and Continuous Delivery (CD) for any development project. CI/CD, a key component of a DevOps strategy, allows you to develop the application lifecycle while maintaining quality by automating tasks like building, testing, code scan / code compliance, end-to-end delivery.

Blackduck (https://community.synopsys.com/s/article/Black-Duck-A-Technical-Introduction) allows you to discover the open source in your code and map discovered components to known vulnerabilities. This tutorial is show how Black Duck scan can be integrated into your dev-ops process for a jenkins pipeline and to monitor your projects in the background and raise alert you as new threats arise.

[Jenkins Pipeline as Code](https://www.jenkins.io/solutions/pipeline/) with the use of Pipeline plugin, jenkins allows to implement a project’s entire build/test/deploy pipeline in a Jenkinsfile that stores alongside the code repository, the pipeline as another piece of code checked into source control.


## Guide

- This is configured as a seperate pipeline since we had to run this scan on a daily basis. 
- This can be integrated in the normal jenkinsFile pipeline script as well, but just to note the requirments as if you don't want to trigger application builds everyday or if you don't need to fail the builds based on the scan results then it is best to keep this a seperate pipeline.

[Jenkins Cron job Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/cron-syntax)

Configure the daily scan values basically using cron job syntax.

```
 // trigger job at everyday
     triggers {
     pollSCM('H H * * *')
    }
```

## Pipeline Parameters 

If you use either dev or prod environments then you can create custom parameters for input in the jenkins gui as below. Also you can configure any other parameters that is required. Email address to be passed as a string vaule was used to send out email notifications once the scan has been completed.

```groovy
    agent any
    parameters {
        booleanParam(name: 'DEV', defaultValue: true, description: "Select Dev build option")
        booleanParam(name: 'PROD', defaultValue: false, description: "Select Prod build option")
        string(name: 'EMAIL', defaultValue: 'emailaddress.gmail.com', description: 'Email notification')
    }
```

## Pipeline Stages 

The pipeline stages are

1. clear dir: This clears the workspace directory everytime the pipeline is ran.
2. Checkout: This is mainly generic to trigger the checkout source code management repository.
3. Scan Code: This takes the Git Branch values and intiates the scan. 
4. Scan Code: Also contains has powershell code that just reads the branch name, project name and build version in to a csv file everytime the scan is ran. The csv file is amended by the main build to record the build number.
5. Post Email Notification: Sends out the email notification once the scan has been completed.


```groovy
                            def output = powershell(returnStdout: true, script:'''
                                        $VAR_A = Import-Csv -Path "${env:JENKINS_COMMON}\\csvlocation\\csvfile.csv"                   
                                            "$(($VAR_A).MainBranch)"
                                            "$(($VAR_A).ProjectName)"
                                            "$(($VAR_A).BuildVersion)" 
                                        
                                        $VAR_A | Select-Object *, @{n="Blackduck";e={"${env:JOB_BASE_NAME}"}} | Export-CSV "${env:JENKINS_COMMON}\\csvlocation\\csvfile.csv -NoTypeInformation
                            ''')
                                        
                            env.MAIN_BRANCH = output.tokenize('\n')[0].trim()
                            env.PROJECT_NAME = output.tokenize('\n')[1].trim()
                            env.BUILD_VERSION = output.tokenize('\n')[2].trim()
```
**NOTE: Remove the above syntax if it is not required in your pipeline**


Pipeline stage for code scan 
- --detect.project.name="Project Name" \ #enter your custom project name here.
- --detect.project.version.name="${PROJECT_STR}_${ENVIRONMENT_STR}" \ #enter your custom project version name here.
- --detect.code.location.name="${PROJECT_STR}_${ENVIRONMENT_STR}" \ # enter your code location here.
- --blackduck.url="https://sca.apps.mydomain/" \ # enter your blackduck url here 
- --blackduck.api.token="${BLACKDUCK_API_TOKEN}" \ # enter your blackduck api connection token here
- --detect.source.path="${PROJECT_PATH}" > "${env.WORKSPACE}\\codescan\\logs\\${PROJECT_STR}_${env.BUILD_NUMBER}.log" # enter your blackduck logs path here

```
 // trigger job at everyday
     triggers {
     pollSCM('H H * * *')
    }
```

## Email Notification

Amend the following 

- to: to email groups or addresses normally this is configured by the pipeline parameters as stated above.
- from: from the email if your jenkins server has been configured with the smptp allow to send emails option from the address.
- subject: Modify the subject to your liking.
- body: Modify the body to your linking.
- 
### Post email notification script.
```groovy
 post {
        always {
            emailext (
                to: "${params.EMAIL}",
                from: "MyJenkins@mydomain.com",
                subject: "JENKINS BUILD - ${PROJECT_STR}: ${ENVIRONMENT_STR} - ${currentBuild.currentResult}",
                attachmentsPattern: "codescan/**/logs/*",
                body: """<p><h3><u> JENKINS BUILD DETAIL - ${PROJECT_STR}_${env.BUILD_NUMBER} </u></h3><br>
                        <b>JOB: </b> ${env.JOB_NAME} <br><br>
                        <b>BUILD STATUS: </b> ${currentBuild.currentResult} <br><br>
                        <b>DURATION: </b> ${currentBuild.durationString} <br><br>
                        <b>GIT BRANCH: </b> <a href='${env.GIT_BRANCH_URL}'>${env.GIT_BRANCH}</a> <br><br>
                        <b>CODESCAN RESULT: </b> Refer attached blackduck and Checkmarx codescan logs <br>
                        <br>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
            )  
            
        }
    }
```

## blackduck Yaml File
Lastly also included is the blackduck yaml file which basically generic and is used by blackduck scan, you can use this file excluded directories, excluded patterns and search depth. Search the [blackduck knowledge base](https://community.synopsys.com/s/black-duck-knowledgebase) for more values that can be added to the yaml file. 


Have fun implementing this and please don't forget to give credit or star on my github profile.
