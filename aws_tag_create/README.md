Role Name
=========

# aws_tag_create

Description
------------
This Ansible role assumes an AWS IAM role, retrieves the instance ID, and applies a custom tag to an EC2 instance.

Requirements
------------

The following ansible collectsions are required:

aws.amazon
cloud.aws_ops
redhat.satellite
redhat.satellite_operations

Role Variables
--------------


| Variable Name      | Description                                           | Default Value  | Type   |
|--------------------|-------------------------------------------------------|---------------|--------|
| `aws_account_no`  | AWS Account Number                                    | None          | String |
| `aws_role`        | IAM Role to assume for tagging operations             | None          | String |
| `aws_region`      | AWS Region where the instance is located              | None          | String |
| `tag_key_value`   | Base key value for tagging                           | None          | String |
| `tag_value`       | Value to assign to the tag                           | None          | String |


Dependencies
------------

The following role can be used to populate the tag_key_value and tag_value variables if the aws_tag_create is used to create the tag containing the Satellite Content View versions 

- role: create_satellite_tag_values

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
---
- name: Update RHEL Servers
  hosts: all
  become: false
  roles:
    - role: aws_tag_create
      tags: aws_tag_create

```

Alternatively: 

```yaml
---
- hosts: servers
  tasks:
    - name: Amend no_proxy
      ansible.builtin.include_role:
        name: cba.cbc_sap_os_config.aws_tag_create
      vars:
        aws_account_no: 
        aws_role:
  	aws_region:
 	tag_key_value:
	tag_value: 

```
