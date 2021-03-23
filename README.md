Overview - Laboratory Environment
==================================

Ansible Tower as a CI/CD tool for automating continuous deployment of an internal three-tier application on QA and production environments.

OPENTLC services for this automation: https://labs.opentlc.com/

	- Ansible Advanced - Homework (Tower)
	- Ansible Advanced NG - OpenStack (QA: Openstack)
	- Ansible Advanced - Three Tier App (PRO: AWS)


Requirements
------------

You must create a workflow in Ansible Tower (https://tower1.XXXX.example.opentlc.com) with following templates:

	- Ansible Tower templates for RHOSP deployment:
      00_Homework Assignment (SCM update - project)
      02_Provision QA Env
      04_3tier app deployment on QA Env
      05_Smoke test QA Env
      

	- Ansible Tower templates for AWS deployment:
      01_Provision Prod Env
      03_06_Prod Three tier inventory source (inventory source)
      07_Prod Check the status of AWS instances 
      08_Prod SSH keys three tier app 
      09_3 tier app on Prod 
      10_Smoke test Prod env

And you must create the following Inventories in Tower:
	- Ansible Tower templates for OSP and AWS deployments:
      scm_inventory (RHOSP workstation)
        - workstation-XXXX.dynamic.opentlc.com
      Prod Three tier inventory (AWS EC2 instances)
        - frontend1.XXXX.internal
        - app1.XXXX.internal
        - app2.XXXX.internal
        - appdb1.XXXX.internal



Ansible Tower template - Role Execution
----------------------------------------

- In order to deploy this environment and make it available to execute training's Final Lab, it is necessary to follow the next steps: 

```
$ cd nextgen_ansible_advanced_homework
```

- Set the env variable from ~/labrc file (See Final Lab instructions)
```
$ source ~/labrc
```

-   Deploy Red Hat Ansible Training Environment
```
$ mv ~/ansible-tower-setup-*/ ~/ansible-tower-setup-latest
$ cp /etc/ansible/hosts ~/ansible-tower-setup-latest/inventory
$ chmod 0400 /root/.ssh/openstack.pem

$ ansible-playbook site-setup-prereqs.yaml -k
```

- Verify a successful connection between the control and workstation hosts using the openstack.pem key:
```
$ ssh -i /root/.ssh/openstack.pem cloud-user@workstation-${OSP_GUID}.${OSP_DOMAIN}
$ exit
```
- From your web browser, connect to tower1.${TOWER_GUID}.example.opentlc.com to verify that the isolated node is operational. Navigate to Instance Groups from the side panel and select osp.
- Update Ansible Tower WebUI admin password. Reset r3dh4t1! as admin password after sign-in using password provided in email. Update Ansible Tower admin password in tower-cli config.
```
cat ~/.tower_cli.cfg
```
- Playbook to create the job templates and workflow template. This launches "config-tower" Ansible role.
Role to configure ansible tower job templates and workflow
```
ansible-playbook site-config-tower.yml -e tower_GUID=${TOWER_GUID} -e osp_GUID=${OSP_GUID} -e osp_DOMAIN=${OSP_DOMAIN} -e opentlc_login=${OPENTLC_ID} -e path_to_opentlc_key=/root/.ssh/mykey.pem -e param_repo_base=${JQ_REPO_BASE} -e opentlc_password=${OPENTLC_PASSWORD} -e REGION_NAME=${REGION} -e EMAIL=${MAIL_ID} -e github_repo=${GITHUB_REPO} -vv
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
- Then review Final Lab pre-requisites as Python3 OSP libraries for first Ansible Tower template "02_Provision QA Env" which launches "site-osp-instances.yml" Play using "osp-servers" role
```
python3 -m venv ~/openstack-venv
source ~/openstack-venv/bin/activate
pip install openstacksdk==0.12.0 ansible==2.9.10 cryptography==2.9.2 (editado) 
pip install python-openstackclient==4.0.0 selinux
pip freeze -> to check Python versions
```
- Then launch "02_Provision QA Env" Ansible Tower template, which provisions OSP Instances (frontend, db, app1,app2)

- In this point, you can test "04_3tier app deployment on QA Env" Ansible Tower template, which setups front-end load balancer tier, application servers tier and DB tier, using "site-3tier-app.yml" Play with these roles:
    - base-config
    - app-tier
    - db-tier
    - lb-tier

- Then launch the last Ansible Tower template "05_Smoke test QA Env" for OSP, which runs Smoke tests, with "site-smoke-osp.yml" Play testing frontend and DB services.

Provision PROD Environment
----------------

- Before launch "aws_provision.yml" Play, you must run the following commands to use order_svc.sh script
against CloudForms (opentlc)
```
$ mkdir ~/bin
$ wget http://www.opentlc.com/download/ansible_bootcamp/scripts/common.sh
$ wget http://www.opentlc.com/download/ansible_bootcamp/scripts/jq-linux64 -O ~/bin/jq
$ wget http://www.opentlc.com/download/ansible_bootcamp/scripts/order_svc.sh
$ chmod +x order_svc.sh ~/bin/jq common.sh

$ touch credential.rc
$ vi credential.rc -> OPENTCL environment variables
$ source credential.rc ;  ./order_svc.sh -y -c 'OPENTLC Automation' -i 'Ansible Advanced - Three Tier App' -t 1 -d 'dialog_expiration=7;region=emea;nodes=1;dialog_runtime=8;notes=Training - As part of course'
```
- Then you can launch Ansible Tower "01_Provision Prod Env" and the other templates for PRO environment


License
-------

BSD

Author Information
------------------
Lara Cancela - PaaS-DevOps/CICD Specialist
lara.cancela@atos.net

