Role Name
=========

# satellite_check

## Description
------------
This Ansible role verifies connectivity to a Satellite or Foreman server, checks credentials, and displays server status. If connectivity fails (due to network or invalid credentials), the role clearly reports the error.

## Requirements
------------
The following Ansible collections are required:

- `ansible.builtin`
- `redhat.satellite`

Your environment should also have:
- Valid credentials to authenticate with Red Hat Satellite/Foreman.
- Proper networking/ports open to access the Satellite server.

## Role Variables
--------------
| Variable Name                           | Description                                                                | Default Value | Type   |
|----------------------------------------|----------------------------------------------------------------------------|--------------|--------|
| `satellite_deployment_server`          | The Satellite/Foreman server URL                                           | None         | String |
| `satellite_deployment_admin_username`  | Username for Satellite/Foreman authentication                              | None         | String |
| `satellite_deployment_admin_password`  | Password for the Satellite/Foreman user                                   | None         | String |
| `validate_certs` (optional)           | Whether to validate SSL certificates (`true` or `false`)                   | `false`      | Boolean |

## Dependencies
------------
This role assumes:
- The Satellite server is up and reachable.
- The credentials provided have permissions to retrieve server status.

## Example Playbook
----------------

### Using the role within a playbook
```yaml
---
- name: Check Satellite Connectivity
  hosts: all
  become: false
  roles:
    - role: satellite_check
      vars:
        satellite_deployment_server: "https://satellite.example.com"
        satellite_deployment_admin_username: "admin"
        satellite_deployment_admin_password: "redhat"
        validate_certs: false
```
