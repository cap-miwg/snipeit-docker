# snipeit-docker
Snipe IT supply management system used for uniforms.

## Server Configuration
> Microsoft Azure Cloud
> Ubuntu 18.04-lts-gen2
> Standard D2s v3 (2vCPUs, 8GiB RAM)

### Create new Server
Spin up the server through the GUI or through a PowerShell script (not included). Future improvements will include a powershell script to completely start a snipe-it instance in Azure.

### Connect to System
Connect to the newly created server via SSH using your favorite choice of terminal. Try checking out MobaXterm!

### Install Docker
Docker will be used to install the applications on top of our server. This streamlines the process of setting up the applications as the docker container will come with all the dependencies within the container. We can then quickly spin them up with a docker compose file.

    # Update the apt package index and install packages to allow apt to use a repository over HTTPS:
    sudo apt-get update
    sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
    # Add Docker's official GPG key:
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    # Update the apt package index, and install the latest version of Docker Engine and containerd
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io

### Clone GitHub Repository
This will copy the docker compose file to the server. Assuming the server was created in the Azure cloud, it will come with the git package. If you choose to use another server, make sure you install the git package prior to this.
    git clone https://github.com/cap-miwg/snipeit-docker.git
