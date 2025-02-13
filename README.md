**Role server_update**
# System Update & Tagging Documentation

This documentation describes an Ansible workflow to:
1. Capture an instance’s ID from the AWS metadata service.  
2. Create snapshots of attached EBS volumes before updates.  
3. Check disk usage to avoid filling local filesystems.  
4. Update the system (packages) asynchronously and reboot.  
5. Ensure `sshd` and other specified services are running post-reboot.  
6. Retrieve and display the latest content view version from Satellite.  
7. Construct and apply an **update tag** on the AWS EC2 instance for tracking.  
8. Clean up old update tags to maintain a clean tag set.

---

## Table of Contents

1. [Overview](#overview)  
2. [Key Steps](#key-steps)  
   - [Include Role Variables](#include-role-variables)  
   - [Retrieve Instance ID from Metadata](#retrieve-instance-id-from-metadata)  
   - [Create Snapshot of Attached Volumes](#create-snapshot-of-attached-volumes)  
   - [Check File System Space](#check-file-system-space)  
   - [Update System Packages](#update-system-packages)  
   - [Reboot and Verify Services](#reboot-and-verify-services)  
   - [Retrieve & Display Satellite CV Info](#retrieve--display-satellite-cv-info)  
   - [Create and Apply Update Tag](#create-and-apply-update-tag)  
   - [Clean Up Old Update Tags](#clean-up-old-update-tags)  
3. [Usage](#usage)  
4. [Notes](#notes)  
5. [Contributing](#contributing)  
6. [License](#license)

---

## Overview

This playbook ensures a robust system update procedure by:
- **Taking snapshots** before patching.  
- **Verifying free disk space**.  
- **Applying updates & rebooting**.  
- **Confirming critical services** are running.  
- **Tagging** the EC2 instance with the content-view information from a Red Hat Satellite environment.  
- **Cleaning up** older update tags to keep only the latest relevant information.

---

## Key Steps

### Include Role Variables
Loads all required variable definitions (e.g., AWS region, Satellite credentials, or service lists) from a dedicated `vars/all.yml`.

### Retrieve Instance ID from Metadata
Uses the instance metadata service (`169.254.169.254`) to gather the **`instance-id`**. This allows the subsequent AWS tasks to identify and manage the correct EC2 instance without hardcoding.

### Create Snapshot of Attached Volumes
Queries AWS for the list of volumes attached to the instance and creates a **snapshot** of each before updating. This provides a rollback point if something goes wrong.

### Check File System Space
Runs a shell command to ensure no mounted filesystems (excluding `/home`) exceed **95% usage**. If any do, the playbook fails to prevent running out of space during updates.

### Update System Packages
Performs a **DNF update** asynchronously, allowing large update operations to continue in the background. The playbook checks the job status until it completes.

### Reboot and Verify Services
Issues a **reboot** to apply the updates. Once the system is back online, it verifies that `sshd` (and any other services listed in `system_services_list`) is running.

### Retrieve & Display Satellite CV Info
- **Content View (CV) details** are fetched from a Satellite server.  
- **Latest CV version** and the component CV versions are extracted.  
- A summary string is built to record the **composite content view** version and associated component versions.

### Create and Apply Update Tag
Generates a unique **`Server_update_[instance_id]_[epoch]`** tag key, populated with the combined content view information. This **tags** the EC2 instance for easy historical tracking of when and which content view was applied.

### Clean Up Old Update Tags
Extracts **existing update tags** starting with `Server_update_` from the instance and removes them. This prevents tags from proliferating after repeated updates.

---

## Usage

1. **Set Variables**:  
   - **`aws_region`**, **`satellite_deployment_server`**, **`satellite_deployment_admin_username`**, **`system_services_list`**, etc.  
   - Ensure these are defined in `vars/all.yml` or passed via inventory/group vars as needed.

2. **Run the Playbook**:  
   - The tasks will automatically detect the instance ID, create snapshots, update the system, reboot, tag the instance, and remove old tags.

3. **Validate**:  
   - Confirm your volumes have new snapshots.  
   - Verify system logs confirm the reboot.  
   - Check the AWS console for the updated tags.  
   - Inspect your Satellite or Foreman server to ensure the content view information matches.

---

## Notes

- **Async Updates**: If your package updates are very large, the async approach helps avoid idle time in Ansible.  
- **Snapshot Cost & Cleanup**: While snapshots protect against failures, they incur costs. Make sure your environment has a strategy for snapshot retention.  
- **Tagging**: This flow keeps the EC2 instance tag set tidy by removing older tags with the same prefix.


---

# Cluster Update Documentation

This documentation describes an Ansible-based approach for managing and verifying critical cluster services in a High Availability (HA) environment. Tasks include checking the status of cluster services and nodes, ensuring fencing and HANA resources are functional, dynamically discovering slave nodes, and performing updates or maintenance.

---
**Role cluster_update**
## Overview

## Table of Contents

1. [Overview](#overview)
2. [Key Tasks](#key-tasks)
    - [Gather Service Facts](#gather-service-facts)
    - [Verify Cluster Services](#verify-cluster-services)
    - [Report Success](#report-success)
    - [Check Cluster Node Status](#check-cluster-node-status)
    - [Extract Lost Nodes](#extract-lost-nodes)
    - [Fail if Nodes Lost](#fail-if-nodes-lost)
    - [Check Fencing Resource Status](#check-fencing-resource-status)
    - [Fail if Fencing Failed](#fail-if-fencing-failed)
    - [Check HANA Resource Status](#check-hana-resource-status)
    - [Extract Master & Slave Nodes](#extract-master--slave-nodes)
    - [Add Slave Nodes to Ad-Hoc Inventory](#add-slave-nodes-to-ad-hoc-inventory)
    - [Include Dynamic Tasks](#include-dynamic-tasks)
3. [Included Task File: `run_on_dynamic_hosts.yml`](#included-task-file-run_on_dynamic_hostsyml)
    - [Verify Connection](#verify-connection)
    - [Backup Configuration](#backup-configuration)
    - [Standby Nodes](#standby-nodes)
    - [Stop Cluster Services](#stop-cluster-services)
    - [Verify Services Stopped](#verify-services-stopped)
    - [Final Status](#final-status)
4. [Usage](#usage)
5. [Notes](#notes)
6. [Contributing](#contributing)
7. [License](#license)

---

## Overview

This playbook manages updates and validations on a cluster. It:

- Gathers service facts  
- Ensures critical HA services are running  
- Detects and handles node failures  
- Checks fencing and HANA resource statuses  
- Dynamically identifies slave nodes to perform actions (standby, stop, etc.)

---

## Key Tasks

### Gather Service Facts
Collects information about all local services. This allows subsequent tasks to verify whether specific cluster services are running or stopped.

### Verify Cluster Services
Ensures the specified HA services are running. If any required service is not in the running state, the playbook fails immediately, prompting manual intervention.

### Report Success
Displays a debug message indicating that the cluster services are successfully running.

### Check Cluster Node Status
Retrieves a list of cluster nodes and their current states (e.g., `lost`, `member`). This helps confirm that all nodes are healthy before proceeding.

### Extract Lost Nodes
Parses the output for any nodes in a `lost` state. If found, they are recorded for further reporting or failure.

### Fail if Nodes Lost
If any nodes appear as `lost`, the playbook fails, indicating a cluster health issue that requires immediate attention.

### Check Fencing Resource Status
Verifies that the fencing resource is started and functioning. Fencing ensures stable cluster operations by isolating faulty nodes.

### Fail if Fencing Failed
If the fencing resource is not started, the playbook fails with a descriptive error. This prevents proceeding with cluster operations on an unstable environment.

### Check HANA Resource Status
Confirms that HANA resources are running as expected. This includes details on master/slave roles or any operational state.

### Extract Master & Slave Nodes
Parses the HANA resource status to identify the master and all slave nodes. This is crucial when performing rolling updates or maintenance on slave nodes first.

### Add Slave Nodes to Ad-Hoc Inventory
Dynamically adds identified slave nodes into an in-memory inventory group. This lets the playbook run additional tasks only on those nodes.

### Include Dynamic Tasks
Brings in additional tasks for the slave nodes, such as creating backups, putting them into standby, or stopping services before updates.

---

## Included Task File: `run_on_dynamic_hosts.yml`

### Verify Connection
Tests connectivity to the newly added slave nodes. Ensures that Ansible can communicate and run tasks on them.

### Backup Configuration
Generates a timestamped backup of the cluster configuration (using `pcs config backup`). This safeguards configurations before proceeding with disruptive operations.

### Standby Nodes
Puts the slave nodes into standby mode, preventing them from serving active roles during maintenance.

### Stop Cluster Services
Stops the cluster services on the slave nodes to prepare them for updates. This task includes a pause to allow the services to fully terminate.

### Verify Services Stopped
Checks that the relevant HA services on slave nodes are no longer running, ensuring a clean state for subsequent updates.

### Final Status
Reports that the slave nodes are now ready for upgrade or further maintenance steps.

---

## Usage

1. **Review Variables**: Ensure the relevant variables (like cluster service names, resource names, etc.) are correctly set in the inventory or variable files.
2. **Run the Playbook**: Execute the main Ansible playbook. It will automatically include the tasks in `run_on_dynamic_hosts.yml` for newly discovered slave nodes.
3. **Validate**: The playbook will fail early if any critical checks (e.g., lost nodes, fencing failures) are encountered, ensuring safe and reliable cluster operations.

---
**Role server_update**

# Satellite Registration Documentation

This documentation describes an Ansible workflow for registering a host to a Red Hat Satellite (or Foreman) server. The workflow checks whether the host is already registered, cleans up any pre-existing repositories if necessary, installs the Katello CA, and then registers the system using an activation key.

---

## Table of Contents

1. [Overview](#overview)
2. [Key Steps](#key-steps)
    - [Include Role Variables](#include-role-variables)
    - [Check Existing Host Registration](#check-existing-host-registration)
    - [Cleanup Unneeded Files](#cleanup-unneeded-files)
    - [Configure Subscription Manager](#configure-subscription-manager)
    - [Install Katello CA](#install-katello-ca)
    - [Register System to Satellite](#register-system-to-satellite)
    - [Verify Registration](#verify-registration)
3. [Usage](#usage)
4. [Important Variables](#important-variables)
5. [Notes](#notes)
6. [Contributing](#contributing)
7. [License](#license)

---

## Overview

This set of tasks performs the following actions:
1. Verifies if the host is already registered to the Satellite server.
2. Removes unwanted local YUM repository files if the host is not registered.
3. Configures the `subscription-manager` to handle repositories.
4. Installs the Katello CA package to secure communication with the Satellite.
5. Registers the system using an activation key.
6. Validates successful registration through the Satellite API.

---

## Key Steps

### Include Role Variables
Loads all necessary variables, including credentials and server details, from a shared file (`vars/all.yml`).

### Check Existing Host Registration
Queries the Satellite server to determine whether the current host is already registered.  
- **Outcome**:  
  - If the host **is** registered, tasks that handle local repository cleanup will be skipped.  
  - If the host **is not** registered, subsequent tasks perform cleanup and registration steps.

### Cleanup Unneeded Files
When the host is not registered, finds any files in a specified local YUM repository directory and removes them. This ensures that no conflicting repository configurations remain before the new registration.

### Configure Subscription Manager
Enables the `subscription-manager` to handle yum repositories (`rhsm.manage_repos=1`). This step allows the system to properly receive content and subscriptions from Satellite.

### Install Katello CA
Installs the Katello CA package from the Satellite (or Capsule) server, with SSL verification disabled. This is necessary so the client trusts the server’s certificate when registering.

### Register System to Satellite
Uses the activation key and organization ID to register the system via `subscription-manager`. The registration is **forced** to ensure re-registration if needed.

### Verify Registration
Queries the Satellite again to confirm that the host is now successfully registered. If no host record is found, the play fails. Otherwise, it reports a successful registration.

---

## Usage

1. **Set or Review Variables**:  
   - `satellite_deployment_server`, `satellite_deployment_admin_username`, and other variables must be properly defined in `vars/all.yml`.
2. **Run the Playbook**:  
   Execute the relevant Ansible playbook that includes these tasks. The playbook automatically checks the existing registration, cleans up repositories if needed, and installs the Katello CA before registering the system.
3. **Validate Registration**:  
   - If the play completes successfully, the host is registered.  
   - If any step fails (for example, `Host is null; the registration did not succeed.`), troubleshoot the Satellite configuration and credentials.

---

## Important Variables

- **`satellite_deployment_server`**: URL of the Satellite or Foreman server.  
- **`satellite_deployment_admin_username`**, **`satellite_deployment_admin_password`**: Credentials to query or update host info.  
- **`satellite_deployment_organization`**: Organization ID/name for registration.  
- **`sat_activation_key`**: Activation key used to register the host.  
- **`yum_repo_dir`**: Directory containing local repository files to be cleaned up if not registered.  
- **`satellite_kat_cert`**: The Katello CA certificate package name, typically in `/pub` directory on the Satellite.

---

## Notes

- **SSL Verification** is disabled for CA installation to handle scenarios where local certificates are not yet trusted. If security policies require it, replace it with a secure approach.
- **Force Registration** ensures that if the system is partially registered or misconfigured, it will be re-registered correctly.

---

# Server Restore Documentation

This documentation outlines the process for **restoring an Amazon EC2 instance** to its latest snapshot. The workflow dynamically discovers the instance’s AWS region and root volume, identifies the most recent snapshot, and replaces the current root volume with one created from that snapshot.

---

## Table of Contents

1. [Overview](#overview)  
2. [Key Steps](#key-steps)  
   - [Include Role Variables](#include-role-variables)  
   - [Get Instance Details](#get-instance-details)  
   - [Extract Dynamic Facts](#extract-dynamic-facts)  
   - [Find the Latest Snapshot](#find-the-latest-snapshot)  
   - [Stop and Replace the Volume](#stop-and-replace-the-volume)  
   - [Start Instance](#start-instance)  
   - [Final Status](#final-status)  
3. [Usage](#usage)  
4. [Notes](#notes)  
5. [Contributing](#contributing)  
6. [License](#license)

---

## Overview

This set of tasks automates restoring an EC2 instance by:

1. Dynamically retrieving instance metadata (including its region and device names).  
2. Locating and selecting the **latest** completed snapshot for the root volume.  
3. Stopping the instance, creating a new volume from the snapshot, and attaching that volume as the new root.  
4. Restarting the instance to verify successful recovery.

---

## Key Steps

### Include Role Variables
Loads necessary variables (credentials, default regions, etc.) from a `vars` file. These variables provide details like the default AWS region if it’s not inferred from the instance.

### Get Instance Details
Retrieves metadata about the target instance, such as its availability zone, current state, and attached block devices.

### Extract Dynamic Facts
1. **Availability Zone**: Used to determine where new volumes should be created.  
2. **AWS Region**: Derived by removing the last letter from the AZ. For example, if the AZ is `us-east-1a`, the region is `us-east-1`.  
3. **Root Device Name**: Identifies which block device is the root volume.  
4. **Old Root Volume ID**: Extracted by matching the device name in the instance’s block device mappings.

### Find the Latest Snapshot
Queries AWS for snapshots of the old root volume.  
- If none exist, the playbook fails immediately.  
- If they do exist, it sorts them by start time and selects the most recent snapshot.

### Stop and Replace the Volume
1. **Stop the Instance**: Ensures it’s safe to detach the volume.  
2. **Wait for Stop**: Polls until the instance is confirmed stopped.  
3. **Create a New Volume** from the snapshot in the same AZ.  
4. **Wait for the Volume to Become Available**: Ensures the volume is fully provisioned before attaching.  
5. **Detach the Old Root Volume**: Removes the existing volume from the instance.  
6. **Attach the New Volume**: Mounts the newly created volume on the same device name (`/dev/sda1` or `/dev/xvda`, depending on the instance).

### Start Instance
Begins running the instance again, now using the restored volume. A wait ensures the instance is fully running before proceeding.

### Final Status
Prints a debug message summarising the successful restore operation, including the new volume ID and snapshot used.

---

## Usage

1. **Define Your Variables**: Make sure `instance_id`, `aws_region` (if necessary), or any required credentials are set.  
2. **Run the Tasks**: Incorporate this set of tasks into your Ansible playbook or role. The role will stop, restore, and restart the instance automatically.  
3. **Validate**: Confirm in the AWS console or via the Ansible output that the instance is running from the new volume and that the state is `running`.

---

## Notes

- This process **overwrites** the current root volume. Ensure you understand the implications of using the **latest** snapshot.
- If your instance has multiple volumes (data volumes, etc.), this workflow focuses only on the **root** volume restore.
- The tasks dynamically determine the region if it wasn’t provided, allowing you to reuse the same code in different environments.

---
**Role snapshot_cleanup

# Snapshot Cleanup Documentation

This documentation describes an Ansible-based workflow that retrieves an instance’s region and volume metadata from the AWS instance metadata service, gathers associated EBS volume IDs, and then cleans up old snapshots while preserving the most recent ones. It also demonstrates how to handle a specific “newest old snapshot” for retention alongside the latest snapshot overall.

---

## Table of Contents

1. [Overview](#overview)  
2. [Key Steps](#key-steps)  
    - [Import Variables](#import-variables)  
    - [Fetch Instance Identity Document](#fetch-instance-identity-document)  
    - [Gather Volume Information](#gather-volume-information)  
    - [Include Snapshot Cleanup Tasks](#include-snapshot-cleanup-tasks)  
    - [Determine Snapshots to Delete](#determine-snapshots-to-delete)  
    - [Delete Old Snapshots](#delete-old-snapshots)  
3. [Usage](#usage)  
4. [Notes on Snapshot Retention](#notes-on-snapshot-retention)  
5. [Contributing](#contributing)  
6. [License](#license)

---

## Overview

This collection of tasks automates the process of:

- **Detecting** your AWS region and EC2 instance ID from the instance metadata service.  
- **Retrieving** information on all volumes attached to the instance.  
- **Locating** existing snapshots for each volume and determining which are the oldest.  
- **Preserving** the latest snapshot and one “newest old snapshot” (the most recent among the older snapshots).  
- **Deleting** all other snapshots to free up resources and control snapshot sprawl.

---

## Key Steps

### Import Variables
Loads role-wide or playbook-wide variables from a file (e.g., credentials, default region settings, or other custom settings).

### Fetch Instance Identity Document
Queries the instance metadata service at `169.254.169.254` to obtain essential information, including the AWS region and the instance ID. This step is necessary for self-discovery, so the tasks can run on any EC2 instance without requiring hardcoded region or instance details.

### Gather Volume Information
Retrieves metadata about the volumes attached to the instance:
1. Identifies all block device mappings for the instance.  
2. Extracts the corresponding **volume IDs**.  
3. Displays a debug message listing the discovered volumes.

### Include Snapshot Cleanup Tasks
For each discovered volume, a separate set of tasks (in `snapshot_cleanup.yml`) runs, retrieving and filtering snapshots.

### Determine Snapshots to Delete
1. **Collect all snapshots** associated with the current volume.  
2. Identify snapshots **older than one week**.  
3. Determine the **latest snapshot overall** (completely up to date).  
4. Identify the **newest among the old snapshots** (kept for additional fallback).  
5. **Build a removal list** by excluding these two snapshots (the very latest and the newest old one) from the full set.

### Delete Old Snapshots
Removes all snapshots in the `remove_list`. This helps clean up stale snapshots and control costs without losing essential recovery points.

---

## Usage

1. **Prepare Your Environment**:  
   - Ensure you have proper AWS credentials or an IAM role assigned to the instance for snapshot deletion.  
   - Make sure `vars/all.yml` (or your chosen variable file) includes any needed configurations.

2. **Run the Playbook**:  
   - From your Ansible control node (or within the instance itself if using a local play), run the main playbook that includes these tasks.  
   - The tasks will dynamically discover region, instance ID, and attached volumes.

3. **Confirm Snapshot Deletion**:  
   - Check the resulting logs or debug messages.  
   - Optionally, verify in the AWS console that the old snapshots have indeed been removed while the latest snapshot and the newest old snapshot remain intact.

---

## Notes on Snapshot Retention

- This workflow **retains** two snapshots per volume:  
  1. The **latest snapshot** for up-to-date recovery.  
  2. The **newest of the old snapshots**, offering one additional fallback in case you need a slightly older restore point.  
- You can adjust the **time window** (one week) by modifying the logic that calculates `one_week_ago`.  
- Deletion logic uses `ec2_snapshot` with `state: absent`, so once deleted, snapshots **cannot** be recovered unless you have additional copies.

---
**Role server_update**

# Satellite Content Upload Documentation

This documentation explains how to download a package from a specified URL, upload it to a Satellite file-based repository, and then clean up the temporary file used during the process.

---

## Table of Contents

1. [Overview](#overview)  
2. [Key Steps](#key-steps)  
   - [Include Role Variables](#include-role-variables)  
   - [Download Package](#download-package)  
   - [Upload to Satellite Repository](#upload-to-satellite-repository)  
   - [Clean Up Temporary File](#clean-up-temporary-file)  
3. [Usage](#usage)  
4. [Notes](#notes)  
5. [Contributing](#contributing)  
6. [License](#license)

---

## Overview

This workflow automates:
1. Loading relevant role variables.  
2. Downloading a package (e.g., an AWS CLI ZIP file) to a temporary location.  
3. Uploading that file to a specific **Satellite product repository**.  
4. Cleaning up the local temporary file afterward.

---

## Key Steps

### Include Role Variables
Fetches variables that might include **`package_url`**, **`package_name`**, **`satellite_repository`**, **`satellite_product`**, and **`satellite_deployment_admin_username`** or **`satellite_deployment_admin_password`**.

### Download Package
Retrieves the package from the specified URL and places it in a defined temporary location. The file’s permissions are set to ensure it is readable by Ansible for subsequent tasks.

### Upload to Satellite Repository
Uploads the downloaded file to the **Satellite** or **Foreman** server’s **file-based repository**. This step requires:
- Valid **credentials** (username/password).  
- **Server URL** pointing to the Satellite/Foreman.  
- Proper **product** and **repository** names where the package should be stored.

### Clean Up Temporary File
Removes the downloaded package to keep the system tidy and prevent accumulation of unnecessary files.

---

## Usage

1. **Confirm Variables**: Ensure that variables such as:
   - **`package_url`** (source URL to download from),  
   - **`satellite_deployment_server`** (FQDN or IP of the Satellite server),  
   - **`satellite_deployment_admin_username`** and **`satellite_deployment_admin_password`**,  
   - **`satellite_repository`** and **`satellite_product`**,  
   - **`package_tmp`** (temporary storage directory),  
   - **`package_name`** (name of the downloaded package),
   are correctly set.

2. **Execute the Play/Role**:  
   - This will download the package to the `{{ package_tmp }}` directory.  
   - Upload the package to the **Satellite** repository.  
   - Remove the local copy of the package.

3. **Verify**:  
   - After completion, log in to **Satellite** or **Foreman** to confirm the file is present in the intended repository.

---

## Notes

- **SSL Validation**: For secure environments with valid certificates, set `validate_certs: true`.  
- **Custom Package Names**: Adjust `package_name` and `package_url` to point to different software if needed.  
- **Permissions**: Confirm that the **temporary directory** is accessible by Ansible and not restricted by SELinux or other security contexts.

---

