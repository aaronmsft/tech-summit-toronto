# Hands on Lab 100 - Using Docker Hub + Django Poll Application  

The purpose of this Hands on Lab (HOL) is to have an understanding of how to:
1. Install Docker Host and Docker CLI tools
2. Build a custom Docker Image
3. Push your custom Docker Image to a Docker Registry (this can be public/private Docker Hub or a private Azure Container Registry aka "ACR")
4. Create an Azure App Service (Specifically a "Web App for Containers" Service)
5. Deploy an instance of your custom Docker Image (container) to Azure App Service
6. Create a webhook to update your Azure App Service when your custom Docker Image has been updated (new updates pushed to the Container Registry)

 
## Notes: 

1. Docker Hub can be replaced with Azure Container Registry (ACR). Use the directions to push an image to ACR instead of Docker Hub related commands in the lab below: https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-docker-cli
2. You will require access to a native Linux Environment for this Hands on Lab (HOL)

# Tasks

## Create a native Linux Environment
    You will require access to a native Linux Enviroment for this Hands on Lab

    You can use ONE (1) of the follow three (3) options:
    1. Use your laptop with a Linux Distro installed (Preferably Ubuntu as per our examples below)
    OR
    2. Windows Subsystem for Linux installed on your Windows 10 Laptop: Installation instructions [here](https://docs.microsoft.com/en-us/windows/wsl/install-win10)
    OR
    3. A Linux VM on Azure: Click this [link](https://portal.azure.com/?feature.customportal=false#create/Canonical.UbuntuServer1710-ARM) to create a Ubuntu VM in Azure
        Note:
            - For ease of creation, use Password authentication instead of SSH Key
                - Password must be min of 12 characters with at least one Uppercase, one numeric or special character 
            - Choose DS1V2 for Virtual machine size 
            - Once the machine is deployed successfully, make a note of the virtual machine's Public IP address 

## Install Putty to access your VM (Windows Users Only!)

    Download putty.exe (https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html ) locally on your laptop to SSH into the virtual machine
    Open Putty.exe and enter the IP address in Host text field and click on Open
     
    You will see an alert, click on Yes 
     
    Login with the credentials you configured during Step 1 (Create a Linux Bash Shell) 

## Install Docker on the linux machine  

    # Run the following commands to set up Docker engine and CLI tools

    # Update your repository manager
    sudo apt-get update 

    # Install package dependencies
    sudo apt-get install apt-transport-https ca-certificates curl software-properties-common 
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - 

    # Add the docker gpg keys
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" 
      
    # Install Docker CE
    sudo apt-get update 
    sudo apt-get install docker-ce 
      
    # Verify Docker Installation 
    sudo docker run hello-world 

## Pull the image files locally  

    # Run git --version to check if git is installed. If not installed, then install git using sudo apt-get install git  
    
    # Create a folder and go to that folder   
    mkdir docker 
    cd docker  
    
    # Now Clone the repo locally  
    git clone https://github.com/Azure-Samples/docker-django-webapp-linux.git  
    cd docker-django-webapp-linux 

## Locally build a Docker image  

    # You can edit the app by making changes to the html files under /app/templates/app folder. use vim [filepath] to open the file to edit. Here are commands if you are not familiar with vim  
      
    # Now  build the image  [Usage : docker build -t [Image Name]:[tag]  Dockerfile Path]. Since you are inside  docker-django-webapp-linux folder you can use . to select the docker file in the current working directory 

    sudo docker build -t starterapp:latest . 
    
    # Run docker images to see your image listed. Make a note of the IMAGE ID for your built image
    sudo docker images
    
    # Sample output:
    REPOSITORY                                         TAG                 IMAGE ID                         CREATED             SIZE 
    django-starter-app                                 latest              <your_image_id>                  18 minutes ago      735MB  

## Create or Login to Docker Hub account to push docker file  

    # If you don’t have any account, create a new account https://hub.docker.com 
    
    # If you have an account, in the putty.exe console type docker login (you must be connected to your Linux Server). 
    # Enter your credentials to login to docker hub.
    
    # Now tag the locally built image to Docker Hub repo. 
    sudo docker tag <your_image_id> <your-docker-user-name>/starterapp:latest
    
    # Now push the image to Docker Hub
    
    sudo docker push  <your-docker-user-name>/starterapp:latest                             
    
    # You should see the repo created in your Docker Hub page 

## Login to Azure and Launch Azure Cloud Shell 

    Login via portal and launch cloud shell  https://docs.microsoft.com/en-us/azure/cloud-shell/quickstart  
    
    1. Launch Cloud Shell from the top navigation of the Azure portal  
    2. Select a subscription to create a storage account and Azure file share 
    3. Select "Create storage" 

    Tip

    You are automatically authenticated for Azure CLI 2.0 in every session. 
      
    If you have multiple subscriptions, please use the following to choose the default subscription as the Azure Pass provided to you  
      
    az account set --subscription my-subscription-name 

## Create a resource group 

    az group create --name myResourceGroup --location "West US" 
      
    # Use the [az appservice list-locations](https://docs.microsoft.com/en-us/cli/azure/appservice?view=azure-cli-latest#list-locations) Azure CLI command to list available locations. 

## Create an app service plan

    az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku S1 --is-linux

## Create a web app

    az webapp create --name <app_name> --resource-group myResourceGroup --plan myAppServicePlan --deployment-container-image-name <your-docker-user-name>/starterapp:latest

## Restart your app

    # Run this command 
     
    az webapp restart --resource-group myResourceGroup --name <your_app_name>

## Browse your app

    https://<your_app_name>.azurewebsites.net 

## Enable Continuous deployment

You can enable the continuous deployment feature using Azure CLI and executing the following command

```
az webapp deployment container config --name <app_name> --resource-group myResourceGroup --enable-cd true
```

## Obtain a webhook

    # You can obtain the Webhook URL 
     
    az webapp deployment container show-cd-url -n <app_name> -g myResourceGroup

    # For the Webhook URL, you need to have the following endpoint: 
    
    https://<publishingusername>:<publishingpwd>@<your_app_name>.scm.azurewebsites.net/docker/hook

    # You can obtain your publishingusername and publishingpwd by downloading the web app publish profile using the Azure portal.1

## Add Web Hook with Docker Hub

Go to your Docker Hub page , click Webhooks, then [CREATE A WEBHOOK](https://docs.docker.com/docker-hub/webhooks/) . With your webhook, you specify a target URL as created above ```https://<publishingusername>:<publishingpwd>@<sitename>.scm.azurewebsites.net/docker/hook.```

 
##  Push an update to Docker image 

Go back to Azure Virtual machine to make more changes. Build the image and then push it to your Docker Hub repository. Follow the steps above to do the same

## Browse the app 

    http://<your_app_name>.azurewebsites.net

