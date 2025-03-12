Cluster Node Operations
=======================

# cluster_node_operations

## Description
------------
This Ansible role manages cluster operations for Pacemaker clusters. It provides capabilities for verifying cluster status, performing backups, managing nodes in standby/unstandby modes, and starting or stopping cluster services.

## Requirements
------------
The following Ansible modules and collections are required:

- `ansible.builtin`

The target system must have:
- Pacemaker and Corosync services installed and running.
- The `pcs` command-line utility installed and properly configured.

## Role Variables
--------------
| Variable Name            | Description                                          | Default Value | Type   |
|--------------------------|------------------------------------------------------|---------------|--------|
| `cluster_verify`         | Verify cluster health and service status             | `false`       | Bool   |
| `cluster_backup`         | Trigger cluster configuration backup                 | `false`       | Bool   |
| `cluster_backup_dir`     | Directory to store backup files                      | None          | String |
| `cluster_node_standby`   | Put the current node into standby mode               | `false`       | Bool   |
| `cluster_node_unstandby` | Remove the current node from standby mode            | `false`       | Bool   |
| `cluster_node_stop`      | Stop cluster services on the current node            | `false`       | Bool   |
| `cluster_node_start`     | Start cluster services on the current node           | `false`       | Bool   |
| `disable_maintenance_mode`| Unset maintenance mode on a cluster                 | `false`       | Bool   |
| `enable_maintenance_mode`| Set maintenance mode on a cluster                    | `false`       | Bool   |
| `hana_pcs_resource`      | HANA resource identifier in Pacemaker                | None          | String |
| `fence_resource_name`    | Fencing resource name                                | None          | String |
| `ha_system_services_list`| List of HA services to validate running status       | None          | List   |

> **Note:** Variables like `cluster_node_name` and node-related details are dynamically retrieved and do not need to be provided explicitly.

## Dependencies
------------
This role requires:
- Pacemaker and Corosync installed and running.
- `pcs` CLI utility must be available.

## Examples
----------------
Below are examples of how to use this role:

### Verify Cluster Status

Verifies cluster nodes, services, fencing, and HANA resources are operational.

**Example Code:**

```yaml
---
- name: Cluster Operations
  hosts: all
  become: true
  roles:
    - role: cluster_node_operations
      vars:
        cluster_verify: true
        ha_system_services_list:
          - pacemaker
          - corosync
        hana_pcs_resource: "hana_resource"
        fence_resource_name: "fence_vmware"
```

### Backup Cluster Configuration

Creates a backup of the current Pacemaker configuration.

**Example Code:**

```yaml
---
- name: Cluster Operations
  hosts: all
  become: true
  roles:
    - role: cluster_node_operations
      vars:
        cluster_backup: true
        cluster_backup_dir: "/backup/pacemaker"
```

### Set Node into Standby

Places the current node into standby mode.

**Example Code:**

```yaml
---
- name: Cluster Operations
  hosts: all
  become: true
  roles:
    - role: cluster_node_operations
      vars:
        cluster_node_standby: true
        hana_pcs_resource: "hana_resource"
```

### Remove Node from Standby

Removes the current node from standby mode.

**Example Code:**

```yaml
---
- name: Cluster Operations
  hosts: all
  become: true
  roles:
    - role: cluster_node_operations
      vars:
        cluster_node_unstandby: true
```

### Stop Cluster Node Services

Stops all cluster-related services on the current node.

**Example Code:**

```yaml
---
- name: Cluster Operations
  hosts: all
  become: true
  roles:
    - role: cluster_node_operations
      vars:
        cluster_node_stop: true
        system_services_list:
          - pacemaker
          - corosync
```

### Start Cluster Node Services

Starts all cluster-related services on the current node.

**Example Code:**

```yaml
---
- name: Cluster Operations
  hosts: all
  become: true
  roles:
    - role: cluster_node_operations
      vars:
        cluster_node_start: true
        system_services_list:
          - pacemaker
          - corosync
```

### Enable Maintenance Mode on a Cluster

Sets the maintenance-mode to true

**Example Code:**

```yaml
---
- name: Cluster Operations
  hosts: all
  become: true
  roles:
    - role: cluster_node_operations
      vars:
        cluster_maintenance: true 
        enable_maintenance_mode: true
```

### Disable Maintenance Mode on a Cluster

Sets the maintenance-mode to false

**Example Code:**

```yaml
---
- name: Cluster Operations
  hosts: all
  become: true
  roles:
    - role: cluster_node_operations
      vars:
        cluster_maintenance: true 
        disable_maintenance_mode: true
```
