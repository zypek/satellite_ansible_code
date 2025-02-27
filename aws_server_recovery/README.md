Role Name
=========

# server_snapshot

## Description
------------
This Ansible role assumes an AWS IAM role, retrieves the instance ID, and creates snapshots of all attached EBS volumes for that instance. It leverages Ansible modules to securely manage AWS credentials (via STS) and perform snapshot operations.

## Requirements
------------
The following Ansible collections are required:

- `amazon.aws`
- `ansible.builtin`

The target system must also have:
- Access to instance metadata (if running on EC2).
- Proper IAM permissions (via the assumed role) to describe instances and create snapshots.

## Role Variables
--------------
| Variable Name     | Description                                                         | Default Value | Type   |
|-------------------|---------------------------------------------------------------------|--------------|--------|
| `aws_account_no`  | The AWS Account Number where the IAM role is located               | None         | String |
| `aws_role`        | The IAM Role to assume for AWS operations                          | None         | String |
| `aws_region`      | The AWS Region where the instance and volumes reside               | None         | String |

> **Note:** `instance_id` is retrieved dynamically using the instance metadata and does not need to be provided as a variable.

## Dependencies
------------
This role assumes that the AWS environment has STS enabled and that the role can be assumed with sufficient permissions.

## Example Playbook
----------------

Including an example of how to use this role:

```yaml
---
- name: Create EBS Volume Snapshots
  hosts: all
  become: false
  roles:
    - role: server_snapshot
      vars:
        aws_account_no: "123456789012"
        aws_role: "MySnapshotRole"
        aws_region: "ap-southeast-2"
      tags: server_snapshot

```

Alternatively: 

```yaml
---
- hosts: servers
  tasks:
    - name: Include aws_snapshot role
      ansible.builtin.include_role:
        name: cba.cbc_sap_os_config.aws_snapshot
      vars:
        aws_account_no: "123456789012"
        aws_role: "MySnapshotRole"
        aws_region: "ap-southeast-2"
```
