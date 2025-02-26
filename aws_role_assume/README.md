Role Name
=========

# aws_role_assume

Description
------------
This Ansible role assumes an AWS IAM role.

Requirements
------------

The following ansible collectsions are required:

aws.amazon
cloud.aws_ops

Role Variables
--------------


| Variable Name      | Description                                           | Default Value  | Type   |
|--------------------|-------------------------------------------------------|---------------|--------|
| `aws_account_no`  | AWS Account Number                                    | None          | String |
| `aws_role`        | IAM Role to assume for tagging operations             | None          | String |
| `aws_region`      | AWS Region where the instance is located              | None          | String |


Example Usage
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
---
- name: Import role
  ansible.builtin.include_role:
    name: aws_role_assume

- name: Verify assumed_role is defined and contains required keys
  ansible.builtin.assert:
    that:
      - assumed_role is defined
      - assumed_role.sts_creds is defined
      - assumed_role.sts_creds.access_key is defined
      - assumed_role.sts_creds.secret_key is defined
      - assumed_role.sts_creds.session_token is defined
    fail_msg: >
      The `assumed_role` variable is not defined or does not contain the required keys.
      Ensure the `aws_role_assume` role is executed successfully.

- name: Create tag on an instance
  amazon.aws.ec2_tag:
    resource: "{{ instance_id }}"
    region: "{{ aws_region | default('ap-southeast-2') }}"
    tags: "{{ { backup_tag_key: backup_tag_value } }}"
    state: present
    access_key: "{{ assumed_role.sts_creds.access_key }}"
    secret_key: "{{ assumed_role.sts_creds.secret_key }}"
    session_token: "{{ assumed_role.sts_creds.session_token }}"
  delegate_to: localhost
```
