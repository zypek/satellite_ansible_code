Role Name
=========

# cluster_node_start

Description
------------
This Ansible role starts a Pacemaker cluster, verifies if the cluster services are started, and asserts their status. It ensures a clean startup of the cluster and prevents services.

Role Variables
--------------


| Variable Name        | Description                                            | Default Value | Type   |
|----------------------|--------------------------------------------------------|--------------|--------|
| `system_services_list` | List of cluster-related services to check | None | List  |


Dependencies
------------

This role assumes the `pcs` command is available on the target system.

Example Playbook
----------------

Including an example of how to use this role:

```yaml
---
- name: Start Pacemaker Cluster
  hosts: all
  become: true
  roles:
    - role: cluster_node_start
      tags: cluster_node_start
```

Alternatively:

```yaml
---
- hosts: servers
  tasks:
    - name: Amend no_proxy
      ansible.builtin.include_role:
        name: cba.cbc_sap_os_config.cluster_node_start
      vars:
        system_services_list:
          - pacemaker
          - corosync
```
