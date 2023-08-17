# Introduction

YAML is a light-weight, human-readable data-serialization language. It stands for 'Yet Another Markup Language.' Similar to XML and JSON files, but uses a more minimalistics syntax. The file is created using extensions ".yaml" or ".yml."

Essentially, YAML is a super set of JSON which means all the features of JSON can be found in YAML. YAML is widely used in writing configuration files in different DevOps tools, cloud platforms and applications. A few of these tools and these applications are:
 - Docker
 - Kubernetes
 - GitHub
 - Ansible
 - Spring
 - AWS
 - CircleCI
 - TeamCity
 - Azure
 - Travis CI
 - Jenkins

## Thumb Rules in Writing YAML

 - Identification of whitespace use used to denote structure. This is very similar to Python uses identation to highlight the blocks of code. 
 - The basic structure of a YAML file is a map. You might call this a dictionary, hash or object, depending on your favorite programming language.
 - Tabs are not included as identation for YAML files. So please be careful with space and tab inside YAML files.
 - YAML is case sensitive in nature.[name: accounts != Name: Accounts]

## Linters, Parsers, and Converters

 You could a linter to parse your YAML file, exposing any issues in construction [here](https://www.yamllint.com/) to provide you an option to validate your YAML file inside your local laptop yourself. 

 YAML parser [here](https://yaml-online-parser.appspot.com/)
 Convert JSON to YAML [here](https://json2yaml.com/)

## YAML Syntax
Syntax Documentation can be found [here]()

## Understanding YAML File Format for CI/CD Pipeline
### Pipeline Basics
There are some key points that must be understood in terms of pipeline integration:
 - Build Pipeline - Also known as CI Build for integrating new changes on cloud hosted app.
 - Release Pipeline - Also known as CD for orchestrating continuous delivery
 - YAML Pipeline - In a single pipeline, we can create a CI + CD or a multi-stage pipeline where 1 stage is focused on integrating new changes as part of a build, and other stages for deployment to different environments

### Basic YAML File Format & Structure

The basis of a YAML file at large has the skeleton as seen below:

```
name: string # build numbering format

Resources:
 pipelines: [pipelineResource]
 containers: [containerResource]
 repositories: [repositoryResource]
variables: {string: string} | {variable | templateReference}
trigger: trigger
pr: pr
stages: [stage | templateReference]
```
1. You will have a name
2. It will make use of resources (i.e., pipelines, containers, repositories) on what it is going to be used
3. Special variables for injection and substitution
4. Triggers
5. Pull requests
6. Stages

 - If you have a single stage, you can omit stafges and directly specify jobs
 - If you have a single stage and a single job, you can ommit those keywords and directly specify steps:

 #### Example
 ```
 name: $(Date:yyyyMMdd)
 variables:
  var1: value1
 jobs:
  - job: One
    steps: 
     - script: echo First step!
 ```
#### Stage
A stage is a logical boundary in the pipeline. It can be used to mark separation of concerns (i.e., Build, run tests, and deploy). Stages are the major divisions in a pipeline.  Every pipeline has at least one stage, even if you do not explicitely define it. Each stage contains one or more jobs. By default, stages run sequentially, starting only after the stage of them has completed. You can manually control when a stage should run using approval checks. 

The full syntax to specify a stage is:

```
- stage: string
 displayName: string            # name of stage
 dependsOn: string | [string]   # friendly name
 condition: string| pool
 variables: {string: string} | {variable | variableReference}
 jobs: [job | templateReference]
```

#### Example
 ```
- stage: Build
 jobs: 
  - job: BuildJob
   steps: 
    - script: echo Building!
- stage: Run Test
 jobs: 
  - job: TestOnWindows
   steps: 
    - script: echo Testing on Windows!
  - job: TestOnLinux
   steps: 
    - script: echo Testing on Linux!
- stage: Deloy
 jobs: 
  - job: Deploy
   steps: 
    - script: echo Deploying code!

 ```

#### Job
A job is a collaction of linear series of steps to be run by an agent on the server. Again, each jobs runs on an agent. It's a unit of work assignable to a particular machine. More than one jobs can run in parallel. 

 ```
jobs:
 - job: MyJob
  displayName: My First Job
  continueOnError: true
  workspace: 
   clean: outputs
  steps:
   - script: echo My First Job!

 ```
#### Agent and Agent Pools
 An agent is an installable software that runs one job at a time.

#### Step
A step is the smallest building block of a pipeline. A step can either be a _script_ or a _task_. For example, a pipeline consist of build and test steps. 

#### Tasks
A task is simply a pre-created script for you to run common build functions. 

 ```
steps:
 - script: echo This runs in the default shell on any machine
 - bash: | 
    echo this Multiline script always runs in Bash.
    echo Even on Windows machines!
 - pwsh: | 
    Write-Host "This multiline script always runs in PowerShelll Core."
    Write-Host "Even on Windows Machines!"
 - task: DotNetCoreCLI@2
  displayName: 'Building the project'
  inputs: 
   command: 'build'
   arguments: '--no-restore --configuration Release'
   projects: '**/*.csproj'

 ```
  #### Creating stages in YAML



 #### Summmary of Flow
 On a YAML pipeline we will have the following order:

 Stage -> Job -> Steps (Script | Task)

 - Every job requires a hosted agent (whether cloud or self-hosted).
 - Jobs can run in parallel, if needed, as well as sequence them. 

 More information can be found [here](https://www.letsdevops.net/post/letsdevops-complete-guide-to-learn-and-setup-yaml-pipeline-in-azure-devops)

  ### Create a YAML CI Pipeline in Azure (Single Stage)

  You typicall want to take care a few concerns:
   - Trigger a build based on a particular branch (i.e., master, development, and/or feature branches)
   - Select a specific agent (VM or local) that will run and publish the build
   - You want to restore the project dependencies
   - You want to build/rebuild the project solution
   - You may want to run some test
   - You want to publish the artifacts

  ```
  trigger: 
   - development

 # Select an image like ubuntu-latest, Windows, etc.
  pool: 
   - vmImage: <ubuntu-latest>

  stages
   - stage: A
     - jobs: 
       - job: A job
            steps:
            - script: echo "some script"
            - task: DotNetCoreCLI@2
                displayName: 'Restore project dependencies'
                inputs:
                command: 'restore'
                projects: '**/*.csproj'
                - task: DotNetCoreCLI@2
                displayName: 'Build the project - Release'
                inputs:
                command: 'build'
                projects: '**/*.csproj'
                arguments: '--configuration $(buildConfiguration)'
                - task: DotNetCoreCLI@2
                displayName: 'Publish'
                inputs:
                command: 'publish'
                arguments: '--configuration $(buildConfiguration) --output $(build.ArtifactStagingDirectory)'
                - task: PublishBuildArtifacts@1
                inputs:
                PathtoPublish: '$(Build.ArtifactStagingDirectory)'
                ArtifactName: 'drop'
                publishLocation: 'Container'

  ```
 ### Create a Multi-Stage YAML Pipeline
 More information can be found [here](https://www.letsdevops.net/post/letsdevops-complete-guide-to-learn-and-setup-yaml-pipeline-in-azure-devops)

  ### Create a YAML CI Pipeline in GitLab
  To define a GitLab pipeline, we need to add a YAML file called _.gitlab-ci.yml_ to your code repository's root folder. That file may include:
   - image: defines the base docker image to pull (for example a python image)
   - before_script: set of commans to execute before the jobs get triggered. It specifies a common configuration for all subsequent jobs.
   - stages: determines the job execution order in the pipeline
   - build: name of ther first job of the pipeline
   - script: a job's set of commands. In this case, you may decide to restore or rebuild the project
   - test: a final job that may be used to run tests

     See more information [here](https://docs.gitlab/com/ee/ci/)
     See More information on CI/CD templates [here](https://docs.gitlab/com/ee/ci/examples)

  #### Example 1
  By default, the yml file could be executed every time a commit to a specific branch is pushed to the repo. 

  ```
  default:
    image: python:3.8
    before_script: 
         - apt-get update
         - apt-get install -y python3-pip
         - pip install -r requirements.txt
  stages: 
    - build
    - test
  build:
    script:
        - some script
  test: 
    script: 
        - some script
  ```
  
 #### Example 2
  ```
  build-job:
    stage: build
    script: 
        - some script
  test-job:
    image: some image
    stage: test
    script: 
        - some script
  ```

   #### Example Cypress
  ```
  stages:
    - test

  test:
    image: cypress/browser:node16.16.9.chrome107-ff107-edge
    stage: test
    script: 
        # install dependencies
        - npm ci
        # start the server in the background
        - npm start &
        # run cypress tests
        - npx cypress run --browser firefox
  ```
  More information [here](https://docs.cypress.io/guides/continuous-integration/gitlab-ci/)

  ### Cypress Test CI/CD Documentation
   - [AWS CodeBuild](https://docs.cypress.io/guides/continuous-integration/aws-codebuild)
   - [GitLab](https://docs.cypress.io/guides/continuous-integration/gitlab-ci/) or [here](https://kailash-pathak.medium.com/how-to-execute-cypress-e2e-test-cases-using-ci-cd-gitlab-ef263154ddd0)
   - [Bitbucket](https://docs.cypress.io/guides/continuous-integration/bitbucket-pipelines)
   - [CircleCI](https://docs.cypress.io/guides/continuous-integration/circleci)
   - [GitHub Actions](https://docs.cypress.io/guides/continuous-integration/github-actions)
   - [Others](https://docs.cypress.io/guides/continuous-integration/ci-provider-examples)

  ### Playwright Test CI/CD Documentation
    - [Playwright CI/CD](https://playwright.dev/docs/ci)

  ### Maven Test CI/CD Examples
   - [Azure](https://blog/clairvoyantsoft.com/setting-up-a-selenium-pipeline-in-azure-devops-3b07b97d7872)
   - [Jenkins](https://www.softwaretestingmaterial.com/selenium-continuous-integration)

  ### SpringBoot CI/CD Documentation

  #### Example Azure
  ```
  trigger:
  branches:
    include:
      - main

    pool:
    vmImage: 'ubuntu-latest'

    steps:
    - task: UseJavaVersion@1
    inputs:
        versionSpec: '11'
        addToPath: true

    - script: |
        mvn clean install
    displayName: 'Build and Package'

    - task: Maven@3
    inputs:
        mavenPomFile: 'pom.xml'
        options: '-Dmaven.test.failure.ignore=true test'
        javaHomeOption: 'JDKVersion'
        jdkVersionOption: '1.11'
        goals: 'test'
    displayName: 'Run Tests'
  ```

  #### Example GitLab

  ```
  stages:
  - build
  - test

    build:
    stage: build
    image: maven:latest
    script:
        - mvn clean install
    artifacts:
        paths:
        - target/

    test:
    stage: test
    image: maven:latest
    script:
        - mvn test
  ```

  ### Nunit Test CI/CD Examples
   - [Azure](https://learn.microsoft.com/en-us/azure/devops/pipelines/?view=azure-devops&viewFallbackFrom=azure-devops-pipelines-2022-01)

  #### Example

  ```
  trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: UsePythonVersion@0
  inputs:
    versionSpec: '3.x'
    addToPath: true

- script: |
    python -m pip install --upgrade pip
    pip install selenium
    pip install webdriver_manager
  displayName: 'Install Selenium and Webdriver Manager'

- script: |
    wget https://github.com/mozilla/geckodriver/releases/download/v0.30.0/geckodriver-v0.30.0-linux64.tar.gz
    tar -xvzf geckodriver-v0.30.0-linux64.tar.gz
    sudo mv geckodriver /usr/local/bin/
    export PATH=$PATH:/usr/local/bin/geckodriver
  displayName: 'Install GeckoDriver'

- script: |
    wget https://github.com/mozilla/geckodriver/releases/download/v0.30.0/geckodriver-v0.30.0-win64.zip
    unzip geckodriver-v0.30.0-win64.zip
    mv geckodriver.exe /usr/local/bin/
    export PATH=$PATH:/usr/local/bin/geckodriver.exe
  displayName: 'Install GeckoDriver for Windows'

- script: |
    wget https://chromedriver.storage.googleapis.com/92.0.4515.107/chromedriver_linux64.zip
    unzip chromedriver_linux64.zip
    sudo mv chromedriver /usr/local/bin/
    export PATH=$PATH:/usr/local/bin/chromedriver
  displayName: 'Install ChromeDriver'

- script: |
    wget https://chromedriver.storage.googleapis.com/92.0.4515.107/chromedriver_win32.zip
    unzip chromedriver_win32.zip
    mv chromedriver.exe /usr/local/bin/
    export PATH=$PATH:/usr/local/bin/chromedriver.exe
  displayName: 'Install ChromeDriver for Windows'

- script: |
    nunit3-console path/to/your/tests.dll
  displayName: 'Run NUnit tests'

  ```