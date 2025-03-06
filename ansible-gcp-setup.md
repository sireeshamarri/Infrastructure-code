# For Ubuntu/Debian
sudo apt update
sudo apt install ansible -y

# Using pip
pip install ansible
Configure Ansible for GCP
Install the required Ansible collections and modules for GCP.
sh
Copy Code


ansible-galaxy collection install google.cloud
pip install requests google-auth
3. Set Up Service Account and API Access
Create a service account in GCP with the necessary permissions (e.g., Compute Admin, Storage Admin).
Download the JSON key file for the service account.
Ensure the required APIs (Compute Engine API, Cloud Storage API, etc.) are enabled in your GCP project.
4. Configure GCP Authentication
Set up environment variables to use the service account credentials.
sh
Copy Code


export GCP_AUTH_KIND=serviceaccount
export GCP_SERVICE_ACCOUNT_FILE=/path/to/your-service-account-file.json
export GCP_PROJECT=your-gcp-project-id
5. Create an Ansible Inventory File
Define your inventory file to specify the hosts and groups. This can be a static inventory file or a dynamic inventory script.
yaml
Copy Code


# gcp_inventory.yaml
all:
  vars:
    ansible_python_interpreter: /usr/bin/python3
    gcp_project: your-gcp-project-id
    gcp_credentials_file: /path/to/your-service-account-file.json
  hosts:
    my-gcp-instance:
      ansible_host: 1.2.3.4
      ansible_user: your-ssh-user
6. Create Ansible Playbook
Write a playbook to define the tasks for provisioning and configuring GCP resources.
yaml
Copy Code


# create_instance.yaml
- name: Provision GCP Instance
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Create a GCP compute instance
      google.cloud.gcp_compute_instance:
        name: test-instance
        machine_type: n1-standard-1
        disks:
          - auto_delete: true
            boot: true
            initialize_params:
              source_image: projects/debian-cloud/global/images/family/debian-9
        network_interfaces:
          - network: default
            access_configs:
              - name: External NAT
                type: ONE_TO_ONE_NAT
        zone: us-central1-a
        state: present
        service_account_file: "{{ lookup('env', 'GCP_SERVICE_ACCOUNT_FILE') }}"
        project: "{{ lookup('env', 'GCP_PROJECT') }}"
      register: instance

    - debug:
        msg: "Created instance with ID {{ instance.id }} and IP {{ instance.network_interfaces[0].access_configs[0].nat_ip }}"
7. Run the Ansible Playbook
Execute the Ansible playbook to provision the GCP resources.
sh
Copy Code


ansible-playbook -i gcp_inventory.yaml create_instance.yaml
Monitor the output to ensure the resources are created successfully.
8. Manage Resources
Use Ansible playbooks to manage other GCP resources like storage, databases, and networking.
Structure your playbooks into roles for better organization and reusability.
9. Implement Security Best Practices
Avoid hardcoding sensitive data (e.g., credentials) in playbooks. Use Ansible Vault to encrypt sensitive variables.
Restrict access to the service account key and use least privilege principle for IAM roles.
10. Continuous Integration and Deployment
Integrate Ansible with CI/CD pipelines (e.g., Jenkins, GitHub Actions, GitLab CI) to automate the deployment process.
Ensure the pipeline includes steps for linting, testing, and executing Ansible playbooks.
11. Logging and Monitoring
Set up logging and monitoring for your GCP resources using Stackdriver (now part of Google Cloud Operations Suite).
Use Ansible to automate the setup of logging and monitoring agents.
12. Clean Up Resources
Create playbooks to teardown or clean up resources when no longer needed.
yaml
Copy Code


# destroy_instance.yaml
- name: Destroy GCP Instance
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Delete the GCP compute instance
      google.cloud.gcp_compute_instance:
        name: test-instance
        zone: us-central1-a
        state: absent
        service_account_file: "{{ lookup('env', 'GCP_SERVICE_ACCOUNT_FILE') }}"
        project: "{{ lookup('env', 'GCP_PROJECT') }}"
Execute the playbook to clean up resources.
sh
Copy Code


ansible-playbook -i gcp_inventory.yaml destroy_instance.yaml
Conclusion
By following this guide, you can effectively use Ansible to manage your GCP infrastructure. Ansible's simplicity and flexibility make it a powerful tool for automating cloud deployments, ensuring consistency, and improving operational efficiency.

This guide should help you set up and manage your infrastructure on Google Cloud Platform using Ansible efficiently.



