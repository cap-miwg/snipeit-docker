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
	sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
	sudo apt-get update
	sudo apt-get install docker-ce docker-ce-cli containerd.io
