# Go App Jenkins Deployment

This repository offers a detailed guide for configuring AWS EC2 instances as node agents for Jenkins. It illustrates the effective utilization of distinct nodes dedicated to executing environment-specific pipeline tasks—such as handling development processes on one node and managing production tasks on another. The implementation of this setup is exemplified through the deployment of a Go application.

## Project Setup Guide

### 1. Creating Virtual Machines (VMs)

To begin, set up the required VMs using AWS EC2:

- **Jenkins Master VM**: This VM will host Jenkins.
- **Development Node (dev-node) VM**: For running development environment jobs.
- **Production Node (prod-node) VM**: For running production environment jobs.

Ensure the following:
- Use Ubuntu 20.04 LTS for all VMs, or choose an operating system according to your preference.
- Create separate key pairs for each VM.
- Allow SSH port 22 and relevant ports like 8080 and 80 for all VMs.

### 2. SSH Key Exchange

Enable SSH key-based authentication between Jenkins master and nodes:

- Copy the Jenkins master's public key to both development and production nodes' `~/.ssh/authorized_keys`.

> *Note:* You can find the Jenkins master's public key by SSHing into the VM and locating it at `~/.ssh/authorized_keys`.


### 3. Setting Up VMs and Installing Necessary Software

- **Jenkins Master VM:**
   - Install Jenkins by following the steps [here](https://www.jenkins.io/doc/book/installing/linux/#debianubuntu).
   - After installation, access Jenkins at `http://<jenkins-master-public-ip>:8080`.
   - Install necessary plugins like "SSH Build Agents" and "Pipeline" from "Manage Jenkins" > "Plugins" page.

- **Development and Production Nodes (dev-node and prod-node):**
   - Install Git if not already installed.
   - Download and install Go. You can use the following commands:
     ```bash
     # Download the Go archive
     sudo wget https://go.dev/dl/go1.22.2.linux-amd64.tar.gz
     
     # Extract the downloaded archive to /usr/local
     sudo tar -C /usr/local -xzf go1.22.2.linux-amd64.tar.gz
     ```
   - Update the PATH environment variable:
   
        You can do this by adding the following line to your $HOME/.profile or /etc/profile (for a system-wide installation):
     ```
     export PATH=$PATH:/usr/local/go/bin
     ```
     Then, reload your shell or run source ~/.bashrc to apply the changes.

   - Verify the installation using `go version`.

### 4. Cloning the Repository

- Create the directory
  ```
  sudo mkdir -p /opt/go-app
  ```

- SSH into dev-node VM and clone the repository into `/opt/go-app` directory using:
  ```
  sudo git clone <repository_url> --branch dev /opt/go-app
  ```
- Similarly, clone the repository into `/opt/go-app` on prod-node VM using the main branch.
  ```
  sudo git clone <repository_url> --branch main /opt/go-app
  ```

### 5. Create jenkins user

For both dev-node VM and prod-node VM, follow these steps:

- **Create the Jenkins user:**
    ```bash
    sudo adduser jenkins
    ```

- **Set permissions for the /opt/ directory:**
    ```bash
    sudo chown -R jenkins:jenkins /opt/
    sudo chmod -R 755 /opt/
    ```

- **Assign sudo privileges:**
    ```bash
    sudo usermod -aG sudo jenkins
    ```

### 6. Configure Jenkins with SSH Keys

- Log in to the Jenkins dashboard.
- Navigate to "Manage Jenkins" > "Credentials" > "Global credentials".
- Click on "Add credentials" and select "SSH Username with private key" as the kind.
- Enter the username (typically ubuntu for Ubuntu VMs) and paste the contents of the private key associated with the Jenkins master.
- Repeat the above steps to add SSH credentials for both dev-node and prod-node, using the appropriate private key for each VM.

### 7. Configure Nodes in Jenkins

- Navigate to "Manage Jenkins" > "Manage Nodes and Clouds" > "New Node" for each node (development and production).
- Enter a name for each node (e.g., "dev-node", "prod-node").
- Choose "Permanent Agent" as the mode of connection.
- Enter /opt under Remote root directory.
- Specify labels for each node (e.g., "dev" for development node, "prod" for production node).
- Select "Launch agent via SSH" as the mode of launching agents.
- Enter the corresponding IP address or hostname of each node.
- Choose the SSH credentials you created in the previous step.
- Under Host Key Verification Strategy, select Non verifying verification strategy.
- Click on "Save" or "Save and Apply" for each node.

### 8. Update Pipeline Script (Optional)

If you are using any other name for labels instead of “dev” and “prod”, make sure to update the pipeline script to utilize the specified labels for each stage.

### 9. Create and Configure the Jenkins Job

- On the left of the Jenkins dashboard, click on "New Item".
- Enter the job name “go-app-deployment”.
- Select "Pipeline" job and click on "OK".
- Under "Pipeline" section, keep "Pipeline script" as Definition and add code from the “Jenkinsfile” present in the repository in the Script.
- Finally, save the job and click "Build Now" to initiate the pipeline execution.

---

### Note:

This project utilizes code from the [go-webapp-sample repository](https://github.com/ybkuroki/go-webapp-sample), which is licensed under the MIT License. For more detailed information about the Go application, please refer to the README of the go-webapp-sample repository.
