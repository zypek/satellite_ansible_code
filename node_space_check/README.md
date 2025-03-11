Role Name
=========

# aws_instance_info


## Description
This Ansible role retrieves metadata from the AWS instance metadata service (`169.254.169.254`) for one or more specified metadata paths. If any retrieval fails, the role gracefully handles the error and stops execution. The retrieved metadata is consolidated into the `metadata_values` fact for subsequent tasks to use.

## Requirements
No special requirements beyond:
- Ansible’s built-in modules (`ansible.builtin.uri`, `ansible.builtin.set_fact`, etc.).

## Variables
- **`metadata_paths`**  
  Example: `['instance-id', 'public-ipv4', 'local-ipv4', 'ami-id', 'placement/availability-zone']`

## Usage Example
In a simple playbook, you might do:

```yaml
---
- name: Retrieve AWS metadata
  hosts: localhost
  become: false
  vars:
    metadata_paths:
      - instance-id
      - public-ipv4
      - hostname
      - local-ipv4
      - 'placement/availability-zone'
  roles:
    - aws_instance_info

- name: Show metadata values
  hosts: localhost
  become: false
  tasks:
    - name: Debug retrieved metadata
      ansible.builtin.debug:
        var: metadata_values
```

Example of using this role in another role: 

```yaml
- name: Import role
  ansible.builtin.include_role:
    name: aws_instance_info

- name: Verify assumed_role is defined and contains required keys
  ansible.builtin.assert:
    that:
      - metadata_values is defined
    fail_msg: >
      The `metadata_values` variable is not defined or does not contain the required keys.
      Ensure the `aws_instance_info` role is executed successfully.

- name: Set Instance ID as Fact
  ansible.builtin.set_fact:
    instance_id: "{{ metadata_values['instance-id'] }}"

```

Example of extracting the AWS Region, AWS AZ and instance_id

```yaml
- name: Import role
  ansible.builtin.include_role:
    name: aws_instance_info

- name: Verify assumed_role is defined and contains required keys
  ansible.builtin.assert:
    that:
      - metadata_values is defined
    fail_msg: >
      The `metadata_values` variable is not defined or does not contain the required keys.
      Ensure the `aws_instance_info` role is executed successfully.

- name: Set Instance ID as Fact
  ansible.builtin.set_fact:
    instance_id: "{{ metadata_values['instance-id'] }}"
    aws_region: "{{ metadata_values['placement/availability-zone'][:-1] }}"
    availability_zone: "{{ metadata_values['placement/availability-zone'] }}"
```
