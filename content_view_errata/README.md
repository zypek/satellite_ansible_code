Role Name
=========

# content_view_errata

## Description
------------
This Ansible role creates or updates a Content View filter rule in Red Hat Satellite/Foreman to include errata (by date) in a specified Content View. The filter can be further refined to include specific types of errata, such as bugfix or security updates.

## Requirements
------------
The following Ansible collections are required:

- `redhat.satellite`
- `redhat.satellite_operations`
- `ansible.builtin`

Your system must have:
- Correct credentials to authenticate with Satellite/Foreman (`satellite_deployment_admin_username` and `satellite_deployment_admin_password`).
- An existing Content View (`satellite_content_view`) and associated filter name to update (`errata_filter_name`).

## Role Variables
--------------
| Variable Name                          | Description                                                             | Default Value | Type   |
|---------------------------------------|-------------------------------------------------------------------------|--------------|--------|
| `satellite_deployment_admin_username` | Username for Satellite/Foreman authentication                          | None         | String |
| `satellite_deployment_admin_password` | Password for Satellite/Foreman user                                    | None         | String |
| `satellite_deployment_server`         | The Satellite/Foreman server URL                                       | None         | String |
| `satellite_content_view`              | Name of the Content View to which the errata filter applies            | None         | String |
| `satellite_deployment_organization`   | The organisation in which the Content View resides                     | None         | String |
| `errata_filter_name`                  | The name of the Content View filter to update with errata rules        | None         | String |
| `errata_end_date`                     | The end date for errata inclusion (in `YYYY-MM-DD` or iso8601 format)  | None         | String |

## Dependencies
------------
This role assumes:
- The relevant Satellite/Foreman environment is set up with the content view and filter created (the filter will be updated to add errata by date).
- The user has permissions to modify content view filter rules.

## Example Playbook
----------------

### Using the Role Directly
```yaml
---
- name: Update Content View with Errata Rule
  hosts: all
  become: false
  roles:
    - role: content_view_errata
      vars:
        satellite_deployment_admin_username: "admin"
        satellite_deployment_admin_password: "redhat"
        satellite_deployment_server: "https://satellite.example.com"
        satellite_content_view: "rhel8_sap_cv"
        satellite_deployment_organization: "sap"
        errata_filter_name: "My_Errata_Filter"
        errata_end_date: "2025-12-31"
      tags: content_view_errata

```

Alternatively: 

```yaml
---
- hosts: servers
  tasks:
    - name: Include content_view_errata role
      ansible.builtin.include_role:
        name: cba.cbc_sap_os_config.content_view_errata
      vars:
        satellite_deployment_admin_username: "admin"
        satellite_deployment_admin_password: "redhat"
        satellite_deployment_server: "https://satellite.example.com"
        satellite_content_view: "rhel8_sap_cv"
        satellite_deployment_organization: "sap"
        errata_filter_name: "My_Errata_Filter"
        errata_end_date: "2025-12-31"
```
