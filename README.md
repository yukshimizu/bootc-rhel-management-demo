# bootc-rhel-management-demo
This repo includes ansible playbooks for a demo project of managing RHEL bootc images with Red Hat Ansible Automation Platform.

## Automating image mode for RHEL with Red Hat Ansible Automation Platform
The goal behind the code is to demonstrate a simple example for automating 
build, test, and deploy operating systems by using image mode for RHEL with Ansible Automation Platform.

## Assumed demo environment
The assumed environment can be set up on AWS EC2 or OCI easily by using playbooks and roles in the [bootc-rhel-management-setup](https://github.com/yukshimizu/bootc-rhel-management-setup) paired repo.

## Included contents
### Playbooks
|Name     |Description|Target  |
|:--------|:----------|:-------|
|`build_image.yml`|Build a new container image and push it to remote registry.|both|
|`create_ami.yml`|Create an AMI image and upload it to AWS.|aws|
|`create_custom_image.yml`|Create a QCOW2 image and import it as OCI Custom Image.|oci|
|`deploy_vm_aws.yml`|Deploy the container image on EC2 as a VM.|aws|
|`deploy_vm_oci.yml`|Deploy the container image on OCI as a VM.|oci|
|`test_app_aws.yml`|Test the application of the VM on EC2.|aws|
|`test_app_oci.yml`|Test the application of the VM on OCI.|oci|
|`update_vm.yml`|Update the VM and reboot it.|both|
|`deregister_ami.yml`|Deregister the AMI.|aws|
|`delete_custom_image.yml`|Delete the OCI Custom Image.|oci|

### Group variables
These variables have already been set as follows, corresponding to the setup by [bootc-rhel-management-setup](https://github.com/yukshimizu/bootc-rhel-management-setup). You can adjust them based on your environment. When using AWS, comment out `oci_` variables and set up `aws_` variables.
```
---
# ---- Common ----
purpose: demo # should not be modified

# ---- bootc VM ----
bootc_test_container: "rhel9-bootc" # adjust with your preference
bootc_builder_image: "registry.redhat.io/rhel9/bootc-image-builder:9.4" # should not be modified
bootc_vm_type: bootc # should not be modified
bootc_vm_name: bootc01 # should not be modified

# ---- OCI Identity ----
oci_availability_domain: hoge:AP-TOKYO-1-AD-1     # REQUIRED: e.g. "IyFQ:AP-TOKYO-1-AD-1"

# ---- Object Storage (replaces S3) ----
oci_bucket_name: rhel9-bootc
oci_dynamic_group_name: bootc-builders
oci_policy_name: bootc-object-storage-policy

# ---- OCI Network ----
oci_vcn_name: demo_vcn
oci_subnet_name: demo_subnet

# ---- OCI Instance ----
oci_bootc_instance_image_name: rhel9-bootc-x86
oci_bootc_instance_shape: VM.Standard.E5.Flex
oci_bootc_shape_config:
  ocpus: 1
  memory_in_gbs: 8

bootc_build_dir: "/home/cloud-user/bootc" # should be modified from cloud-user to ec2-user for AWS
ansible_user: cloud-user # should be modified from cloud-user to ec2-user for AWS

# ---- AWS ----
# aws_region: ap-northeast-1 # needs to be aligned with your environment's creation
# aws_bootc_instance_ami: rhel9-bootc-x86 # adjust with your preference
# aws_bootc_instance_size: t2.small # can be bigger instance size
# aws_s3_bucket: rhel9-bootc # needs to be aligned with your environment's creation
# aws_vpc: demo_vpc # needs to be aligned with your environment's creation
# aws_vpc_subnet_name: demo_subnet # needs to be aligned with your environment's creation
# aws_securitygroup_name: demo_sg # needs to be aligned with your environment's creation
```

### requirements.yml
`collections/requirements.yml` should be adjusted for usage whether on AWS or on OCI. 
```
---
collections:
#  - name: amazon.aws          # For AWS
  - name: oracle.oci          # For OCI
  - name: containers.podman
  - name: community.general
```

### Containerfile
`files/Containerfile` is used in the playbooks. Since the current Containerfile is for OCI, change the file name `Containerfile_aws` to `Containerfile` for AWS and push it.

## Installation and usage
Assuming the demo environement has been already created by the way of before mentioned and you've also performed `git clone` this repo.

- For AWS setup: [SETUP_AWS.md](SETUP_AWS.md)
- For OCI setup: [SETUP_OCI.md](SETUP_OCI.md)
