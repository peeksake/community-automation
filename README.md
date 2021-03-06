# CloudPak Community Automation

## Introduction
This repo represents the Community Automation effort where teams can contribute automation to be shared with other teams.  We desided as a guild to use Jenkins and Ansible combination for our implementations.  Jenkins and Ansible details below.

## Prereq
Make sure your ansible client is at version 2.9 or greater

## Play list
| play | Description | status | Comments |
|------|-------------|--------|----------|
|common-services-cat-src-inst-play|Install Common Services Operator Catalog Source|Available|none
|common-service-fyre-play|install csi-cephfs and common services on FYRE| Available | none |
|common-service-play|deploy common-services on any infrastructure|Available|none|
|csi-cephfs-fyre-play|deploy cephfs storage on your fyre cluster|Available | none|
|deploy-ova-vmware-play|deploy a new RHCOS template to VMWare|Available|none|
|request-ocp-fyre-play|deploy an OCP cluster on old fyre and fyre OCP+ beta|Availalbe|none|
|request-ocp-ceph-fyre-play|deploy fyre OCP+beta cluster with cephfs|Availalbe|none|
|request-ocp-cs-install-fyre-play|deploy fyre OCP+beta cluster and install csi-cephfs and common-services|Availalbe|none|
|request-crc-fyre-play|Install Redhat CodeReadyContainer Instance|Availble| none|
|request-ocp-aws-play|deploy an OCP cluster on aws|WIP|none|
|request-ocp-roks-play|deploy an OCP cluster on roks|Available|none|
|request-ocp4-logging-fyre-play|Install OCP logging onto OCP+Beta Fyre clusters|Available| none|
|request-ocp4-logging-play|Install OCP logging onto OCP 4.x clusters|Available| none|
|request-ocpplus-cluster-transfer-fyre-play|Transfer OCP+Beta Cluster|Available| none|
|request-ocs-play|Install Openshift Container Storage AWS or VMware|Available| none|
|recover-machine-config-play|Recover machine-config, not rolling out|Available| none|
|common-service-cat-src-inst-play|Install the Common Services Catalog Source|Available| none|
|request-rhel-jmeter-fyre-play|Install Jmeter on Fyre RHEL 8|Availble| none|


## Supporting Roles
| role | Description | status | Comments |
|------|-------------|--------|----------|
|ocp-login | used when OCP Login is needed for your play | Availalbe | will automatically install oc client |
|oc-client-install|installs oc command| Available | is automatic when using ocp-login role|

## Jenkins

