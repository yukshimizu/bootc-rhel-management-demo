# bootc-rhel-management-demo
This repo includes ansible playbooks for a demo project of managing RHEL bootc images with Red Hat Ansible Automation Platform on AWS EC2.

## Automating image mode for RHEL with Red Hat Ansible Automation Platform
The goal behind the code is to demonstrate a simple example for automating 
build, test, and deploy operating systems by using image mode for RHEL with Ansible Automation Platform.

## Assumed demo environment
The assumed environment can be set up on AWS EC2 easily by using playbooks and roles in the [bootc-rhel-management-setup](https://github.com/yukshimizu/bootc-rhel-management-setup) paired repo.

## Included contents
### Playbooks
|Name     |Description|
|:--------|:----------|
|`build_image.yml`|Build a new container image and push it to remote registry.|
|`create_ami.yml`|Create an AMI image and upload it to AWS.|
|`deploy_vm.yml`|Deploy the container image on EC2 as a VM.|
|`test_app.yml`|Test the application of the VM.|
|`update_vm.yml`|Update the VM and reboot it.|
|`deregister_ami.yml`|Deregister the AMI.|

### Group variables
These variables have already been set as follows, corresponding to the setup by [bootc-rhel-management-setup](https://github.com/yukshimizu/bootc-rhel-management-setup). You can adjust them based on your environment.
```
---
aws_region: ap-northeast-1 # needs to be aligned with your environment's creation
aws_bootc_instance_ami: rhel9-bootc-x86 # adjust with your preference
aws_bootc_instance_size: t2.small # can be bigger instance size
aws_s3_bucket: rhel9-bootc # needs to be aligned with your environment's creation
aws_vpc: demo_vpc # needs to be aligned with your environment's creation
aws_vpc_subnet_name: demo_subnet # needs to be aligned with your environment's creation
aws_securitygroup_name: demo_sg # needs to be aligned with your environment's creation

purpose: demo # should not be modified

bootc_build_dir: "/home/ec2-user/bootc" # should not be modified
bootc_test_container: "rhel9-bootc" # adjust with your preference
bootc_builder_image: "registry.redhat.io/rhel9/bootc-image-builder:latest" # should not be modified
bootc_vm_type: bootc # should not be modified
bootc_vm_name: bootc01 # should not be modified
bootc_vm_environment: apdb # should not be modified
```

## Installation and usage
Assuming the demo environement has been already created by the way of before mentioned and you've also performed `git clone` this repo.

### Manually
Ensure that you are logged in to your Ansible Automation Controller before proceeding with the following steps.

### Create credentials
At leaset the following two credentails need to be defined.

#### Credential for AWS
1. Click `Credentials` in the left menu.
2. Click `Add` button.
3. Enter the following fields:
   - Name: `aws_cred`
   - Credential Type: `Amazon Web Services`
   - Access Key: your AWS_ACCESS_KEY_ID
   - Secret Key: your AWS_SECRET_ACCESS_KEY
4. Click `Save` button.

Please refer to [Ansible Doc](https://docs.ansible.com/automation-controller/latest/html/userguide/credentials.html#amazon-web-services) for more details.

#### Credential for ssh to AWS instances
1. Click `Credentials` in the left menu.
2. Click `Add` button.
3. Enter the following fields:
   - Name: `aws_key`
   - Credential Type: `Machine`
   - SSH Private Key: your AWS private key
4. Click `Save` button.

Please refer to [Ansible Doc](https://docs.ansible.com/automation-controller/latest/html/userguide/credentials.html#machine) for more details.

### Create inventories
1. Click `Inventories` in the left menu.
2. Click `Add` button and select `Add inventory`.
3. Enter the following fields:
   - Name: `RHEL_Demo`
4. Click `Save` button and then select `Sources` tab.
5. Click `Add` button.
6. Enter the following fields:
   - Name: `AWS`
   - Source: `Amazon EC2`
   - Credential: `aws_cred`
   - Update options: `Overwrite`, `Overwrite variables`, `Update on launch`
   - Source variables:
   ```
   ---
   # Minimal example using environment variables
   # Fetch all hosts taged with "purpose" tag as "demo" in ap-northeast-1
   
   plugin: amazon.aws.aws_ec2
   keyed_groups:
     - prefix: tag
       key: tags

   # Change regions corresponding to your environment
   regions:
     - ap-northeast-1 # adjust with your preference

   # Filter only objects taged with "purpose" tag as "demo"
   filters:
     tag:purpose: demo

   # Ignores 403 errors rather than failing
   strict_permissions: false
   ```
7. Click `Save` button.

### Create a project
1. Click `Projects` in the left menu.
2. Click `Add` button.
3. Enter the following fields:
   - Name: `RHEL_Bootc_Demo`
   - Organization: `Default` (or your prefered organization)
   - Execution Environment: `Default execution environment`
   - Source Control Type: `Git`
   - Source Control URL: your git repositoriy
4. Click `Save` button

Please refer to [Ansible Doc](https://docs.ansible.com/automation-controller/latest/html/userguide/projects.html#add-a-new-project) for more details.

### Create job templates
Each job template is equivalent to a playbook in this repository. Repeat these steps for each template/playbook that you want to use and change the variables specific to the individual playbook. Please refer to [Ansible Doc](https://docs.ansible.com/automation-controller/latest/html/userguide/job_templates.html) for more details.

1. Click `Templates` in the left menu.
2. Click `Add` button and select `Add job template`.
3. Follow the next steps respectively.
4. Click `Save` button

#### Build Bootc Image
- Name: `Build Bootc Image`
- Job Type: `Run`
- Inventory: `RHEL_Demo`
- Project:  `RHEL_Bootc_Demo`
- Playbook: `build_image.yml`
- Credentials: `aws_key`
- Survey Varibales: The followings should be configured and passed via survey. Please note that the survey needs to be disabled when running the job within the worflow jobs described later.
    ```
    ---
    bootc_image_name:
    bootc_image_tag:
    bootc_page_title:
    bootc_remote_registry:
    rhsm_username:
    rhsm_passwd:
    registry_username:
    registry_passwd:
    ```

#### Create Bootc AMI
- Name: `Create Bootc AMI`
- Job Type: `Run`
- Inventory: `RHEL_Demo`
- Project:  `RHEL_Bootc_Demo`
- Playbook: `create_ami.yml`
- Credentials: `aws_cred` `aws_key`
- Survey Varibales: The followings should be configured and passed via survey. Please note that the survey needs to be disabled when running the job within the worflow jobs described later.
    ```
    ---
    bootc_image_name:
    bootc_image_tag:
    bootc_remote_registry:
    rhsm_username:
    rhsm_passwd:
    registry_username:
    registry_passwd:
    ```

#### Deploy Bootc VM
- Name: `Deploy Bootc VM`
- Job Type: `Run`
- Inventory: `RHEL_Demo`
- Project:  `RHEL_Bootc_Demo`
- Playbook: `deploy_vm.yml`
- Credentials: `aws_cred`
- Survey Varibales: The followings should be configured and passed via survey. Please note that the survey needs to be disabled when running the job within the worflow jobs described later.
    ```
    ---
    aws_keypair_name:
    ```

#### Test Bootc App
- Name: `Test Bootc App`
- Job Type: `Run`
- Inventory: `RHEL_Demo`
- Project:  `RHEL_Bootc_Demo`
- Playbook: `test_app.yml`
- Credentials: `aws_cred`
- Survey Varibales: The followings should be configured and passed via survey. Please note that the survey needs to be disabled when running the job within the worflow jobs described later.
    ```
    ---
    bootc_page_title:
    ```

#### Update Bootc VM
- Name: `Update Bootc VM`
- Job Type: `Run`
- Inventory: `RHEL_Demo`
- Project:  `RHEL_Bootc_Demo`
- Playbook: `update_vm.yml`
- Credentials: `aws_key`
- Variables:
    ```
    ---
    target_host: tag_Name_bootc01
    ```
- Survey Varibales: The followings should be configured and passed via survey. Please note that the survey needs to be disabled when running the job within the worflow jobs described later.
    ```
    ---
    bootc_image_name:
    bootc_image_tag:
    bootc_remote_registry:
    ```

#### Deregister Bootc AMI
- Name: `Deregister Bootc AMI`
- Job Type: `Run`
- Inventory: `RHEL_Demo`
- Project:  `RHEL_Bootc_Demo`
- Playbook: `deregister_ami.yml`
- Credentials: `aws_cred`

### Create workflow templates
Above job templates are acutually configured as separate workflow templates for initial creation and for updates respectively. Follow the next steps for each environment. Please refer to [Ansible Doc](https://docs.ansible.com/automation-controller/latest/html/userguide/workflow_templates.html) for more details.

#### Create the Bootc Instance

1. Click `Templates` in the left menu.
2. Click `Add` button and select `Add workflow template`.
3. Click `Save` button.
4. Click `Start` and launch Visualizer.
5. Configure the workflow template as follows:

6. Click `Save` button.
7. Click `Survery` tab and click `Add` button.
8. Add the following two surveys and enable them:
    - Bootc Image Name
        - Type: text
        - Answer variable name: `bootc_image_name`
        - Example: `namespace/rhel9-bootc`
    - Bootc Image Tag
        - Type: text
        - Answer variable name: `bootc_image_tag`
        - Example: `1.0`
    - Bootc Page Title
        - Type: text
        - Answer variable name: `bootc_page_title`
        - Example: `bootc-http-v1` # needs to be aligned with the Containerfile
    - Bootc Remote Registry
        - Type: text
        - Answer variable name: `bootc_remote_registry`
        - Example: `quay.io`
    - RHSM Username
        - Type: text
        - Answer variable name: `rhsm_username`
        - Example: `name@example.com`
    - RHSM Password
        - Type: password
        - Answer variable name: `rhsm_passwd`
    - Registry Username
        - Type: text
        - Answer variable name: `registry_username`
        - Example: `name@example.com`
    - Registry Password
        - Type: password
        - Answer variable name: `registry_passwd`
    - AWS Keypair Name
        - Type: text
        - Answer variable name: `aws_keypair_name`
        - Example: `keypairXYZ`

NOTE: Although `xxx_passwd` should be encrypted in production, using vault for example, I just use easier way for demo purpose.

#### Update the Bootc Instance

1. Click `Templates` in the left menu.
2. Click `Add` button and select `Add workflow template`.
3. Click `Save` button.
4. Click `Start` and launch Visualizer.
5. Configure the workflow template as follows:

6. Click `Save` button.
7. Click `Survery` tab and click `Add` button.
8. Add the following two surveys and enable them:
    - Bootc Image Name
        - Type: text
        - Answer variable name: `bootc_image_name`
        - Example: `namespace/rhel9-bootc`
    - Bootc Image Tag
        - Type: text
        - Answer variable name: `bootc_image_tag`
        - Example: `2.0`
    - Bootc Page Title
        - Type: text
        - Answer variable name: `bootc_page_title`
        - Example: `bootc-http-v2` # needs to be aligned with the Containerfile
    - Bootc Remote Registry
        - Type: text
        - Answer variable name: `bootc_remote_registry`
        - Example: `quay.io`
    - RHSM Username
        - Type: text
        - Answer variable name: `rhsm_username`
        - Example: `name@example.com`
    - RHSM Password
        - Type: password
        - Answer variable name: `rhsm_passwd`
    - Registry Username
        - Type: text
        - Answer variable name: `registry_username`
        - Example: `name@example.com`
    - Registry Password
        - Type: password
        - Answer variable name: `registry_passwd`

NOTE: Before running the workflow for update, **your image repository needs to be marked as public**.
