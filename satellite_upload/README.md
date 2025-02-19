Role Name
=========

# satellite_upload

## Description
------------
This Ansible role downloads a package (e.g., AWS CLI ZIP file) to a local or remote machine, then uploads it to a file-based repository in Red Hat Satellite, and finally removes the temporary downloaded file.

## Requirements
------------
The following Ansible collections are required:

- `ansible.builtin`
- `theforeman.foreman`

The target system (or control node) must have:
- Valid credentials for the Satellite server.
- Access to the specified package URL.
- Write permissions to the specified local temporary path.

## Role Variables
--------------
| Variable Name                         | Description                                                            | Default Value | Type   |
|--------------------------------------|------------------------------------------------------------------------|--------------|--------|
| `package_url`                        | The URL to download the package from (e.g., AWS CLI ZIP)              | None         | String |
| `package_name`                       | The name of the package to be saved locally and uploaded              | None         | String |
| `package_tmp`                        | Temporary directory path where the downloaded file will be stored     | `/tmp`       | String |
| `satellite_deployment_admin_username`| Username with rights to upload to the Satellite server                | None         | String |
| `satellite_deployment_admin_password`| Password for the Satellite server user                                 | None         | String |
| `satellite_deployment_server`        | URL of the Satellite server                                           | None         | String |
| `satellite_deployment_organization`  | The organisation name in Satellite                                    | None         | String |
| `satellite_product`                  | Name of the Satellite Product to which the repository belongs          | None         | String |
| `satellite_repository`               | Name of the Satellite file-based repository                            | None         | String |

## Dependencies
------------
This role assumes that:
- The specified Satellite product and repository already exist.
- The user has permissions to upload content to that repository.

## Example Playbook
----------------

### Using the role within a playbook
```yaml
---
- name: Upload package to Satellite
  hosts: all
  become: false
  roles:
    - role: satellite_upload
      vars:
        package_url: "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip"
        package_name: "awscliv2.zip"
        package_tmp: "/tmp"
        satellite_deployment_admin_username: "admin"
        satellite_deployment_admin_password: "redhat"
        satellite_deployment_server: "https://satellite.example.com"
        satellite_deployment_organization: "Default_Organization"
        satellite_product: "MyProduct"
        satellite_repository: "AWS_CLI_Repo"
```

Alternatively: 

```yaml
---
- hosts: servers
  tasks:
    - name: Include satellite_upload role
      ansible.builtin.include_role:
        name: cba.cbc_sap_os_config.satellite_upload
      vars:
        package_url: "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip"
        package_name: "awscliv2.zip"
        package_tmp: "/tmp"
        satellite_deployment_admin_username: "admin"
        satellite_deployment_admin_password: "redhat"
        satellite_deployment_server: "https://satellite.example.com"
        satellite_deployment_organization: "Default_Organization"
        satellite_product: "MyProduct"
        satellite_repository: "AWS_CLI_Repo"
```
