# Hands on Lab 100 - Using Docker Hub + Django Poll Application  

The purpose of this Hands on Lab (HOL) is to have an understanding of how to:
1. Install Docker Host and Docker CLI tools
2. Build a custom Docker Image
3. Push your custom Docker Image to a Docker Registry (this can be public/private Docker Hub or a private Azure Container Registry aka "ACR")
4. Create an Azure App Service (Specifically a "Web App for Containers" Service)
5. Deploy an instance of your custom Docker Image (container) to Azure App Service
6. Create a webhook to update your Azure App Service when your custom Docker Image has been updated (new updates pushed to the Container Registry)

 
## Pre-requisites:

1. Azure Subscription or Sign up for Azure Fre Trial
2. You will require access to a Windows Environment with Docker for this Hands on Lab (HOL)

# Tasks

## Before you install Docker on Windows 

**README FIRST for Docker Toolbox and Docker Machine users**: Docker for Windows requires Microsoft Hyper-V to run. The Docker for Windows installer enables Hyper-V for you, if needed, and restart your machine. After Hyper-V is enabled, VirtualBox no longer works, but any VirtualBox VM images remain. VirtualBox VMs created with docker-machine (including the default one typically created during Toolbox install) no longer start. These VMs cannot be used side-by-side with Docker for Windows. However, you can still use docker-machine to manage remote VMs.   

1. Virtualization must be enabled. Typically, virtualization is enabled by default. This is different from having Hyper-V enabled. For more detail see Virtualization must be enabled in Troubleshooting.
2.  The current version of Docker for Windows runs on 64bit Windows 10 Pro, Enterprise and Education (1607 Anniversary Update, Build 14393 or later).
3. Containers and images created with Docker for Windows are shared between all user accounts on machines where it is installed. This is because all Windows accounts use the same VM to build and run containers.

