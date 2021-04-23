# Installing `kubectl` on IBM Cloud Virtual Servers for VPC with IBM Cloud Schematics

[Virtual Servers for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-about-advanced-virtual-servers) are an Infrastructure-as-a-Service (IaaS) offering that gives you access to all of the benefits of IBM Cloud VPC, including network isolation, security, and flexibility.

This playbook is designed to install the Kubernetes CLI `kubectl` on an [IBM Cloud Virtual Servers for VPC instance](https://cloud.ibm.com/docs/vpc?topic=vpc-about-advanced-virtual-servers) by using the built-in Ansible capabilities in Schematics. You can use the Kubernetes CLI to work with and manage Kubernetes clusters.

​IBM Cloud Schematics provides powerful tools to automate your cloud infrastructure provisioning and management process, the configuration and operation of your cloud resources, and the deployment of your app workloads. To do so, Schematics uses open source projects, such as Terraform, Ansible, OpenShift, Operators, and Helm, and delivers these capabilities to you as a managed service. Rather than installing each open source project on your workstation, and learning the API or CLI. You declare the tasks that you want to run in IBM Cloud and watch Schematics run these tasks for you. For more information about Schematics, see [About IBM Cloud Schematics](https://cloud.ibm.com/docs/schematics?topic=schematics-about-schematics).
​
## About this playbook
​
To run this playbook, you must have a Virtual Private Cloud (VPC) and a Virtual Servers for VPC instance (VSI) where you want to install the Kubernetes CLI (`kubectl`). When you run this playbook, Schematics securely connects to the target VSI by using the SSH key that you configured when you created the VSI. To ensure that access to your target VSI is secured always, this playbook requires a bastion host to be configured within your VPC in addition to the target VSI. You can automate the setup of your VPC, your target VSI, and the bastion host by using a [Schematics Terraform template](https://github.com/Cloud-Schematics/multitier-bastion-vpc-lamp).

The playbook in this repository installs `kubectl` on virtual machine from Ansible Galaxy role **andrewrothstein.kubectl**. Also tested on IBM Cloud VPC Generation 2 VSIs that run CentOS 7.x. You can use this playbook on virtual servers that run either CentOS or RHEL. The playbook might not work with other Linux distributions.

## Prerequisites
​
To run this playbook, complete the following tasks:
* Make sure that you have the required permissions to [create an IBM Cloud Schematics action](https://cloud.ibm.com/docs/schematics?topic=schematics-access).
* Make sure that you have the required permissions to [create and work with IBM Cloud VPC infrastructure components](https://cloud.ibm.com/docs/vpc?topic=vpc-iam-getting-started).
* [Create an upload an SSH key to the VPC Generation 2 dashboard](https://cloud.ibm.com/docs/vpc?topic=vpc-ssh-keys). This SSH key is used to access your bastion host and the VSI in your VPC. Make sure that you upload the SSH key to the same region where you want to create your VSIs.
* [Provision a multitier VPC with a bastion host](https://github.com/Cloud-Schematics/multitier-bastion-vpc-lamp) and a VSI that you can use to install the Kubernetes CLI. To create this environment, you use the built-in Terraform capabilities in Schematics workspaces. For more information about Schematics workspaces, see [Creating workspaces](https://cloud.ibm.com/docs/schematics?topic=schematics-workspace-setup#create-workspace).
​
## Input variables
​
You must retrieve the following values to run the playbook in IBM Cloud Schematics.
​
|Input variable|Required/ optional|Data type|Description|
|--|--|--|--|
|`Bastion_IP`|Required|String|Bastion floating IP address.|
|`VSI_IP`|Required|String|IBM Cloud VSI IP address.|
|`SSH_KEY`|Required|String|IBM Cloud SSH Private key used for VSI installation.|
​
## Running the playbook in Schematics by using the console

1. Open the [Schematics action configuration page](https://cloud.ibm.com/schematics/actions/create?name=ansible-kubectl&url=https://github.com/Cloud-Schematics/ansible-kubectl).
2. Review the name for your action, and the resource group and region where you want to create the action. Then, click **Create**.
3. Select the playbook that you want to use.
4. Select the **Verbosity** level to control the depth of information are shown when you run the playbook in Schematics.
5. Click **Save**.
6. In the **IBM Cloud resource inventory** section, click the **edit** button.
7. Enter the following details:
   - **Bastion host IP** Enter the public IP address of the Bastion host that you created.
   - Select an existing inventory group, or click **Create inventory** to create a new one. You can choose to import hosts from an existing workspace or to manually enter your hosts by using the following format. For more information about how to define your host inventory, see the [Ansible documentation](https://docs.ansible.com/ansible/2.9_ja/plugins/inventory/ini.html).

     ```
     [webserver]
     172.16.0.4
     ```
     **Note**
     You can either create dynamic or Static inventory host group. Dynamic and static inventory creations are shown in the screen capture.

     **Sample Dynamic host group creation**

     ![Dynamic host group creation](/images/dyn_invgrp.png)

     **Sample static host group with resource query creation**

     ![Static host group with resource query creation](/images/static_inv.png)

   - **IBM Cloud resource inventory SSH key** Enter the private SSH key that you want to use to connect to your virtual servers. The private SSH key must match the public key that you added to the virtual server when you created it. If you stored the private key on your local workstation, you can run `cat ~/.ssh/id_rsa` to see the private key. **Note** You can run  `pbcopy < ~/.ssh/id_rsa` to copy entire private SSH key and paste.

   - **IBM Cloud resource inventory SSH key** Enter the private SSH key that you want to use to connect to your virtual servers. The private SSH key must match the public key that you added to the virtual server when you created it. If you stored the private key on your local workstation, you can run `pbcopy < ~/.ssh/id_rsa` to copy the private key.

8. Click **Save** to store the values that you entered.
9. Click **Check action** to verify your action details. The **Jobs** page opens automatically. You can view the results of this check by looking at the logs.
8. Click **Run action** to perform the operation on your virtual server. You can monitor the progress of this action by reviewing the logs on the **Jobs** page.
​
## Running the playbook in Schematics by using the command line

1. Create a `hosts.ini` file on your local workstation and add the private IP addresses of the virtual servers where you want to install the Kubernetes CLI.
   ```
   [webserver]
   172.16.0.4
   ```

2. Retrieve the public IP address of the bastion host that you created.
3. Create the Schematics action. Replace `<bastion_host_IP>` with the public IP address of your IP address. When you run this command and are prompted to enter a GitHub token, enter the return key to skip this prompt.
   ```
   ibmcloud schematics action create --name kubectl --location us-south --resource-group default --template https://github.com/Cloud-Schematics/ansible-kubectl --playbook-name kubectl.yml --bastion <bastion_host_IP> --target-file hosts.ini --credential ~/.ssh/id_rsa
   ```

4. Verify that your Schematics action is created and note the ID that was assigned to your action.
   ```
   ibmcloud schematics action list
   ```

5. Create a job to run a check for your action. Replace `<action_ID>` with the action ID that you retrieved. In your CLI output, note the **ID** that was assigned to your job.
   ```
   ibmcloud schematics job run --command-object action --command-object-id <action_ID> --command-name ansible_playbook_check
   ```

   Example output
   ```
   ID                  us-south.JOB.kubectl.fedd2fab
   Command Object      action
   Command Object ID   us-south.ACTION.kubectl.1aa11a1a
   Command Name        ansible_playbook_check
   Name                JOB.kubectl.ansible_playbook_check.2
   Resource Group      a1a12aaad12b123bbd1d12ab1a123ca1
   ```

6. Verify that your job ran successfully by retrieving the logs.
   ```
   ibmcloud schematics job logs --id <job_ID>
   ```

7. Create another job to run the action. Replace `<action_ID>` with your action ID.
   ```
   ibmcloud schematics job run --command-object action --command-object-id <action_ID> --command-name ansible_playbook_run
   ```

8. Verify that your job ran successfully by retrieving the logs.
   ```
   ibmcloud schematics job logs --id <job_ID>
   ```

## Verification

You can view the Kubernetes CLI `kubectl` on your IBM Cloud Virtual Servers for VPC instance details that are created in your Cluster ID. Follow these steps to view the `kubectl` commands.

1. Login to IBM Cloud account on command, where the IBM Cloud Virtual Servers for VPC instance is created.
2. Run `ssh -i ~/.ssh/id_rsa -o ProxyCommand=“ssh -i ~/.ssh/id_rsa -W %h:%p root@169.47.95.98” root@10.240.64.6` to connect to your IBM Cloud Virtual Servers for VPC instance as a root user to view the output as shown in the screen capture.

   **Syntax**

   ```
   ssh -i <ssh_privatekey_path> -o ProxyCommand="ssh -i <ssh_privatekey_path> -W %h:%p root@<bastion_ip>" root@<vsi_ip>
   ```

   **Sample list of `kubectl` commands from a VSI root user**

   ![Kubernetes commands](/images/kubectl_output.png)

## Deleting the action
​
1. From the [Schematics actions dashboard](https://cloud.ibm.com/schematics/actions), find the action that you want to delete.
2. From the actions menu, click **Delete**.
​
## Reference
​
Review the following links to find more information about Schematics and IBM Cloud VPC Virtual Servers

- [IBM Cloud Schematics documentation](https://cloud.ibm.com/docs/schematics)
- [IBM Cloud VPC documentation](https://cloud.ibm.com/docs/vpc?topic=vpc-getting-started)
- [Kubernetes CLI documentation](https://kubernetes.io/docs/reference/kubectl/)
​
## Getting help
​
For help and support with using this template in IBM Cloud Schematics, see [Getting help and support](https://cloud.ibm.com/docs/schematics?topic=schematics-schematics-help).
