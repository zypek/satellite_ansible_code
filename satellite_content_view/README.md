Role Name
=========

# satellite_content_view

## Description
------------
This Ansible role publishes and promotes a specified Satellite Content View through one or more lifecycle environments on a Red Hat Satellite/Foreman instance. It uses the [theforeman.foreman](https://galaxy.ansible.com/theforeman/foreman) collection to manage the lifecycle and version of a given Content View.

## Requirements
------------
The following Ansible collections are required:

- `theforeman.foreman`
- `redhat.satellite`
- `redhat.satellite_operations`
- `ansible.builtin`

Additionally, the user/system must have:
- Valid credentials to authenticate against the Satellite/Foreman server.
- The target Content View (`satellite_content_view`) must exist.

## Role Variables
--------------
| Variable Name                          | Description                                                     | Default Value | Type   |
|---------------------------------------|-----------------------------------------------------------------|--------------|--------|
| `satellite_deployment_admin_username` | Username with sufficient rights to manage Content Views         | None         | String |
| `satellite_deployment_admin_password` | Password for the above user                                    | None         | String |
| `satellite_deployment_server`         | The Satellite/Foreman server URL                                | None         | String |
| `satellite_content_view`              | Name of the Content View to be published and promoted           | None         | String |
| `satellite_deployment_organization`   | The organisation where the Content View resides                | None         | String |
| `satellite_lifecycle`                 | A list of lifecycle environments to which the CV is promoted    | None         | List   |

> **Note:** `validate_certs` is set to `false` in the example. If your Satellite/Foreman server uses valid certificates, set `validate_certs: true` for secure communication.

## Dependencies
------------
This role assumes:
- Theforeman/Foreman modules are available (via `theforeman.foreman` collection).
- The specified Content View and lifecycle environments exist.

## Example Playbook
----------------

### Using the role directly
```yaml
---
- name: Publish and Promote a Satellite Content View
  hosts: all
  become: false
  roles:
    - role: satellite_content_view
      vars:
        satellite_deployment_admin_username: "admin"
        satellite_deployment_admin_password: "redhat"
        satellite_deployment_server: "https://satellite.example.com"
        satellite_content_view: "rhel8_comp_cv"
        satellite_deployment_organization: "sap"
        satellite_lifecycle:
          - "Build"
          - "NonProd"
          - "PreProd"
          - "Prod"
      tags: satellite_content_view
```

Alternatively: 

```yaml
---
- hosts: servers
  tasks:
    - name: Publish and Promote the Content View
      ansible.builtin.include_role:
        name: cba.cbc_sap_os_config.satellite_content_view
      vars:
        satellite_deployment_admin_username: "admin"
        satellite_deployment_admin_password: "redhat"
        satellite_deployment_server: "https://satellite.example.com"
        satellite_content_view: "rhel8_comp_cv"
        satellite_deployment_organization: "sap"
        satellite_lifecycle:
          - "Build"
          - "NonProd"
          - "PreProd"
          - "Prod"

