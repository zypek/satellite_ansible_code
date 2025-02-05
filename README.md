**Role server_update**
## Overview

This playbook is designed to manage updates on a RHEL server by performing tasks such as retrieving metadata, creating snapshots of attached volumes, checking filesystem space, updating packages, rebooting the server, verifying critical services, querying Satellite for content view information, and finally tagging the instance with update details. It also cleans up old update tags. The playbook utilises various Ansible modules as well as AWS and Foreman modules to accomplish these tasks.

## Detailed Description

1. **Include Role Variables**  
   - **Task:** Loads variables from the file `vars/all.yml` using the `include_vars` module.  
   - **Purpose:** Ensures that all required variables (e.g., `aws_region`, `satellite_deployment_*`, `system_services_list`) are available for the subsequent tasks.

2. **Retrieve Instance ID from Metadata**  
   - **Task:** Uses the `uri` module to access the metadata URL (`http://169.254.169.254/latest/meta-data/instance-id`) to retrieve the instance ID.  
   - **Purpose:** Captures the current instance’s ID and stores it in `instance_metadata.content` for further processing.

3. **Set Instance ID as Fact**  
   - **Task:** Sets a new fact named `instance_id` with the value retrieved from the metadata.  
   - **Purpose:** Makes the instance ID available to later tasks in the playbook.

4. **Retrieve Attached Volume ID(s)**  
   - **Task:** Uses the `amazon.aws.ec2_instance_info` module to query AWS for details about the instance (using its `instance_id` and `aws_region`).  
   - **Purpose:** Obtains information about block device mappings (EBS volumes) attached to the instance, which is used for snapshot creation.

5. **Create Snapshot of the Root Volume**  
   - **Task:** Uses the `amazon.aws.ec2_snapshot` module to create snapshots for each EBS volume attached to the instance.  
   - **Variables:**  
     - `item`: Represents each volume ID extracted from the instance information.  
     - `aws_region`: Specifies the AWS region in which the snapshot is created.  
   - **Purpose:** Creates snapshots of the root (or other attached) volumes with a description that includes both the volume and instance IDs.

6. **Verify Snapshot Creation**  
   - **Task:** Uses the `debug` module to output a list of snapshot IDs created.  
   - **Purpose:** Provides confirmation that the snapshot creation process has completed successfully.

7. **Check File System Space**  
   - **Task:** Executes a shell command using the `shell` module to run `df -h` and `awk` to verify that no filesystem (except `/home`) is more than 95% full.  
   - **Purpose:** Ensures that at least 5% free space is available on all filesystems (other than `/home`) before proceeding with updates.  
   - **Behavior:**  
     - The task fails if any filesystem exceeds 95% usage.

8. **Report Insufficient Space if Necessary**  
   - **Task:** Uses the `fail` module to halt the playbook with an error message if insufficient space is detected.  
   - **Purpose:** Prevents the update process from continuing if critical disk space requirements are not met.

9. **Update Host Using DNF**  
   - **Task:** Uses the `dnf` module to update all installed packages asynchronously (using `async` and `poll` to manage background execution).  
   - **Purpose:** Ensures that all system packages are updated to their latest versions.

10. **Check Sync Status**  
    - **Task:** Uses the `async_status` module to poll the status of the asynchronous DNF update until the job completes.  
    - **Purpose:** Confirms that the package update process finishes successfully.

11. **Debug Status of the Update**  
    - **Task:** Uses the `debug` module to output whether the asynchronous update job has finished.  
    - **Purpose:** Provides feedback regarding the update status.

12. **Reboot Server**  
    - **Task:** Uses the `reboot` module to reboot the server with a timeout of 300 seconds.  
    - **Purpose:** Applies changes that require a system restart after updating packages.

13. **Verify sshd Service is Running**  
    - **Task:** Gathers service facts using the `service_facts` module.  
    - **Purpose:** Ensures that critical services (like `sshd`) are running post-reboot.

