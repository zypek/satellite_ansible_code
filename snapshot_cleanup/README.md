Role Name
=========

# snapshot_cleanup

## Description
------------
This Ansible role assumes an AWS IAM role to gather information about all EBS volumes attached to the current instance. It then identifies snapshots older than one week for each volume and deletes them—while preserving:
1. The latest snapshot overall.
2. The newest snapshot among the old snapshots.

This ensures that at least two snapshots (the latest one overall and the newest older snapshot) remain for each volume, preventing complete loss of older restore points.

## Requirements
------------
The following Ansible collections are required:

- `amazon.aws`  
- `ansible.builtin`  

Your environment should also have:
- The ability to assume the specified AWS IAM role (`sts:AssumeRole`).
- Sufficient permissions to list, describe, and delete EC2 snapshots.

## Role Variables
--------------
| Variable Name    | Description                                                                 | Default Value | Type   |
|------------------|-----------------------------------------------------------------------------|--------------|--------|
| `aws_account_no` | The AWS account number where the IAM role is located.                        | None         | String |
| `aws_role`       | The IAM role to assume for AWS operations (e.g., snapshot management).       | None         | String |
| `aws_region`     | The AWS region of the instance/volumes. <br>*Automatically discovered from the instance metadata if not provided* | None | String |

> **Note:** `instance_id` and `region` may be dynamically retrieved from the instance metadata service.  
> **Note:** The role uses `ansible_date_time.epoch` to compute "one week ago" for snapshot cleanup logic.

## Dependencies
------------
This role assumes:
- It’s being executed on an EC2 instance capable of reaching the metadata service `169.254.169.254`.
- The `amazon.aws.sts_assume_role`, `amazon.aws.ec2_snapshot_info`, `amazon.aws.ec2_snapshot`, and `amazon.aws.ec2_instance_info` modules can be used successfully with the provided credentials.

## Example Playbook
----------------

### Using the Role Directly
```yaml
---
- name: Cleanup Old EBS Snapshots
  hosts: all
  become: false
  roles:
    - role: snapshot_cleanup
      vars:
        aws_account_no: "123456789012"
        aws_role: "SnapshotCleanupRole"
        # aws_region is optional; the role will detect it via instance metadata
      tags: snapshot_cleanup
```

Alternatively: 

```yaml
---
- hosts: servers
  tasks:
    - name: Include snapshot_cleanup role
      ansible.builtin.include_role:
        name: cba.cbc_sap_os_config.snapshot_cleanup
      vars:
        aws_account_no: "123456789012"
        aws_role: "SnapshotCleanupRole"
        # aws_region is optional; the role will detect it via instance metadata
```
