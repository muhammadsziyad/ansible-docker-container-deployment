
# Automating Docker Installation and Container Deployment

This project uses Ansible to automate the installation of Docker on a group of servers and deploy a simple Docker container.

## Prerequisites

1. **Ansible Installed**: Ensure Ansible is installed on your control node (the machine from which you'll run Ansible).

   **To Install Ansible on a Red Hat-based system:**
   ```bash
   sudo yum install epel-release -y
   sudo yum install ansible -y
   ```

2. **SSH Access**: Ensure you have SSH access to all target servers where Docker will be installed. Setting up SSH keys for passwordless access is recommended.

## Project Setup

### Step 1: Set Up Your Ansible Environment

1. **Verify Ansible Installation**:
   ```bash
   ansible --version
   ```

2. **Set Up SSH Access**: 
   - Make sure you have SSH access to the target servers. You might need to configure SSH keys for passwordless login.

### Step 2: Create Your Inventory File

1. **Create an Inventory File**:
   - Create a file named `hosts` in your project directory with the following content:
   ```ini
   [docker_hosts]
   server1.example.com
   server2.example.com
   ```
   - Replace `server1.example.com` and `server2.example.com` with the actual hostnames or IP addresses of your target servers.

### Step 3: Write the Ansible Playbook

1. **Create a Playbook File**:
   - Create a file named `setup_docker.yml` in your project directory with the following content:
   
   ```yaml
   ---
   - name: Install Docker and run a container
     hosts: docker_hosts
     become: yes

     tasks:
       - name: Install required system packages
         yum:
           name: "{{ item }}"
           state: present
         loop:
           - yum-utils
           - device-mapper-persistent-data
           - lvm2

       - name: Add Docker repository
         command: >
           yum-config-manager --add-repo
           https://download.docker.com/linux/centos/docker-ce.repo
         args:
           creates: /etc/yum.repos.d/docker-ce.repo

       - name: Install Docker
         yum:
           name: docker-ce
           state: present
           enablerepo: docker-ce-stable

       - name: Start and enable Docker
         service:
           name: docker
           state: started
           enabled: yes

       - name: Run a simple Docker container (Hello World)
         docker_container:
           name: hello-world
           image: hello-world
           state: started
   ```

   **Explanation of Playbook**:
   - **Install Required Packages**: Installs necessary system packages for Docker.
   - **Add Docker Repository**: Adds the official Docker repository to the system so that Docker can be installed.
   - **Install Docker**: Installs Docker from the added repository.
   - **Start and Enable Docker**: Starts Docker service and ensures it starts on boot.
   - **Run a Docker Container**: Pulls the `hello-world` Docker image and runs it to test Docker installation.

### Step 4: Run Your Ansible Playbook

1. **Execute the Playbook**:
   - Run the playbook using the following command:
   ```bash
   ansible-playbook -i hosts setup_docker.yml
   ```
   - This command tells Ansible to use the `hosts` inventory file and run the tasks defined in the `setup_docker.yml` playbook on the servers listed under the `[docker_hosts]` group.

2. **Verify Docker Installation and Container Execution**:
   - SSH into one of your servers and check if Docker is running:
   ```bash
   systemctl status docker
   ```
   - To verify that the `hello-world` container ran successfully, use:
   ```bash
   docker ps -a
   ```

### Step 5: Customize and Extend Your Ansible Project

- **Deploy Different Docker Containers**: Modify the playbook to deploy different Docker containers or services as needed.
- **Create Docker Networks**: Use Ansible to create Docker networks if your containers need to communicate with each other.
- **Use Docker Compose**: Extend the playbook to install Docker Compose and deploy multi-container applications using Compose files.

## Conclusion

This project demonstrates how to automate the installation of Docker and the deployment of a simple Docker container using Ansible. Feel free to extend this project to suit your needs for managing Docker across multiple servers.
