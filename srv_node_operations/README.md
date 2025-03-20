Cluster Node Operations
=======================

# srv_node_operations

## Description
------------
This Ansible role manages server node operations. The role provides the following functionality: 

- File system space check.

This play excludes all the tmp file systems and verifies if there is at least 5% of free space on each file system configured on the server

- Service check

This play checks if all the services are in a running state. The play can be used for checking the services after the server has been restarted, as it waits for the services to transition from starting to running. 

- Node reboot

This play reboots the server using the reboot command and waits for the reboot for a set period of time. The play uses the rescue block, which implements the AWS instance restart, in case the server reboot fails
 

## Requirements
------------
The following Ansible modules and collections are required:

- `ansible.builtin`
- `aws.amazon`
- `cloud.aws_ops`


The target system must have:

## Role Variables
--------------
| Variable Name            | Description                                          | Default Value | Type   |
|--------------------------|------------------------------------------------------|---------------|--------|
| `aws_account_no`         | AWS Account Number                                   | None          | String |
| `aws_role`               | IAM Role to assume for tagging operations            | None          | String |
| `aws_region`             | AWS Region where the instance is located             | None          | String |
| `reboot_timeout`         | Maximum seconds to wait for machine to reboot and respond to a test command.                                | 600          | String |
| `connection_timeout`     | Maximum number of seconds to wait for a connection to happen before closing and retrying.       | 300          | String   |
| `restart_instance_on_failure`     | Enable or disable AWS instance restart in a failure block       | false          | bool   |
| `node_srv_check`         | Enable or disable services check on a server          | false        | bool   |
| `node_space_check`       | Enable or disable file systems space check on a server| false        | bool   |
| `node_reboot`            | Enable or disable systems reboot                      | false        | bool   |

> **Note:** The node_srv_check play requires the variable `srv_check_system_services_list` which is a list of services to check. Example:
> srv_check_system_services_list:
>      - pacemaker
>      - corosync
>      - sshd


## Dependencies
------------
This role requires:

## Examples
----------------
Below are examples of how to use this role:

### Verify Services on the server

Verify the services on a server

**Example Code:**

```yaml
---
- name: Verify Services
  hosts: all
  become: true
  roles:
    - role: srv_node_operations
      vars:
        node_srv_check: true
        srv_check_system_services_list:
          - pacemaker
          - corosync
          - sshd
```

### Verify if there is at least 5% of free space on available file systems

Verify space on file systems

**Example Code:**

```yaml
---
- name: Verify space on file systems
  hosts: all
  become: true
  roles:
    - role: srv_node_operations
      vars:
        node_space_check: true
```

### Reboot the server

Reboot the server using a standard reboot command. Restart the instance in case the reboot fails

**Example Code:**

```yaml
---
- name: Reboot the server
  hosts: all
  become: true
  roles:
    - role: srv_node_operations
      vars:
        node_reboot: true
        reboot_timeout: 600  # 10 min
        connection_timeout: 300  # 5 min
        restart_instance_on_failure: false  # Toggle this to true to enable AWS stop
```

