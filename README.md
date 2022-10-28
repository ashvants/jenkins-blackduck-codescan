# jenkins-blackduck-codescan
This is a Jenkins Pipeline script for automated blackduck code scan and notification.


## Overview

[Jenkins](https://www.jenkins.io/) is a an open source automation server mainly used to implement Continuous Integration (CI) and Continuous Delivery (CD) for any development project. CI/CD, a key component of a DevOps strategy, allows you to develop the application lifecycle while maintaining quality by automating tasks like building, testing, code scan / code compliance, end-to-end delivery.

Black Duck (https://community.synopsys.com/s/article/Black-Duck-A-Technical-Introduction) allows you to discover the open source in your code and map discovered components to known vulnerabilities. This tutorial is show how Black Duck scan can be integrated into your dev-ops process for a jenkins pipeline and to monitor your projects in the background and raise alert you as new threats arise.

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

[Enterprise level](https://docs.github.com/en/enterprise-server@2.22/actions/hosting-your-own-runners/adding-self-hosted-runners#adding-a-self-hosted-runner-to-an-enterprise) runners are not working yet as there's no API definition for those.

## Pipeline Stages 

The pipeline stages are

1. clear dir: This clears the workspace directory everytime the pipeline is ran.
2. Checkout: This is mainly generic to trigger the checkout source code management repository.
3. Scan Code: This takes the Git Branch values and intiates the scan. 
4. Scan Code: Also contains has powershell code that just records the branch name, project name and build version everytime the scan is ran.



```groovy
    agent any
    parameters {
        booleanParam(name: 'DEV', defaultValue: true, description: "Select Dev build option")
        booleanParam(name: 'PROD', defaultValue: false, description: "Select Prod build option")
        string(name: 'EMAIL', defaultValue: 'emailaddress.gmail.com', description: 'Email notification')
    }
```


Regardless of which authentication method you use, the same permissions are required, those permissions are:
- Repository: Administration (read/write)
- Repository: Actions (read)
- Organization: Self-hosted runners (read/write)


**NOTE: It is extremely important to only follow one of the sections below and not both.**

