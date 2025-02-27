Role Name
=========

# aws_server_recovery

## Description
------------
This Ansible role manages AWS snapshots for EC2 instances by assuming an AWS IAM role, retrieving instance details, creating snapshots of attached EBS volumes, cleaning up old snapshots, and restoring instances from snapshots when needed.

## Requirements
------------
The following Ansible collections are required:

- `amazon.aws`
- `ansible.builtin`

The following roles are required: 

- `aws_role_assume`
- `aws_instance_info`

The target system must also have:
- Access to instance metadata (if running on EC2).
- Proper IAM permissions (via the assumed role) to describe instances, create, delete, and restore snapshots.

## Role Variables
--------------
| Variable Name          | Description                                                       | Default Value | Type   |
|------------------------|-------------------------------------------------------------------|--------------|--------|
| `aws_account_no`       | AWS Account Number where the IAM role is located                 | None         | String |
| `aws_role`            | IAM Role to assume for AWS operations                            | None         | String |
| `aws_region`          | AWS Region where the instance and volumes reside                 | None         | String |
| `aws_snapshot_create` | Boolean flag to trigger snapshot creation                        | `false`      | Bool   |
| `aws_snapshot_cleanup`| Boolean flag to trigger snapshot cleanup                         | `false`      | Bool   |
| `aws_snapshot_delete` | Boolean flag to trigger snapshot deletion                        | `false`      | Bool   |
| `aws_server_restore`  | Boolean flag to trigger instance restoration from snapshots      | `false`      | Bool   |
| `aws_snapshot_id`     | Snapshot ID to delete                                            | None         | String |

> **Note:** `instance_id` and `availability_zone` are dynamically retrieved using the instance metadata and do not need to be provided as variables.

## Dependencies
------------
This role assumes that the AWS environment has STS enabled and that the role can be assumed with sufficient permissions.

## Examples
----------------
Including an example of how to use this role:
### Create a Snapshot

With the role configured as below, the ansible will connect to the remote host, it will retrieve the instance_id, aws_region, aws_availability_zone etc using the metadata call. All the variables required for the Snapshot will be set usingset_fact. The aws_role_assume will generate temporary credentials. The required variables will be passed to the included task. The tasks included in aws_snapshot_create.yml will:

- Retrieve Attached Volume IDs for the instance
- Create Snapshot for all attached volumes 

**Example Code**

```yaml
---
- name: Manage AWS Snapshots
  hosts: all
  become: false
  roles:
    - role: aws_snapshot_management
      vars:
        aws_account_no: "123456789012"
        aws_role: "MySnapshotRole"
        aws_region: "ap-southeast-2"
        aws_snapshot_create: true
```
### Delete a Snapshot

With the role configured as below, the ansible will connect to the remote host, it will retrieve the instance_id, aws_region, aws_availability_zone etc using the metadata call. All the variables required for the Snapshot will be set usingset_fact. The aws_role_assume will generate temporary credentials. The required variables will be passed to the included task. The tasks included in aws_snapshot_delete.yml will:

- Verify if the variable `aws_snapshot_id` has been defined and is not null
- Verify if the snapshot exists in AWS 
- Delete the snapshot

**Example Code**

```yaml
---
- name: Manage AWS Snapshots
  hosts: all
  become: false
  roles:
    - role: aws_snapshot_management
      vars:
        aws_account_no: "123456789012"
        aws_role: "MySnapshotRole"
        aws_region: "ap-southeast-2"
        aws_snapshot_delete: true
```

### Snapshot Cleanup

With the role configured as below, the ansible will connect to the remote host, it will retrieve the instance_id, aws_region, aws_availability_zone etc using the metadata call. All the variables required for the Snapshot will be set usingset_fact. The aws_role_assume will generate temporary credentials. The required variables will be passed to the included task. The tasks included in aws_snapshot_cleanup.yml will:

- Gather AWS instance info
- Extract volume IDs for all the volumes attached to the instance
- Cleanup the snapshots leaving the latest two snapshots for each volume

**Example Code**

```yaml
---
- name: Manage AWS Snapshots
  hosts: all
  become: false
  roles:
    - role: aws_snapshot_management
      vars:
        aws_account_no: "123456789012"
        aws_role: "MySnapshotRole"
        aws_region: "ap-southeast-2"
        aws_snapshot_cleanup: true
```
### AWS Instance Recovery from Snapshot

With the role configured as below, the ansible will connect to the remote host, it will retrieve the instance_id, aws_region, aws_availability_zone etc using the metadata call. All the variables required for the Snapshot will be set usingset_fact. The aws_role_assume will generate temporary credentials. The required variables will be passed to the included task. The tasks included in aws_server_restore.yml will:

- Gather all attached volumes
- Extract root device
- Find latest snapshots for each device. If the snapshot is not found for any of the devices, the play fails
- Create new volumes from snapshots
- Stop the EC2 instance
- Detach old volumes
- Attach new volumes
- Start the EC2 instance

**Example Code**

```yaml
---
- name: Manage AWS Snapshots
  hosts: all
  become: false
  roles:
    - role: aws_snapshot_management
      vars:
        aws_account_no: "123456789012"
        aws_role: "MySnapshotRole"
        aws_region: "ap-southeast-2"
        aws_server_restore: true
```
