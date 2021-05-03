# snipeit-docker
Snipe IT supply management system used for uniforms.

## To-Do List
This repo is currently being developed. These items are the things that still need to be worked and added to the README file.

 - Custom domain name configuration
 - SSL configuration (required for SAML integration)

## Server Configuration
> Microsoft Azure Cloud
> Ubuntu 18.04-lts-gen2
> Standard D2s v3 (2vCPUs, 8GiB RAM)

### Example Environment Diagram
![Diagram](https://user-images.githubusercontent.com/28149061/116938724-39564300-ac39-11eb-9de9-bfcbc9b17c7b.png)

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
    # Set up the stable repository.
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    # Update the apt package index, and install the latest version of Docker Engine and containerd
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose

### [Optional] Google LDAP Sync
To allow users to authenticate to the application using their MIWG account (hosted through Google Workspace), we must utilize Snipe-IT's LDAP/SAML features. The application will utilize SAML to authenticate the user, but the user must already have an account within the application first. We will utilize LDAP sync to pull user accounts into the application. The steps we will take are as follows:
 - Configure an LDAP client in Google
 - Generate and download certificates
 - Build an stunnel on Ubuntu host to utilize Google secure LDAP

#### Add an LDAP Client in Google
> Reference: https://support.google.com/a/answer/9048434?hl=en&ref_topic=9173976

 1. Sign in to the Google Admin console at admin.google.com. Be sure to sign in using your super administrator account, and not your personal Gmail account.
 
 2. Go to Apps and then LDAP.
 
 3. Click ADD CLIENT.
 
 4. Type a name in the LDAP client name field—for example, Snipe-IT.
 
 5. Optional: Type a description for the LDAP client—for example, Uniform-tracking app for Michigan Wing hosted in the Azure Cloud. You can also use the description to add contact details or to specify the owner of the app.
 
 6. Click CONTINUE.

#### Configure Access Permissions for LDAP Client
> Reference: https://support.google.com/a/answer/9058751
 
 1. Choose the correct Access permissions for the client. For example, if you have a top-level OU for users, choose the option for selected organizational units and select your top-level user OU.
 
 2. Choose the correct Access level for verifying user credentials. Just like the above example, if you have a top-level user OU, choose the option for the selected organizational units and select your top-level user OU.
 
 3. Click Add LDAP Client
 
 4. Download the generated certificate. This should be a zip file with a certificate and a key file.

#### Uploading Certificate and Key File
Push the certificate and key file you just downloaded to the ubuntu host. This could be via secure copy, SFTP, or through a terminal emulator like MobaXTerm. These files could be placed anywhere on the system so long as the full file path is listed in the stunnel configuration. However, it's recommended to place these files in the **/etc/stunnel/** directory. You should also secure the key file by only allowing root and the owner to read it. You can do this by running this command:

    chmod 600 /path/to/Google_XXXX_XX_XX_XXXXX.pem

#### Build an stunnel
> Reference: https://support.google.com/a/answer/9089736

Snipe-IT doesn't offer a way to authenticate to LDAP with a client certificate, so we will use a stunnel (secure tunnel) as a proxy. This stunnel will be bound to a local port on the host machine that the application can send traffic to in order to reach Google's LDAP directory. These are the steps required in order to that.

 1. Install stunnel. For example, on Ubuntu:

        sudo apt install stunnel4
 
 2. Create a configuration file **/etc/stunnel/google-ldap.conf** with the following contents (replace the file names with the downloaded certificate and key file names along with their full directory path):

        [ldap]
        client = yes
        accept = 127.0.0.1:1636
        connect = ldap.google.com:636
        cert = /home/user/Google_XXXX_XX_XX_XXXXX.crt
        key = /home/user/Google_XXX_XX_XX_XXXXX.key
 
 3. To enable stunnel, edit **/etc/default/stunnel4** and set **ENABLED=1**.
 
 4. Restart stunnel.

        sudo /etc/init.d/stunnel4 restart
 
This should succesfully create the secure tunnel. Later on, we will configure the application to point ldap requests to **ldap://127.0.0.1:1636**

## App Installation

### Clone GitHub Repository
This will copy the docker compose file and template .env file to the server. Assuming the server was created in the Azure cloud, git will come pre-installed. If you choose to use another server, make sure you install the git package prior to this.

    git clone https://github.com/cap-miwg/snipeit-docker.git

### Configure Docker Compose File
The docker compose file is a yaml file that contains the required configuration and variables the containers will utilize when started. You will be able to fill out everything except the API key which will be generated in the next step (Assuming this is the first time running the containers). Edit this file to your needs. More info [here.](https://snipe-it.readme.io/docs/docker)

### [First Time] Generate API Key
You will need to generate an application key to paste into your docker-compose file. The following command will pull the snipe-it container image and run it. Snipe-IT will generate an application key and exit. The --rm flag will automatically remove the container when it exits. Take the generated application key and update the corresponding variable in the docker compose file.

    docker run --rm snipe/snipe-it

### Test Setup
Bring all the containers up with the following command to test. This attaches the terminal to the container to see logs. Use ctrl+c to exit and shutdown the containers successfully. Fix any errors shown in the logs and/or move on.

    docker-compose up 

### Run
If everything is running fine, run the following command. The -d flag runs the container detached from the terminal.

    docker-compose up -d
