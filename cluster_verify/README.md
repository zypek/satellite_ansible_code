Role Name
=========

# cluster_verify

## Description
------------
This Ansible role verifies that all critical cluster services (Pacemaker, Corosync, HANA resources, and fencing resources) are running and healthy. It checks for any lost nodes, ensures the fencing resource is active, and confirms that the HANA resource is promoted on the master node.

## Requirements
------------
The following Ansible collections are required:

- `ansible.builtin`

The target system must have:
- `crm_node` and `pcs` commands available for cluster operations.
- Properly configured Pacemaker/Corosync cluster.

## Role Variables
--------------
| Variable Name             | Description                                                | Default Value | Type   |
|---------------------------|------------------------------------------------------------|--------------|--------|
| `ha_system_services_list` | List of cluster-related services to verify are running    | None         | List   |
| `fence_resource_name`     | Name of the fencing resource in the cluster               | None         | String |
| `hana_pcs_resource`       | Name of the HANA PCS resource to check status for         | None         | String |

## Dependencies
------------
This role assumes that the `pcs`, `crm_node`, `pacemaker`, and `corosync` packages (and services) are available and installed on the target system.

## Example Playbook
----------------

Including an example of how to use this role:

```yaml
---
- name: Verify Cluster Health
  hosts: all
  become: true
  roles:
    - role: cluster_verify
      tags: cluster_verify
```

Alternatively: 

```yaml
---
- hosts: servers
  become: true
  tasks:
  - name: Include cluster_verify role
    ansible.builtin.include_role:
      name: cba.cbc_sap_os_config.cluster_verify
    vars:
      ha_system_services_list:
        - pacemaker
        - corosync
      fence_resource_name: fence_device
      hana_pcs_resource: hana_clone

```
