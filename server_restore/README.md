Role Name
=========

# server_restore

## Description
------------
This Ansible role stops an AWS EC2 instance, checks for the most recent snapshots of each attached EBS volume (and fails if none are found for any attached volume), creates new volumes from these snapshots, and reattaches them to the instance. It then restarts the instance with the restored volumes.

## Requirements
------------
The following Ansible collections are required:

- `amazon.aws`
- `ansible.builtin`

The target system or control node must have:
- Valid AWS credentials with permission to assume the specified role and manage EC2 instances/volumes.
- Network access to the instance metadata service if running on EC2.

## Role Variables
--------------
| Variable Name     | Description                                                                 | Default Value | Type   |
|-------------------|-----------------------------------------------------------------------------|--------------|--------|
| `aws_account_no`  | The AWS account number where the IAM role is located.                        | None         | String |
| `aws_role`        | The name of the IAM role to assume for AWS operations.                       | None         | String |
| `aws_region`      | The AWS region where the instance and volumes are located.                   | None         | String |

> **Note:** `instance_id`, `availability_zone`, and `block_device_snapshots` are set automatically during role execution based on the instance's metadata and discovered volumes.

## Dependencies
------------
This role assumes:
- The `sts:AssumeRole` action is allowed for the specified role.
- EC2 permissions to describe, stop, start, create volumes, and attach/detach volumes.
- The `aws_region` variable can be discovered from instance metadata if not provided explicitly.

## Example Playbook
----------------

### Using the Role in a Playbook

```yaml
---
- name: Restore EBS Volumes from Snapshots
  hosts: all
  become: false
  roles:
    - role: server_restore
      vars:
        aws_account_no: "123456789012"
        aws_role: "SomeIAMRole"
        aws_region: "ap-southeast-2"
      tags: server_restore

```

Alternatively: 

```yaml
---
- hosts: servers
  tasks:
    - name: Include server_restore role
      ansible.builtin.include_role:
        name: cba.cbc_sap_os_config.server_restore
      vars:
        aws_account_no: "123456789012"
        aws_role: "SomeIAMRole"
        aws_region: "ap-southeast-2"
```
