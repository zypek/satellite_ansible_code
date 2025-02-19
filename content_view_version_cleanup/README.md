Role Name
=========

# content_view_version_cleanup

## Description
------------
This Ansible role cleans up old, unused Content View (CV) versions in a Red Hat Satellite environment, both for composite and non-composite content views. It retains a configurable number of the most recent versions, determined by `satellite_content_view_version_cleanup_keep`, and deletes all older versions that are not currently promoted in a lifecycle environment.

> **Note:** The role is looking for ALL the Content Views and prunes old versions for ALL of them


## Requirements
------------
The following Ansible collections are required:

- `redhat.satellite`
- `redhat.satellite_operations`
- `ansible.builtin`

The target environment must have:
- Proper credentials to authenticate and manage content views in Satellite.
- Enough privileges to delete CV versions.

## Role Variables
--------------
| Variable Name                                    | Description                                                                   | Default Value | Type   |
|--------------------------------------------------|-------------------------------------------------------------------------------|--------------|--------|
| `satellite_deployment_server`                    | The Satellite server URL                                                      | None         | String |
| `satellite_deployment_admin_username`            | Username for Satellite server authentication                                  | None         | String |
| `satellite_deployment_admin_password`            | Password for Satellite server authentication                                  | None         | String |
| `satellite_deployment_organization`             | The organisation name in Satellite                                           | None         | String |
| `satellite_content_view_version_cleanup_keep`    | The number of Content View versions to keep (must be >= 0)                   | None         | Integer |

## Dependencies
------------
- The Satellite server must already be set up with the content views you wish to manage.
- The user must have permissions to list and delete content view versions.

## Example Playbook
----------------

### Using the Role
```yaml
---
- name: Cleanup old Content View versions
  hosts: all
  become: false
  roles:
    - role: content_view_version_cleanup
      vars:
        satellite_deployment_server: "https://satellite.example.com"
        satellite_deployment_admin_username: "admin"
        satellite_deployment_admin_password: "redhat"
        satellite_deployment_organization: "sap"
        satellite_content_view_version_cleanup_keep: 2
      tags: content_view_version_cleanup

```

Alternatively: 

```yaml
---
- hosts: servers
  tasks:
    - name: Publish and Promote the Content View
      ansible.builtin.include_role:
        name: cba.cbc_sap_os_config.content_view_version_cleanup
      vars:
        satellite_deployment_admin_username: "admin"
        satellite_deployment_admin_password: "redhat"
        satellite_deployment_server: "https://satellite.example.com"
        satellite_content_view: "rhel8_comp_cv"
        satellite_deployment_organization: "sap"
```