14. **Check Each Service Status**  
    - **Task:** Iterates over a list (`system_services_list`) and uses the `assert` module to verify that each listed service is running.  
    - **Purpose:** Validates that all required system services are operational, and fails with an error message if any service is not running.

15. **Report Update Success**  
    - **Task:** Uses the `debug` module to print a success message confirming the update and the running status of `sshd`.  
    - **Purpose:** Indicates that the update process has been successfully completed.

16. **Check The CV Version**  
    - **Task:** Uses the `theforeman.foreman.content_view_info` module to retrieve details about a Satellite content view, including its latest version and components.  
    - **Variables:**  
      - `satellite_deployment_admin_username`, `satellite_deployment_admin_password`, `satellite_deployment_server`, `satellite_content_view`, `satellite_deployment_organization`  
    - **Purpose:** Retrieves current content view information to be used in tagging the instance with update details.

17. **Report the CV Version**  
    - **Task:** Uses the `debug` module to output the details of the content view version retrieved from Satellite.  
    - **Purpose:** Provides visibility into the current content view configuration.

18. **Initialize Composite CV Fact**  
    - **Task:** Sets a fact named `cv_comp` that holds the content view’s name, its latest version, and an initially empty list for components.  
    - **Purpose:** Prepares a composite data structure for storing detailed update information.

19. **Populate Component CV Details**  
    - **Task:** Iterates over each component in the content view and updates the `cv_comp` fact with detailed component information.  
    - **Variables:**  
      - `item`: Each component in the content view.  
      - `component_id` and `component_version`: Identifiers used to look up component details.  
      - `content_view_component` and `component_cvv`: Used to extract the published date information.  
      - `component_entry`: A dictionary containing the component's name, version, and published date, which is added to `cv_comp.components`.  
    - **Purpose:** Aggregates detailed version and published date information for each component of the content view.

20. **Show Composite CV Structure**  
    - **Task:** Uses the `debug` module to display the full composite content view (`cv_comp`) structure.  
    - **Purpose:** Provides an overview of the aggregated update information.

21. **Create an Update Tag Summary String**  
    - **Task:** Sets a new fact named `update_summary` that constructs a string summarising the composite content view version and each component’s version.  
    - **Purpose:** Prepares a concise string representing the update details, ensuring it is within a 256-character limit.

22. **Debug the Update Summary**  
    - **Task:** Uses the `debug` module to output the update summary string and its character count.  
    - **Purpose:** Verifies that the summary string meets the length requirements.

23. **Create Tag Key and Value Facts**  
    - **Task:** Sets two facts:
      - `backup_tag_key`: A unique key incorporating the instance ID and current epoch time.
      - `backup_tag_value`: The update summary string.
    - **Purpose:** Prepares key/value pairs that will be used to tag the instance with update details.

24. **Apply Composite CV Update Tag to Instance**  
    - **Task:** Uses the `amazon.aws.ec2_tag` module to apply the new tag (with the key and value defined above) to the instance.  
    - **Purpose:** Tags the instance in AWS with update details for record-keeping.

25. **Get Current Instance Tags**  
    - **Task:** Uses the `amazon.aws.ec2_instance_info` module to retrieve current tags assigned to the instance.  
    - **Purpose:** Gathers existing tags to identify which ones should be cleaned up.

26. **Extract Tags for Cleanup**  
    - **Task:** Uses the `set_fact` module to filter the instance’s tags and create a dictionary (`cleanup_tags`) of those starting with `Server_update_`.  
    - **Purpose:** Identifies old update tags that should be removed to prevent tag clutter.

27. **Remove Old Server Update Tags**  
    - **Task:** Uses the `amazon.aws.ec2_tag` module to remove any tags found in `cleanup_tags` from the instance.  
    - **Condition:** This task executes only if there are tags to remove.  
    - **Purpose:** Cleans up outdated update tags from the instance.

---

This detailed description outlines the flow of the playbook and explains the purpose of each task and variable. The steps range from initial variable loading and metadata retrieval to system updates, service verification, and the tagging process for update events.

