# How-To Jenkins Jobs

[Jenkins Job Builder](http://docs.openstack.org/infra/jenkins-job-builder/index.html)

##Jenkins-Jobs Builder
* Allows SCM of Jenkins Jobs
* Build Jenkins Jobs from CLI, rather than UI
* Does not SCM:
  * Plugins
  * Jenkins Environment
  * Credentials
* Cannot export existing job

##Setup Environment

###Get Jenkins-Jobs-Builder
`pip install jenkins-jobs-builder`

###Configuration file jenkins_jobs.ini
1. Log into Jenkins
2. User >> Configure
3. Show API Token

Config used in examples:
```
[job_builder]
ignore_cache=True
keep_descriptions=False
allow_duplicates=False
recursive=False

[jenkins]
user=jenkins
password=e9492775e67e4163273f34f2aba7efd3
url=http://192.168.122.115:8080
query_plugins_info=True
```

Directory structure in examples:
```
.
├── extras
│   └── include_demonstrations.yaml
├── first_job.yml
├── jenkins_jobs.ini
├── jobs
│   ├── ex1-basic_job.yaml
│   ├── ex2-basic_pipeline.yaml
│   ├── ex3-basic_template.yaml
│   ├── ex4-basic_include.yaml
│   ├── ex5-basic_include_raw.yaml
│   ├── ex6-build_parameters.yaml
│   ├── ex7-build_properties.yaml
│   └── ex8-build_scm.yaml
├── pres
│   └── how-to-JJB.odp
├── README.md
└── scripts
    └── somescript.sh

3 directories, 13 files
```

###Running Jenkins Job builder
* Can point at specific job
  * `jenkins-jobs --conf jenkins_jobs.ini update jobs/ex1-basic_job.yaml`
* Can point at directory of jobs
  * `jenkins-jobs --conf jenkins_jobs.ini update jobs`
* Should run `test` first to ensure valid XML
  * `jenkins-jobs --conf jenkins_jobs.ini test jobs`

##Job Basics
* Basic Types:
  * Freestyle
  * Pipeline
* Writing Features
  * Templates
  * Include
  * Include-Raw

###Freestyle Job
* Default project-type
```
- job:
      name: basic_job
      project-type: freestyle
      builders:
          - shell: 'echo "Hello"'
```

###Pipeline Job
* Using DSL to configure job
```
- job:
      name: test_job
      project-type: workflow
      dsl: |
        //********************
        stage 'PreDeploy'
        //********************
        node('master') {
                println("Hello world")
                sh'hostname'
        }
        //********************
        stage 'Deploy'
        //********************
        node('master') {
                sh'pwd'
        }
        //********************
        stage 'Post-Deploy'
        //********************
        node('master') {
                sh'ls -lah'
        }
```

###Templates
* Multiple similar jobs
```
- project:
      name: template_demonstration
      jobs:
        - 'template-demo-{name}':
                name: template-first
                shell_var: "Hello"
        - 'template-demo-{name}':
                name: template-second
                shell_var: "Goodbye"

- job-template:
    name: 'template-demo-{name}'
    id: 'template-demo-{name}'
    builders:
        - shell: |
            echo "{shell_var}, World!"
```

###Include
* Include separate yaml file
* Replace `!include` with yaml block
* Treats file as block of yaml (parsed)
```
- project:
      name: include_demonstration
      jobs:
        - 'include-demonstration-{name}':
                name: include-first
        - 'include-demonstration-{name}':
                name: include-second

- job-template:
    name: 'include-demonstration-{name}'
    id: 'include-demonstration-{name}'
    VAR1: "Hello"
    VAR2: "World"
    builders:
        !include: extras/include_demonstrations.yaml
```
`extras/include_demonstration.yaml`:
```
- shell: |
    echo "include-test-{name}"
    echo "{VAR1}"
    echo "{VAR2}"
```
###Include Raw
* Include seperate file of any kind
* Replaces `!include-raw` with text from file
* Does not parse
```
- project:
      name: include_raw_demonstration
      jobs:
        - 'include-raw-test-{name}':
                name: include-raw-first
                shell_var: "City"
        - 'include-raw-test-{name}':
                name: include-raw-second
                shell_var: "Country"

- job-template:
    name: 'include-raw-test-{name}'
    id: 'include-raw-test-{name}'
    builders:
        - shell:
            !include-raw:
                - scripts/somescript.sh
```
`scripts/somescript.sh`:
```
echo "Hello {shell_var}"
```

##Job Modules
"The bulk of the job definitions come from the following modules."    
[Jenkins Job Builder Documentation](http://docs.openstack.org/infra/jenkins-job-builder/definition.html#modules)
* ExternalJob Project
* Flow Project
* Freestyle Project
* Matrix Project
* Maven Project
* MultiJob Project
* Workflow Project
* Builders
* Hipchat
* Metadata
* Notifications
* Parameters
* Properties
* Publishers
* Reporters
* SCM
* Triggers
* Wrappers
* Zuul    

*Too many to go over in this presentation, a few examples...*     

* Parameters
* Properties
* Publishers
* SCM
* Triggers
* Wrappers

###Parameters
* Specify build parameters
```
builders:
    - shell: echo Hello, ${PARAM}!
parameters:
    - string:
        name: PARAM
        default: "World"
        description: "A parameter named PARAM to be echoed"
```

###Properties
* Multiple properties for build
  * `build-discarder`
```
properties:                                                       
    - build-discarder:                                            
        days-to-keep: 1                                           
        num-to-keep: 1      
```
###Publishers
*  Actions on completion of job
  * `email`, `artifact-deployer`
```
publishers:
  - email:
      recipients: foo@example.com
```

###SCM
* Specify SCM location
```
scm:
    - git:
        url: https://github.com/dankolbrs/jenkins-jobs-howto.git
        branches:
            - master
```

###Triggers
* Run based on... trigger.
  * github - run on push or PR
  * timed - cron
  * build-result - Result of other build

###Wrappers
* Alter the way a job is run
  * `credentials-binding`
    * Passes credential as environment variable, does not log in output
    * UUID is pulled from the URL of the credentials page in Jenkins
```
- credentials-binding:
    - username-password-separated:
       credential-id: b3e6f337-5d44-4f57-921c-1632d796caae
       username: myUsername
       password: myPassword
```
  * `ssh-credentials`
    * loads in ssh-agent
    * UUID is pulled from the URL of the credentials page in Jenkins
```
wrappers:
  - ssh-agent-credentials:
        users:
            - '44747833-247a-407a-a98f-a5a2d785111c'
```