### Create your jenkins file
The "Jenkinsfile" should live in your top play folder  
Here is sample ( [Jenkinsfile-example](ansible/Jenkinsfile-example) ). You will update the paramList and the stage() sections. The rest is static for now.  
```
#! groovy

// Standard job properties
def jobProps = [
  buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '50')),
  disableResume(),
  durabilityHint("PERFORMANCE_OPTIMIZED"),
  [$class: 'RebuildSettings', autoRebuild: false, rebuildDisabled: false]
]

def paramsList = [
    string(name: 'API_Server_URL', defaultValue: "api.", description: 'clustre api server url'),
    string(name: 'API_Server_Port', defaultValue: "6443", description: 'clustrer api serfver port number'),
    string(name: 'cluster_admin', defaultValue: "kubeadmin", description: 'clustrer admin user'),
    password(name: 'cluster_admin_password', description: 'cluster admin password'),
    string(name: 'ocp_client_version', defaultValue: "4.2.0"),
    string(name: 'machine_config', description: 'machine config found from oc get mc, newest version')
  ]
jobProps.push(parameters(paramsList))

properties(jobProps)

timestamps {
  ansiColor('xterm') {
    node ( 'kube_pod_slave' ) {

      checkout scm

      stage('Recover Machine Config') {
        sh """
          set +x # hide sensitive info being echo'd to log
          cp ./ansible/recover-machine-config-play/examples/mc_vars.yml ./ansible/recover-machine-config-play/;\
          cp ./ansible/recover-machine-config-play/examples/inventory ./ansible/recover-machine-config-play/;\
          ansible-playbook -i ./ansible/recover-machine-config-play/inventory \
          ./ansible/recover-machine-config-play/recover-machine-config-play.yml \
          -e kubeadmin_user=${params.cluster_admin} \
          -e kubeadmin_password=${params.cluster_admin_password} \
          -e ocp_api_url=${API_Server_URL}:${API_Server_Port} \
          -e arch="linux" \
          -e ocp_client_version=${ocp_client_version} \
          -e machine_config=${machine_config} -vv
        """.stripIndent()
      }
   }
  }
}
```
### Setting up your jenkins job
[Community Jenkins Server](https://hyc-ibm-automation-guild-team-jenkins.swg-devops.com/)  
Login to jenkins server  
Navigate to "**Cluster Ops**"  
Select "**New Item**" from left nav.  
Name your job.  
Select "**MultiBranch Pipeline**" from list.  
Enter "**community-pipeline-template**" in the "Copy from" field.  
Select "**OK**"

### edit job settings
Select "**Single repository & branch**" from "**Add Source**" dropdown.  
Enter your branch name in the "**Name**" field.  Should be "master" most of the time.  
Update Credentials, using github username and token.  Working to get a FUNCTIONAL_ID.  
Under "**Build Configuration**" update "**Script Path**"  
Select "**Save**"

## Ansible
[Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/index.html)

## Folder Structure
The folder structure was taken from the [ansible best practices document](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)


The following is a snippet to help understand the folder structure and how we are using it.  
We have plays and roles, and each are at the top level under ansible/.  
Play folders end in -play, while the machine role would contain the playname without "-play"    

There will be roles without a corresponding play folder, these roles would be common function that could be shared across plays

To ensure we can load the roles correctly you will notice a symbolic link to the top roles directory.  This is a work around until ansible collections become available.


```
└── ansible
    ├── LICENCE.txt
    ├── README.md
    ├── ansible.cfg
    ├── common-service-play
    │   ├── Jenkinsfile
    │   ├── README.md
    │   ├── common-services.yml
    │   ├── examples
    │   │   ├── cs_vars.yml
    │   │   └── inventory
    │   └── roles -> ../roles
    ├── recover-expired-certificates-play
    │   ├── Jenkinsfile
    │   ├── readme.md
    │   ├── recover-expired-certificates-play.yml
    │   └── roles -> ../roles
    ├── recover-machine-config-play
    │   ├── Jenkinsfile
    │   ├── readme.md
    │   ├── recover-machine-config-play.yml
    │   └── roles -> ../roles
    ├── request-ocp-aws-play
    │   └── roles -> ../roles
    ├── request-ocp-fyre-play
    │   └── roles -> ../roles
    ├── request-crc-fyre-play
    │   └── roles -> ../roles
    ├── request-ocp-roks-play
    │   └── roles -> ../roles
    └── roles
        ├── common-services
        │   ├── README.md
        │   ├── defaults
        │   │   └── main.yml
        │   ├── tasks
        │   │   └── main.yml
        │   └── templates
        │       ├── cs-group.yaml.j2
        │       ├── cs-request.yaml.j2
        │       ├── cs-sub.yaml.j2
        │       ├── cs-validation.bash.j2
        │       └── opencloud-source.yaml.j2
        ├── ocp-login
        │   └── tasks
        │       ├── main.yml
        │       └── ocp-login.yml
        ├── recover-epxired-certificates
        │   ├── defaults
        │   │   └── main.yml
        │   ├── files
        │   ├── meta
        │   │   └── main.yml
        │   ├── tasks
        │   │   ├── main.yml
        │   │   └── recover-expired-certificates.yml
        │   ├── templates
        │   └── vars
        ├── recover-machine-config
        │   ├── defaults
        │   │   └── main.yml
        │   ├── files
        │   ├── meta
        │   │   └── main.yml
        │   ├── readme.md
        │   ├── tasks
        │   │   ├── main.yml
        │   │   └── recover-machine-config.yml
        │   ├── templates
        │   └── vars
        ├── request-ocp-aws
        │   ├── default
        │   ├── readme.md
        │   ├── tasks
        │   └── templates
        ├── request-ocp-fyre
        │   ├── defaults
        │   ├── readme.md
        │   ├── tasks
        │   └── templates
        ├── jmeter
        ├── jmeter_fyrevm
        ├── jmeter_java
        ├── java
        ├── jmeter_prereqs
        └── request-ocp-roks
            ├── defaults
            ├── readme.md
            ├── tasks
            └── templates
```

#
## Common Repositories

Terraform Automation (VMWare, AWS, Google, and Azure)
https://github.ibm.com/ICP-DevOps/tf_openshift_4

Some useful tools (cluster recovery scripts)  
https://github.ibm.com/ICP-DevOps/tf_openshift_4_tools


## ROKS Automation
ROKS info being provided until an ansible solution can be worked out in this community repo  

DTEs ROKS Provisioning Tooling https://github.ibm.com/dte/roksprovisioning - See the README.md and I included an installer script to get started.

The following is used in the above repo for reference:

IBM Cloud Terraform Docker Image: https://hub.docker.com/r/ibmterraform/terraform-provider-ibm-docker/

IBM Cloud Terraform Documentation: https://cloud.ibm.com/docs/terraform?topic=terraform-getting-started
