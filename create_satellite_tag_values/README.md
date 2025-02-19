Role Name
=========

# create_satellite_tag_values

## Description
------------
This Ansible role connects to a Red Hat Satellite server (using the [theforeman.foreman](https://galaxy.ansible.com/theforeman/foreman) collection) to retrieve a specified Content View (CV) with its component content views. It then constructs a composite structure of the CV and all components, finally creating two facts:

- `tag_key_value`: A unique key for tagging (e.g., `Server_update_<instance_id>_<epoch_time>`).
- `tag_value`: A compact update summary string describing the content view and its components.

## Requirements
------------
The following Ansible collections are required:

- `theforeman.foreman`
- `redhat.satellite`
- `redhat.satellite_operations`
- `ansible.builtin`

The target environment must have:
- Credentials for the Satellite/Foreman server.
- The Composite Content View name to query. (rhel8_sap_cv)
- Organisation to which the Content View belongs. (sap)

## Role Variables
--------------
| Variable Name                          | Description                                            | Default Value | Type   |
|---------------------------------------|--------------------------------------------------------|--------------|--------|
| `satellite_deployment_admin_username` | Username with rights to query the Satellite server     | None         | String |
| `satellite_deployment_admin_password` | Password for the Satellite server user                | None         | String |
| `satellite_deployment_server`         | URL of the Satellite/Foreman server                   | None         | String |
| `satellite_content_view`              | Name of the Content View to look up                   | None         | String |
| `satellite_deployment_organization`   | The organisation name where the Content View resides   | None         | String |

> **Note:** The role also uses `instance_id` and `ansible_date_time` facts to construct tagging information. These are typically discovered automatically by Ansible on most systems.

## Dependencies
------------
- A valid Red Hat Satellite/Foreman setup.
- The user must have permissions to read content views and components.

## Example Playbook
----------------

Including an example of how to use this role:

```yaml
---
- name: Gather Satellite CV data and create tagging key/value
  hosts: all
  become: false
  roles:
    - role: create_satellite_tag_values
      vars:
        satellite_deployment_admin_username: "admin"
        satellite_deployment_admin_password: "redhat"
        satellite_deployment_server: "https://satellite.example.com"
        satellite_content_view: "RHEL_CV"
        satellite_deployment_organization: "Default_Organization"
      tags: create_satellite_tag_values
```

Alternatively:

```yaml
---
- hosts: servers
  tasks:
    - name: Include create_satellite_tag_values role
      ansible.builtin.include_role:
        name: cba.cbc_sap_os_config.create_satellite_tag_values
      vars:
        satellite_deployment_admin_username: "admin"
        satellite_deployment_admin_password: "redhat"
        satellite_deployment_server: "https://satellite.example.com"
        satellite_content_view: "rhel8_comp_cv"
        satellite_deployment_organization: "sap"
```
