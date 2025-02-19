Role Name
=========

# cluster_unstandby_node

## Description
------------
This Ansible role gathers cluster node details, checks if the current node is a cluster member, determines its role in the cluster, and ensures that the node can be set to unstandby.

## Requirements
------------
The following Ansible collections are required:

- `ansible.builtin`

The target system must have:
- `crm_node` and `pcs` commands available for cluster operations.
- Access to AWS instance metadata (if applicable).

## Role Variables
--------------

| Variable Name        | Description                                            | Default Value | Type   |
|----------------------|--------------------------------------------------------|--------------|--------|
| `hana_pcs_resource`  | The PCS resource name for HANA                        | None         | String |

## Dependencies
------------
This role assumes the `pcs` and `crm_node` commands are available on the target system.

## Example Playbook
----------------

Including an example of how to use this role:

```yaml
---
- name: Ensure Cluster Node is set to unstandby
  hosts: all
  become: true
  roles:
    - role: cluster_unstandby_node
      tags: cluster_unstandby_node

```

Alternatively:

```yaml
---
- hosts: servers
  tasks:
    - name: Set cluster host in unstandby
      ansible.builtin.include_role:
        name: cba.cbc_sap_os_config.cluster_unstandby_node
      vars:
        hana_pcs_resource: # Hana clone resource name
```
