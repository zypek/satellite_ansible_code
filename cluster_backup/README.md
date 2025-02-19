Role Name
=========

# cluster_backup

## Description
------------
This Ansible role ensures that Pacemaker and Corosync are running, backs up the cluster configuration using the `pcs` command, verifies backup integrity, and rotates old backups older than 7 days.

## Requirements
------------
The following Ansible collections are required:

- `ansible.builtin`

The target system must have:
- Pacemaker and Corosync installed and running.
- The `pcs` command installed and in the system's PATH.
- The `date` command for generating timestamps.

## Role Variables
--------------
| Variable Name         | Description                                                 | Default Value | Type   |
|-----------------------|-------------------------------------------------------------|--------------|--------|
| `cluster_backup_dir`  | The directory where the PCS backup files will be stored.   | None         | String |

## Dependencies
------------
This role assumes the `pcs`, `pacemaker`, and `corosync` services are available on the target system.

## Example Playbook
----------------

Including an example of how to use this role:

```yaml
---
- name: Backup Pacemaker/Corosync Configuration
  hosts: all
  become: true
  roles:
    - role: pcs_config_backup
      vars:
        cluster_backup_dir: /var/backups/cluster
      tags: pcs_backup

```

```yaml
---
- hosts: servers
  tasks:
  - name: Backup with pcs_config_backup
    ansible.builtin.include_role:
      name: cba.cbc_sap_os_config.pcs_config_backup
    vars:
      cluster_backup_dir: /var/backups/cluster #This can be a NFS directory
```
