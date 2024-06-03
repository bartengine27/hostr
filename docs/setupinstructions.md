### Prerequisites

Before running the Ansible playbooks, ensure that you have the following:

* Ansible installed on your local machine or a control node.
* Sufficient privileges to manage the systems designated in your inventory.
* Machines (VMs, containers, or bare metal) defined in your [hosts file](./../playbooks/hosts) and accessible over your network.

### Running the Playbooks

To deploy the roles listed above, follow these steps:

1. **Navigate to your local clone of this repository:**
   Open a terminal and ensure you are in the repository directory.

2. **Setup your inventory:**
   Ensure that your `hosts` file in the `./playbooks` directory is updated to reflect the infrastructure where you want to deploy the roles.

3. **Configure role variables**
   Default variables are provided for all roles in `role_folder/defaults/vars.yml` and initialized by reading environment variables. Example environment variables are [available](./../example.env).

4. **Execute the Ansible Playbook:**
   Run the following command to start the deployment process. This command will prompt you for your sudo password for the hosts defined in the inventory due to the `-K` flag and provide detailed output with the `-vvv` verbosity level.

   ```shell
   ansible-playbook site.yml -K -i hosts -vvv
   ```

### Expected Outcome

After executing the playbook, all specified roles should be successfully configured and running across your specified machines. You can verify the status of each role through the corresponding management interfaces or by checking the services directly on each machine.

### Troubleshooting

If you encounter any issues during the setup, consider the following troubleshooting steps:

* Verify network connectivity between your control node and the target machines.
* Check the permissions and authentication details for the accounts used.
* Review the verbose output provided by the Ansible execution for errors or warnings.
