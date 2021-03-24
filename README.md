# Overview - Laboratory Environment

Ansible Tower as a CI/CD tool for automating continuous deployment of an internal three-tier application on QA and production environments.

OPENTLC services for this automation: https://labs.opentlc.com/

	- Ansible Advanced - Homework (Tower)
	- Ansible Advanced NG - OpenStack (QA: Openstack)
	- Ansible Advanced - Three Tier App (PRO: AWS)


Requirements
=============

You must create a workflow in Ansible Tower (https://tower1.XXXX.example.opentlc.com) with following templates:
- [Ansible Tower templates for RHOSP deployment](#Ansible-Tower-templates-for-RHOSP-deployment)
  - 00_Homework Assignment (SCM update - project)
  - 02_Provision QA Env
  - 04_3tier app deployment on QA Env
  - 05_Smoke test QA Env     
- [Ansible Tower templates for AWS deployment](#Ansible-Tower-templates-for-AWS-deployment)
  - 01_Provision Prod Env
  - 03_06_Prod Three tier inventory source (inventory source)
  - 07_Prod Check the status of AWS instances 
  - 08_Prod SSH keys three tier app 
  - 09_3 tier app on Prod 
  - 10_Smoke test Prod env

And you must create the following Inventories in Tower:
- scm_inventory (RHOSP workstation)
  - workstation-XXXX.dynamic.opentlc.com
- Prod Three tier inventory (AWS EC2 instances)
  - frontend1.XXXX.internal
  - app1.XXXX.internal
  - app2.XXXX.internal
  - appdb1.XXXX.internal

Ansible Tower Config
=============

* From the cloned repo run `site-config-tower.yml` playbook to create job templates and workflow template.

### Playbooks

| Job Template                              | Playbook name          | Purpose                                                                                                       |
|-------------------------------------------|------------------------|---------------------------------------------------------------------------------------------------------------|
| 01_Provision Prod Env                     | aws_provision.yml      | Provision production environment in AWS                                                                       |
| 02_Provision QA Env                       | site-osp-instances.yml | Provision QA environment in OpenStack                                                                         |
| 04_3tier app deployment on QA Env         | site-3tier-app.yml     | Deploy three tier application to QA                                                                           |
| 05_Smoke test QA Env                      | site-smoke-osp.yml     | Playbook to test three tier app on OSP                                                                        |
| 07_Prod Check the status of AWS instances | aws_status_check.yml   | Check aws instances are up or not                                                                             |
| 08_Prod SSH keys three tier app           | aws_creds.yml          | Use ssh private key from AWS bastion host and use it to create machine credential to connect to AWS instances |
| 09_3 tier app on Prod                     | site-3tier-app.yml     | Deploy three tier application to Production                                                                   |
| 10_Smoke test Prod env                    | site-smoketest-aws.yml | test three tier app on AWS                                                                                    |
| Nuke QA Env                               | site-osp-delete.yml    | Delete QA OpenStack instances                                                                                 |
| Executed on controller node only          | site-config-tower.yml  | Configure Ansible Tower with jobs from this project                                                           |
| Executed on controller node only          | site-setup-prereqs.yml | Configure Ansible Tower to use isolated node                                                                  |
| Executed on controller node only          | grading-script.yml     | Self grading script                                                                                           |

### Roles

Basic overview of roles.
Documentation is part of each role.

| Playbook name       | Purpose                                                                  |
|---------------------|--------------------------------------------------------------------------|
| app-tier            | Install application on server                                            |
| db-tier             | Install postgresql server                                                |
| lb-tier             | Install HA proxy server                                                  |
| base-config         | Setup yum repo and base packages role                                    |
| setup-workstation   | Setup workstation, create network, ssh keypair, security group etc. role |
| osp-servers         | Provision OSP Instances role                                             |
| osp-instance-delete | Delete OSP Instances role                                                |
| osp-facts           | Generate in-memory inventory for OSP instances role                      |
| config-tower        | Role to configure ansible tower job templates and workflow               |

Ansible Tower workflow - Roles Execution
----------------------------------------

- In order to deploy this environment and make it available to execute training's Final Lab, it is necessary to follow the next steps (as root) user:

```
$ cd nextgen_ansible_advanced_homework
```

- Set the env variable from ~/labrc file (See Final Lab instructions)
```
$ source ~/labrc
```
- You must copy the OPENTLC key to the control host. The OPENTLC key is the private key that you use to connect to the lab environments from your laptop.
This is, the role 'config-tower' with post-config-tower.yml play uses this private key.

-   Deploy Red Hat Ansible Training Environment
```
$ mv ~/ansible-tower-setup-*/ ~/ansible-tower-setup-latest
$ cp /etc/ansible/hosts ~/ansible-tower-setup-latest/inventory
$ chmod 0400 /root/.ssh/openstack.pem
$ cat hosts -> Check 'workstation' host with your specific GUID value
$ ansible-playbook site-setup-prereqs.yaml -k
```

- Verify a successful connection between the control and workstation hosts using the openstack.pem key:
```
$ ssh -i /root/.ssh/openstack.pem cloud-user@workstation-${OSP_GUID}.${OSP_DOMAIN}
$ exit
```
- From your web browser, connect to tower1.${TOWER_GUID}.example.opentlc.com to verify that the isolated node is operational. Navigate to Instance Groups from the side panel and select osp.
![imagen](https://user-images.githubusercontent.com/74468683/112119327-90d6ad00-8bbd-11eb-9382-db1a59c30877.png)

- Update Ansible Tower WebUI admin password. Reset r3dh4t1! as admin password after sign-in using password provided in email. Update Ansible Tower admin password in tower-cli config.
```
cat ~/.tower_cli.cfg
```
- Playbook to create the job templates and workflow template. This launches "config-tower" Ansible role.
Role to configure ansible tower job templates and CICD workflow
```
ansible-playbook site-config-tower.yml \
      -e tower_GUID=${TOWER_GUID} \
      -e osp_GUID=${OSP_GUID} \
      -e osp_DOMAIN=${OSP_DOMAIN} \
      -e opentlc_login=${OPENTLC_ID} \
      -e path_to_opentlc_key=/root/.ssh/mykey.pem \
      -e param_repo_base=${JQ_REPO_BASE} \
      -e opentlc_password=${OPENTLC_PASSWORD} \
      -e REGION_NAME=${REGION} \
      -e EMAIL=${MAIL_ID} \
      -e github_repo=${GITHUB_REPO} -vv
```

- All of the job templates and workflow are created for you. You could check first for example, you've a template "02_Provision QA Env" which launches "site-osp-instances.yml" Playbook against osp Instance Group.

Provision QA Environment
------------------------

- To provision RHOSP instances, you must check that the keys are present on the control host
```
$ ls -l /root/.ssh/
Sample Output
-r--------. 1 root root 2622 Mar 19 08:43 mykey.pem
-r--------. 1 root root 1679 Mar 19 08:23 openstack.pem
-r--r-----. 1 root root  381 Mar 19 08:23 openstack.pub
```
- Then review Final Lab pre-requisites as Python3 OSP libraries for first Ansible Tower template "02_Provision QA Env" which launches "site-osp-instances.yml" Play using "osp-servers" role. This is, connect to OSP workstation and prepare the machine to run OSP commands (Lab 08_01)
```
python3 -m venv ~/openstack-venv
source ~/openstack-venv/bin/activate
pip install openstacksdk==0.12.0 ansible==2.9.10 cryptography==2.9.2 (editado) 
pip install python-openstackclient==4.0.0 selinux
pip freeze -> to check Python versions

mkdir -p /etc/openstack
cd /etc/openstack/
touch clouds.yaml
vi clouds.yaml -> From http://horizon-XXXX.dynamic.opentlc.com/dashboard/project/api_access with password
ansible localhost -m os_auth -a cloud=openstack -> to check authentication

export OS_CLOUD=openstack -> to test OS CLI
ansible localhost -m os_auth
ansible localhost -m os_user_info
```
- NOTE. These previous OSP steps for workstation is launched with 'setup-workstation' role, using 'site-setup-prereqs.yaml' Playbook from control machine.

- Finally, prepare your workstation for ssh (Lab 09_01). NOTE: This is done by 'osp-servers' role
```
cd ~/.ssh
curl -o openstack.pem http://www.opentlc.com/download/ansible_bootcamp/openstack_keys/openstack.pem
cat << EOF  > ~/.ssh/config
Host *
  User cloud-user
  IdentityFile ~/.ssh/openstack.pem
  ControlMaster auto
  ControlPath /tmp/%h-%r
  ControlPersist 5m
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
EOF
```
- Then launch "02_Provision QA Env" Ansible Tower template, which provisions OSP Instances (frontend, db, app1,app2)

- In this point, you can test "04_3tier app deployment on QA Env" Ansible Tower template, which setups front-end load balancer tier, application servers tier and DB tier, using "site-3tier-app.yml" Play with these roles:
    - base-config
    - app-tier
    - db-tier
    - lb-tier -> NOTE. See 'tasks/main.SolucionAlterntiva' version Playbook with tower-cli check. This is other solution for LoadBalancer for OSP and AWS - get TASK [osp-facts : 
      Add host] as get facts "host_name" value is different in OSP than AWS environment. In first case (OSP), it works with 'inventory_hostname' and in the second case (AWS), it 
      works with 'private_ip_address'.This play uses QA and PRO HA templates:
         - haproxy.cfg.qa.j2
         - haproxy.cfg.pro.j2
      With this version, it's required to get tower-cli installed in workstation OSP to run 'tower-cli job list' command

- Then launch the last Ansible Tower template "05_Smoke test QA Env" for OSP, which runs Smoke tests, with "site-smoke-osp.yml" Play testing frontend and DB services.

Provision PROD Environment
--------------------------

- Before launch "aws_provision.yml" Play, you must run the following commands to use order_svc.sh script against CloudForms Api (opentlc)
- Note. The region must be "na", the same region as AWS credential created in Tower
```
$ mkdir ~/bin
$ wget http://www.opentlc.com/download/ansible_bootcamp/scripts/common.sh
$ wget http://www.opentlc.com/download/ansible_bootcamp/scripts/jq-linux64 -O ~/bin/jq
$ wget http://www.opentlc.com/download/ansible_bootcamp/scripts/order_svc.sh
$ chmod +x order_svc.sh ~/bin/jq common.sh

$ touch credential.rc
$ vi credential.rc -> OPENTCL environment variables
$ source credential.rc ;  ./order_svc.sh -y -c 'OPENTLC Automation' -i 'Ansible Advanced - Three Tier App' -t 1 -d 'dialog_expiration=7;region=na;nodes=1;dialog_runtime=8;notes=Training - As part of course'
```
- Then you can launch Ansible Tower "01_Provision Prod Env" and the other templates for PRO environment


### Check Ansible source code

- From control node, launch next playbook to check all components have been deployed correctly.
```
$ OSP_GUID=XXXX
$ ANSIBLE_ADVANCE_GUID=ZZZZ
$ ansible-playbook grading-script.yml \
  -e OSP_GUID=${OSP_GUID} \
  -e ANSIBLE_ADVANCE_GUID=${ANSIBLE_ADVANCE_GUID} \
  -e OSP_DOMAIN=dynamic.opentlc.com
```

## Notes

Connection to prod bastion instance didn't work for AWS user in **08_Prod SSH keys three tier app** template,
To fix this, you could copy manually your ssh keys to bastion host and re-run the workflow from failed step or from start or
you can modify the AWS Credential in Tower configuration changing to basic authentication with password received by email from Cloudforms.

License
-------

BSD

Author Information
------------------
Lara Cancela - PaaS-DevOps/CICD Specialist
lara.cancela@atos.net

