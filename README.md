# Managing OCI resources using GitHub actions
## Overview
Needs is to manage an Oracle Cloud Infrastructure (OCI) resources using GitHub Actions through the workflow in your GitHub repository. Define a YAML file for the workflow, specifying the trigger event and required steps. OCI CLI or SDK help in the workflow to authenticate and interact with OCI services, issuing commands to manage the specified resources.
As an illustration of how to handle resource management, utilize a GitHub workflow for instance restart within your repository to showcase the methodology behind achieving such functionality
## Flow diagram
The flow diagram delineates the collaboration between GitHub Actions and Oracle Cloud. The diagram outlines the use of secrets within GitHub for securely storing API keys. Additionally, it showcases the integration of the 'oracle-actions/run-oci-cli-command@v1.1.1' module, facilitating connection to Oracle Cloud Infrastructure (OCI) through the Command Line Interface (CLI) tool. The workflow also incorporates a script to perform actions on a specific resource, exemplified in this case by an instance.
Furthermore, an Identity and Access Management (IAM) policy has been implemented to ensure that the relevant user possesses the necessary permissions to execute actions with the specified service.
## OCI Create User and Policy
Creating a user and policy in Oracle Cloud Infrastructure (OCI) involves several steps, including creating the user, assigning the necessary permissions using policies, and generating required credentials. Below are the general steps to achieve this using the OCI Console:
### Step 1: Create a User
1. Login to OCI Console: Log in to your Oracle Cloud Infrastructure Console.
2. Access Identity & Security: Navigate to the "Identity & Security" section in the Console.
3. Users: Select "Users" from the left-hand menu.
4. Create User: Click on the "Create User" button.
5. Enter User Details: Fill in the required user details, including the username, description, and tags.
6. Authentication: Choose how the user will authenticate. For example, you can create a password for the user or use the user's public key.
7. Add to Groups: Optionally, add the user to one or more groups to inherit policies from those groups.
8. Create: Click "Create" to create the user.

### Step 2: Create a Policy
1. Access Identity & Security: Navigate to the "Identity & Security" section in the Console.
2. Policies: Select "Policies" from the left-hand menu.
3. Create Policy: Click on the "Create Policy" button.
4. Enter Policy Details: Fill in the required policy details, including the policy name, description, and policy statements.
5. Define Statements: Define the statements that grant permissions to the user. For example:
Adjust the statements based on the specific permissions you want to grant.

```Allow group <group-name> to manage instance-family in tenancy```

6. Create: Click "Create" to create the policy.
### Step 3: Generate API Key
1. Access Identity & Security: Navigate to the "Identity & Security" section in the Console.
2. Users: Select "Users" from the left-hand menu.
3. Select User: Click on the user you created.
4. API Keys: Under the "API Keys" tab, click on "Add Public Key" to create the public key for the user.
5. Download key for using them in ~/.oci/config
6. Click Add
### Important Note:
   It's crucial to follow the principle of least privilege, granting only the necessary permissions to users and policies.
   Adjust the policy statements according to your specific use case and requirements.
   Safeguard the user's private key and credentials.

## OCI config generation
To generate an OCI (Oracle Cloud Infrastructure) configuration file (~/.oci/config), you typically follow these steps:
1. Install OCI CLI: Make sure you have the OCI CLI (Command Line Interface) installed on your machine. You can download it from the official Oracle website.
2. Configure OCI CLI: Run the following command in your terminal:
  ```oci setup config```

   This command will prompt you to enter the required information interactively. You'll be asked for your user OCID, tenancy OCID, region, and whether you want to create a new API key pair or use an existing one.
3. Alternatively, Manually Create the Configuration File:
   If you prefer creating the configuration file manually, you can follow the format you provided in your example. Create a file named config in the ~/.oci/ directory and add the following content:
   ```
   cat ~.oci/config
   [DEFAULT]
   user=ocid1.user.oc1..aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
   fingerprint=00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
   tenancy=ocid1.tenancy.oc1..aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
   region=eu-frankfurt-1
   key_file=~/.oci/instance_restart.pem
   ```
   Replace the placeholders (ocid1.user.oc1.., 00:00:00:00:00:00:00:00, etc.) with your actual values. Make sure the specified key_file points to the location of your private key file generated before.
   Remember, your private key file (~/.oci/instance_restart.pem) should be kept secure, and you should not share it or expose it to unauthorized users. It's also crucial to follow OCI security best practices and manage your API keys securely.
## GitHub actions
```
   name: 'Restart Instance every 6 day'
##### UNCOMENT HERE
#on:
#schedule:
#  - cron: "0 1 1-30/6 * *"
# push:
#   branches: [ "master" ]
# pull_request:
permissions:
contents: read
jobs:
my-instances:
runs-on: ubuntu-latest
name: Restart Instance every 6 day
env:
OCI_CLI_USER: ${{ secrets.OCI_CLI_USER }}
OCI_CLI_TENANCY: ${{ secrets.OCI_CLI_TENANCY }}
OCI_CLI_FINGERPRINT: ${{ secrets.OCI_CLI_FINGERPRINT }}
OCI_CLI_KEY_CONTENT: ${{ secrets.OCI_CLI_KEY_CONTENT }}
OCI_CLI_REGION: ${{ secrets.OCI_CLI_REGION }}
OCI_INSTANCE_ID: ${{ secrets.OCI_INSTANCE_ID }}
steps:
- name: oci restart ${{ OCI_INSTANCE_ID }}
uses: oracle-actions/run-oci-cli-command@v1.1.1
id: restart_instance
with:
command: 'compute instance action --instance-id ${{ OCI_INSTANCE_ID }} --action SOFTRESET'
- name: Show output
run: |
echo ${{ steps.restart_instance.outputs.output }} | jq .
```
The provided YAML code represents a GitHub Actions workflow named 'Restart Instance every 6 day.' The workflow is scheduled to run every 6 days using a cron expression (currently commented out) and is triggered on pushes to the 'master' branch or pull requests. The specified permissions allow reading of repository contents.
The 'my-instances' job runs on an Ubuntu latest environment and is named 'Restart Instance every 6 day.' It defines environment variables for Oracle Cloud Infrastructure (OCI) CLI authentication, including user, tenancy, fingerprint, key content, region, and instance ID.
Within the job steps, the OCI instance identified by the specified instance ID undergoes a soft reset action using the 'oracle-actions/run-oci-cli-command@v1.1.1' module. The output of this action is then displayed using the 'jq' command.
## Note: 
The scheduling part is currently commented out, and it's recommended to uncomment and adjust the schedule as needed for your specific requirements. Additionally, ensure that the secrets mentioned in the workflow are properly set up in your GitHub repository.
## Summary
This GitHub Actions workflow demonstrates how to manage Oracle Cloud Infrastructure (OCI) resources, specifically restarting instances. The workflow is triggered on a schedule, runs on an Ubuntu environment, and utilizes OCI CLI authentication through environment variables. A flow diagram and IAM policy ensure secure collaboration and appropriate permissions for resource management. Additional links provide access to relevant GitHub Actions for OCI and cron scheduling.
## Links:
https://dmytrovedetskyi.medium.com/managing-oci-resources-using-github-actions-d893b23b25d4
https://github.com/marketplace/actions/run-an-oracle-cloud-infrastructure-oci-cli-command
https://github.com/marketplace/actions/set-cron-schedule
https://github.com/helli0n/oci-instance-restart
