Role Name
=========

# cluster_node_stop

## Description
------------
This Ansible role stops a Pacemaker cluster, verifies if the cluster services are stopped, and asserts their status. It ensures a clean shutdown of the cluster and prevents services from running unexpectedly.

## Role Variables
--------------

| Variable Name          | Description                                            | Default Value | Type   |
|------------------------|--------------------------------------------------------|--------------|--------|
| `system_services_list` | List of cluster-related services to check             | None         | List   |

## Dependencies
------------
This role assumes the `pcs` command is installed and available on the target system.

## Example Playbook
----------------

```yaml
---
- name: Stop Pacemaker Cluster
  hosts: all
  become: true
  roles:
    - role: cluster_node_stop
      tags: cluster_node_stop
```

Alternatively:

yaml
---
- hosts: servers
  tasks:
    - name: Amend no_proxy
      ansible.builtin.include_role:
        name: cba.cbc_sap_os_config.cluster_node_stop
      vars:
        system_services_list:
          - pacemaker
```
