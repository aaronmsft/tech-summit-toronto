# Hands on Lab 100 - Using Docker Hub + Django Poll Application   [ Using Windows Environment ] 

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


## Exercise 1: Create native Windows Environment

### Step 1 : Before you install Docker on Windows 

Requires Microsoft Windows 10 Professional or Enterprise 64-bit. Install [Docker Toolbox](https://docs.docker.com/toolbox/overview/) for older version of Windows 

Documentation : [Docker on Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows)
### Step 2 : Setup Docker on Windows 10 (Video) 
**Check out the video on how to install docker on Windows**:[https://www.youtube.com/watch?v=S7NVloq0EBc](https://www.youtube.com/watch?v=S7NVloq0EBc)

For manual isntructions see below .... 

### Step 3 : Install
Double-click Docker for Windows Installer to run the installer. 

**[Get Docker CE for Windows](https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe)**

When the installation finishes, Docker starts automatically. The whale  in the notification area indicates that Docker is running, and accessible from a terminal.

### Step 4 : Verify Docker installation

Open Command prompt or Powershell in administrator mode and run the following commands to set up Docker engine and CLI tools

 ```
PS C:\Users\jdoe> docker --version
Docker version 17.03.0-ce, build 60ccb22

PS C:\Users\jdoe> docker run hello-world

Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
1. The Docker client contacted the Docker daemon.
2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
3. The Docker daemon created a new container from that image which runs the executable that produces the output you are currently reading.
4. The Docker daemon streamed that output to the Docker client, which sent it to your terminal.
```

## Exercise 2 : Dockerizing your application 

### Step 1 : Pull the image files locally  

Install [GIT](https://git-scm.com/download/win) if not already installed on your machine
    
    # Create a folder and go to that folder   
    mkdir docker 
    cd docker  
    
    # Now Clone the repo locally  
    git clone https://github.com/Azure-Samples/docker-django-webapp-linux.git  
    cd docker-django-webapp-linux 

## Step 2: Locally build a Docker image  
**Always run docker CLI commands on administrator mode in Powershell on Command Prompt**

    # You can edit the app by making changes to the html files under /app/templates/app folder. use vim [filepath] to open the file to edit. Here are commands if you are not familiar with vim  
    
    # Now  build the image  [Usage : docker build -t [Image Name]:[tag]  Dockerfile Path]. Since you are inside  docker-django-webapp-linux folder you can use . to select the docker file in the current working directory 

    docker build -t starterapp:latest . 
    
    # Run docker images to see your image listed. Make a note of the IMAGE ID for your built image
    docker images
    
    # Sample output:
    REPOSITORY                                         TAG                 IMAGE ID                         CREATED             SIZE 
    starterapp                                         latest              <your_image_id>                  18 minutes ago      735MB? 


## Exercise 3 : Create Azure Container registry (ACR) Server for your image

### Step 1: Login to Azure and Launch Azure Cloud Shell 

    Login via portal and launch cloud shell  https://docs.microsoft.com/en-us/azure/cloud-shell/quickstart  
    
    1. Launch?Cloud Shell?from the top navigation of the Azure portal? 
    If it's the first time you open the Cloud Shell:
    2. Select a subscription to create a storage account and Azure file share 
    3. Select "Create storage" 

    Tip

    - You are automatically authenticated for Azure CLI 2.0 in every session.
    - If you have multiple subscriptions, please use the following to choose the default subscription as the Azure Pass provided to you 
    
    az account set --subscription my-subscription-name 

### Step 2: Create a resource group 

    # Use the Azure Cloud Shell console for the next commands.
    
    # Use the[az appservice list-locations](https://docs.microsoft.com/en-us/cli/azure/appservice?view=azure-cli-latest#list-locations) Azure CLI command to list available locations. 
    # Create a resource group for the ACR and web app 
    
    az group create --name myResourceGroup --location "West US"    
   

### Step 3: Create Azure Container Registry 

    # Create a ACR container registry. Choose a unique name instead of myContainerRegistry

     az acr create --resource-group myResourceGroup --name myContainerRegistry --sku Basic
     
    # Enable admin credentials on the new registry
    
    az acr update -n myContainerRegistry --admin-enabled true
     
    # Get Login credentials 
   
    az acr credential show --name myContainerRegistry
  
### Step 4: Upload your image to the ACR registry 

    # Use your previously used local command line (PowerShell or CMD) for the next commands. 
    
    # Login to ACR to pull or push images. Use the credentials received from the previous command. Ignore the security warning. Run docker CLI commands on administrator mode in Powershell on Command Prompt

    docker login myContainerRegistry.azurecr.io -u <YOUR-USERNAME> -p <YOUR-PASSOWRD>    
            
    # Tag the locally built image to ACR repo. Replace ```your_image_id``` with Image ID from locally building your image above. 
   
    docker tag <your_image_id> myContainerRegistry.azurecr.io/starterapp:latest

    # Push docker image to ACR . Replace ```<acrLoginServer>``` with the login server name of your ACR instance.
   
    docker push myContainerRegistry.azurecr.io/starterapp:latest
 
    # Verify the Push was successful. Do it in the Azure Cloud Shell
   
    az acr repository list -n myContainerRegistry

## Exercise 4: Create a Web App on Azure 

### Step 1 : Create an app service plan
  
    #Executenext commands in the Azure Cloud Shell

    az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku S1 --is-linux

### Step 2 :Create a web app. Give it a unique name. Specify any runtime (it will be replaced later)

    az webapp create --name <app_name> --resource-group myResourceGroup --plan myAppServicePlan --deployment-container-image-name <your-docker-user-name>/starterapp:latest
    
### Step 3: Configure web app to use ACR image 
```az webapp config container set``` command to assign the custom Docker image to the web app. Replace <app_name>, <docker-registry-server-url>, <registry-username>, and <password>. For Azure Container Registry, <docker-registry-server-url> is in the format https://<azure-container-registry-name>.azurecr.io.
 
       az webapp config container set --name <app_name> --resource-group myResourceGroup --docker-custom-image-name  myContainerRegistry.azurecr.io/starterapp --docker-registry-server-url https://myContainerRegistry.azurecr.io --docker-registry-server-user <registry-username> --docker-registry-server-password <password>
 
### Step 4 : Restart your app

    # Run this command 
     
    az webapp restart --resource-group myResourceGroup --name <your_app_name>

### Step 5: Browse your app

    https://<your_app_name>.azurewebsites.net 

### Step 6 :  Push an update to Docker image 

Go back to Azure Virtual machine to make more changes. Build the image and then push it to your Docker Hub repository. Follow the steps above to do the same

### Step 7:  Browse the app 

    http://<your_app_name>.azurewebsites.net

## Exercise 5:  Configure Continuous Integration/Continuous deployment on ACR 

### Step 1 : Obtain a webhook

    # You can obtain the Webhook URL 
     
    az webapp deployment container show-cd-url -n <your_app_name> -g myResourceGroup

    # For the Webhook URL, you need to have the following endpoint: 
    
    https://<publishingusername>:<publishingpwd>@<your_app_name>.scm.azurewebsites.net/docker/hook

    # You can obtain your publishingusername and publishingpwd by downloading the web app publish profile using the Azure portal.
    
### Step 2: Add a webhook to ACR 
 Replace ```<webhook-url-web app>``` with web hook URL endpoint ```https://<publishingusername>:<publishingpwd>@<your_app_name>.scm.azurewebsites.net/docker/hook```
 
    az acr webhook create --registry myContainerRegistry --name myacrwebhook01 --actions push --uri <webhook-url-web app>
 
 When the image gets updated, the web app get updated automatically with the new image.
 
  
### Step 3: Push an update to Docker image 

Go back to Azure Virtual machine to make more changes. Build the image and then push it to your Docker Hub repository. Follow the steps above to do the same

### Step 4: Browse the app 

    http://<your_app_name>.azurewebsites.net


