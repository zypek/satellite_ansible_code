Satellite CV Operations
=======================

# satellite_cv_operations

## Description
------------
This Ansible role manages content view operations for Red Hat Satellite. It supports publishing and promoting content views, managing errata inclusion based on dates, and performing clean-up of unused content view versions.

## Requirements
------------
The following Ansible collections are required:

- `redhat.satellite`
- `ansible.builtin`

The target system must have:
- Red Hat Satellite configured and accessible.
- Proper user credentials and permissions to manage Satellite resources.

## Role Variables
--------------
| Variable Name                                | Description                                                        | Default Value | Type   |
|----------------------------------------------|--------------------------------------------------------------------|---------------|--------|
| `satellite_deployment_admin_username`        | Satellite administrative username                                  | None          | String |
| `satellite_deployment_admin_password`        | Satellite administrative password                                  | None          | String |
| `satellite_deployment_server`                | Satellite server URL                                               | None          | String |
| `satellite_deployment_organization`          | Satellite organization                                             | None          | String |
| `satellite_content_views`                    | List of content views to manage                                    | None          | List   |
| `satellite_content_view`                     | Specific content view name                                         | None          | String |
| `sat_cv_publish`                             | Flag to trigger content view publishing                            | `false`       | Bool   |
| `sat_cv_promote`                             | Flag to trigger content view promotion                             | `false`       | Bool   |
| `satellite_lifecycle`                        | Lifecycle environment for promotion                                | None          | String |
| `sat_cv_filter_errata`                       | Flag to trigger errata filtering                                   | `false`       | Bool   |
| `sat_errata_filter_name`                     | Name of the errata filter to apply                                 | None          | String |
| `sat_errata_end_date`                        | End date for errata inclusion                                      | Current Date  | String |
| `sat_cv_ver_cleanup`                         | Flag to trigger content view version cleanup                       | `false`       | Bool   |
| `satellite_content_view_version_cleanup_keep`| Number of CV versions to retain during cleanup                     | 0             | Int    |

## Dependencies
------------
This role requires:
- Red Hat Satellite server access and appropriate API credentials.

## Examples
----------------
Below are examples of how to use this role:

### Publish and Promote Content View

Publishes and promotes specified content views to a lifecycle environment.

**Example Code:**

```yaml
---
- name: Satellite CV Operations
  hosts: all
  become: false
  roles:
    - role: satellite_cv_operations
      vars:
        satellite_deployment_admin_username: "admin"
        satellite_deployment_admin_password: "password"
        satellite_deployment_server: "https://satellite.example.com"
        satellite_deployment_organization: "Example Org"
        satellite_content_views:
          - "RHEL_Base"
        satellite_content_view: "RHEL_Base"
        sat_cv_publish: true
        sat_cv_promote: true
        satellite_lifecycle: "Production"
```

### Filter Errata by Date

Filters errata by date and applies to a specific content view.

**Example Code:**

```yaml
---
- name: Satellite CV Operations
  hosts: all
  become: false
  roles:
    - role: satellite_cv_operations
      vars:
        satellite_deployment_admin_username: "admin"
        satellite_deployment_admin_password: "password"
        satellite_deployment_server: "https://satellite.example.com"
        satellite_deployment_organization: "Example Org"
        satellite_sap_content_view: "RHEL_SAP"
        sat_cv_filter_errata: true
        sat_errata_filter_name: "filter-security-bugfix"
        sat_errata_end_date: "2024-12-31"
```

### Cleanup Unused CV Versions

Removes unused content view versions beyond a specified retention number.

**Example Code:**

```yaml
---
- name: Satellite CV Operations
  hosts: all
  become: false
  roles:
    - role: satellite_cv_operations
      vars:
        satellite_deployment_admin_username: "admin"
        satellite_deployment_admin_password: "password"
        satellite_deployment_server: "https://satellite.example.com"
        satellite_deployment_organization: "Example Org"
        sat_cv_ver_cleanup: true
        satellite_content_view_version_cleanup_keep: 3
```