## Install Docker Tool box 
Install [Docker Toolbox](https://docs.docker.com/toolbox/overview/), which uses Oracle Virtual Box instead of Hyper-V. 
What the Docker for Windows install includes: The installation provides Docker Engine, Docker CLI client, Docker Compose, Docker Machine, and Kitematic.

For  full documentation on installing Docker toolbox , click [here](https://docs.docker.com/toolbox/toolbox_install_windows/#step-2-install-docker-toolbox)

## Check if docker is installed 

Open Command prompt or Powershell in administrator mode and run the following commands to set up Docker engine and CLI tools

 ```
PS C:\Users\Docker> docker --version
Docker version 17.03.0-ce, build 60ccb22

PS C:\Users\Docker> docker-compose --version
docker-compose version 1.11.2, build dfed245

PS C:\Users\Docker> docker-machine --version
docker-machine version 0.10.0, build 76ed2a6
```

## Pull the image files locally  

Install [GIT](https://git-scm.com/download/win) if not already installed on your machine
    
    # Create a folder and go to that folder   
    mkdir docker 
    cd docker  
    
    # Now Clone the repo locally  
    git clone https://github.com/Azure-Samples/docker-django-webapp-linux.git  
    cd docker-django-webapp-linux 

## Locally build a Docker image  

    # You can edit the app by making changes to the html files under /app/templates/app folder. use vim [filepath] to open the file to edit. Here are commands if you are not familiar with vim  
    ? 
    # Now  build the image  [Usage : docker build -t [Image Name]:[tag]  Dockerfile Path]. Since you are inside  docker-django-webapp-linux folder you can use . to select the docker file in the current working directory 

    docker build -t starterapp:latest . 
    
    # Run docker images to see your image listed. Make a note of the IMAGE ID for your built image
    docker images
    
    # Sample output:
    REPOSITORY                                         TAG                 IMAGE ID                         CREATED             SIZE 
    django-starter-app                                 latest              <your_image_id>                  18 minutes ago      735MB? 
   
## Login to Azure and Launch Azure Cloud Shell 

    Login via portal and launch cloud shell  https://docs.microsoft.com/en-us/azure/cloud-shell/quickstart  
    
    1. Launch?Cloud Shell?from the top navigation of the Azure portal? 
    2. Select a subscription to create a storage account and Azure file share 
    3. Select "Create storage" 

    Tip

    - You are automatically authenticated for Azure CLI 2.0 in every session.
    - If you have multiple subscriptions, please use the following to choose the default subscription as the Azure Pass provided to you 
    
    az account set --subscription my-subscription-name 

## Create a resource group 

    # Use the[az appservice list-locations](https://docs.microsoft.com/en-us/cli/azure/appservice?view=azure-cli-latest#list-locations) Azure CLI command to list available locations. 
    # Create a resource group for the ACR and web app 
    
    az group create --name myResourceGroup --location "West US"    
   

## Create Azure Container registry 

    # Create ACR server

     az acr create --resource-group myResourceGroup --name myContainerRegistry --sku Basic
     
    # Get Login credentials 
   
    az acr credential show --name myContainerRegistry 
    
    # Login to ACR to pull or push images . Use the credentials received from the previous command

    docker login myContainerRegistry.azurecr.io -u <YOUR-USERNAME> -p <YOUR-PASSOWRD>    
            
    # Tag the locally built image to ACR repo. Replace ```your_image_id``` with Image ID from locally building your image above. 
   
    docker tag <your_image_id> myContainerRegistry.azurecr.io/starterapp:latest

    # Push docker image to ACR . Replace ```<acrLoginServer>``` with the login server name of your ACR instance.
   
    docker push myContainerRegistry.azurecr.io/starterapp:latest
 
    # Verify the Push was successful 
   
    az acr repository list -n myContainerRegistry.azurecr.io
   
## Create an app service plan

    az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku S1 --is-linux

## Create a web app

    az webapp create --name <app_name> --resource-group myResourceGroup --plan myAppServicePlan 
    
## Configure web app to use ACR image 
```az webapp config container set``` command to assign the custom Docker image to the web app. Replace <app_name>, <docker-registry-server-url>, <registry-username>, and <password>. For Azure Container Registry, <docker-registry-server-url> is in the format https://<azure-container-registry-name>.azurecr.io.
 
       az webapp config container set --name <app_name> --resource-group myResourceGroup --docker-custom-image-name  myContainerRegistry.azurecr.io/starterapp --docker-registry-server-url https://myContainerRegistry.azurecr.io --docker-registry-server-user <registry-username> --docker-registry-server-password <password>
 

## Restart your app

    # Run this command 
     
    az webapp restart --resource-group myResourceGroup --name <your_app_name>

## Browse your app

    https://<your_app_name>.azurewebsites.net 

##  Push an update to Docker image 

Go back to Azure Virtual machine to make more changes. Build the image and then push it to your Docker Hub repository. Follow the steps above to do the same

## Browse the app 

    http://<your_app_name>.azurewebsites.net

# Configure CI/CD with ACR 

## Obtain a webhook

    # You can obtain the Webhook URL 
     
    az webapp deployment container show-cd-url -n sname1 -g rgname

    # For the Webhook URL, you need to have the following endpoint: 
    
    https://<publishingusername>:<publishingpwd>@<your_app_name>.scm.azurewebsites.net/docker/hook

    # You can obtain your publishingusername and publishingpwd by downloading the web app publish profile using the Azure portal.
    
## Add a webhook to ACR 
 Replace ```<webhook-url-web app>``` with web hook URL endpoint ```https://<publishingusername>:<publishingpwd>@<your_app_name>.scm.azurewebsites.net/docker/hook```
 
    az acr webhook create --registry myContainerRegistry --name myacrwebhook01 --actions push --uri <webhook-url-web app>
 
 When the image gets updated, the web app get updated automatically with the new image.
 
  
##  Push an update to Docker image 

Go back to Azure Virtual machine to make more changes. Build the image and then push it to your Docker Hub repository. Follow the steps above to do the same

## Browse the app 

    http://<your_app_name>.azurewebsites.net


