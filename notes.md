# Developing Solutions for Microsoft Azure
Author: Calin Simon
Date: now

[1 Develop Azure Compute Solutions (25-30%)](#1)
[1.1 Implement IaaS solutions](#1.1)
[1.2 Create Azure App Service Web Apps](#1.2)
[1.3 Implement Azure Functions](#1.3)
[2 Develop for Azure Storage (15-20%)](#2)
[2.1 Develop solutions that use Cosmos DB storage](#2.1)
[2.2 Develop solutions that use blob storage](#2.2)
[3 Implement Azure Security (20-25%)](#3)
[3.1 Implement user authentication and authorization](#3.1)
[3.2 Implement secure cloud solutions](#3.2)

## 1 Develop Azure Compute Solutions (25-30%) {#1}

### 1.1 Implement IaaS solutions {#1.1}

#### Pluralsight notes

##### Provisioning and Configuring Azure Virtual Machines
Virtual Machine Components
- VMs are contained in Resource Groups and deployed in specific regions.
- VM size (CPU, ram, etc.). The size is easily modified after deployment
- Network. A VM has internal IP address, for private networking. A public IP address can be assigned as well.
- VM Images. Images are the base image used to deploy on to our VM, most often Windows or Linux. Many different images are available from the Marketplace.
- Virtual Disk. At least one per VM to host the OS.

Methods to create an Azure Virtual Machine
- Azure Portal
First, you define the basics, disks, networking, management, advancement, tags. Then, you define the instance details: name, region, availability options, image (available in selecte region), size, azure spot instance (setting to let Azure take compute power from you if needed). Next, create a username and password for admin control of the VM. With linux, you can also add a SSH public key. Lastly, select which VM network ports are accessible from the public internet. You can further customize the network configurations from the networking tab.
- Azure CLI
- Azure Powershell (Az Module)
- Azure ARM Templates
- Other: via API's (out of scope of this course)

Creating VMs programmatically
- Add consistency to your deployments and VM creation
- Any production system should be implemented using automation
- Construct similar down-level environments, such as DEV/TEST

Tools for creating a VM Programmatically
- Azure CLI
- Azure Powershell (Az Module)
- Azure ARM templates

Steps to create a VM programmatically:
1. Create a resource group in a region
2. Create the virtual machine
3. Ensure remote access port is open (if needed). You can also add rules to the network security group.
4. Retrieve the public IP address.

__Azure CLI__
```sh
#Login interactively and set a subscription to be the current active subscription
az login
az account set --subscription "Demonstration Account"

#Let's create a Windows VM.
az group list --output table 

#Create a resource group if needed.
az group create \
    --name "psdemo-rg" \
    --location "centralus"

#Creating a Windows Virtual Machine
az vm create \
    --resource-group "psdemo-rg" \
    --name "psdemo-win-cli" \
    --image "win2019datacenter" \
    --admin-username "demoadmin" \
    --admin-password "password123$%^&*" 

#Open RDP for remote access, it may already be open. Note: this opens port 3389 on the network security group that this VM is attached to.
az vm open-port \
    --resource-group "psdemo-rg" \
    --name "psdemo-win-cli" \
    --port "3389"

#Get the IP Addresses for Remote Access
az vm list-ip-addresses \
    --resource-group "psdemo-rg" \
    --name "psdemo-win-cli" \
    --output table

#TODO: Log into the Windows VM via RDP.

#Creating a Linux Virtual Machine
az vm create \
    --resource-group "psdemo-rg" \
    --name "psdemo-linux-cli" \
    --image "UbuntuLTS" \
    --admin-username "demoadmin" \
    --authentication-type "ssh" \
    --ssh-key-value ~/.ssh/id_rsa.pub 

#OpenSSH for remote access, it may already be open
az vm open-port \
    --resource-group "psdemo-rg" \
    --name "psdemo-linux-cli" \
    --port "22"


#Get the IP address for Remote Access
az vm list-ip-addresses \
    --resource-group "psdemo-rg" \
    --name "psdemo-linux-cli" \
    --output table

#Log into the Linux VM over SSH
ssh demoadmin@PASTE_PUBLIC_IP_HERE

#Clean up the resources in this demo for the next demo.
az group delete --name "psdemo-rg"
```
__Azure PowerShell__
```ps1
#Let's login, may launch a browser to authenticate the session.
Connect-AzAccount -SubscriptionName 'Demonstration Account'

#Ensure you're pointed at your correct subscription
Set-AzContext -SubscriptionName 'Demonstration Account'

#Create a Resource Group
New-AzResourceGroup -Name "psdemo-rg" -Location "CentralUS"

#Create a credential to use in the VM creation
$username = 'demoadmin'
$password = ConvertTo-SecureString 'password123$%^&*' -AsPlainText -Force
$WindowsCred = New-Object System.Management.Automation.PSCredential ($username, $password)

#Create a Windows Virtual Machine, can be used for both Windows and Linux.
#Note, you can create Windows or Linux VMs with PowerShell by specifying the correct Image parameter.
New-AzVM `
    -ResourceGroupName 'psdemo-rg' `
    -Name 'psdemo-win-az' `
    -Image 'Win2019Datacenter' `
    -Credential $WindowsCred `
    -OpenPorts 3389

#Get the Public IP Address
Get-AzPublicIpAddress `
    -ResourceGroupName 'psdemo-rg' `
    -Name 'psdemo-win-az' | Select-Object IpAddress

#TODO: Open an RDP Client and log into it using this IP address and the credential defined above.

#Clean up after this demo
Remove-AzResourceGroup -Name 'psdemo-rg'
```
__ARM Templates__
- JSON file that defines your resources
- Building block for automation
- Templates are submitted to ARM for provisioning
- Export an ARM template in Azure Portal
- Write your own
- Deploy from the Quickstart template library

The ARM template format:
- schema: location of the JSON schema file that describes the schema's language 
- contentversion: defined by user
- apiProfile: versioning of resource types that are defined in the template
- parameters { }
- variables { }: values used throughout the template 
- functions [ ]: a commonly used function is one that sets the resource group name based on the environment it is deployed in. Check functions at [this link](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-functions)
- resources [ ]: VMs, storage accounts, etc.
- output { }: return values from the resources being deployed. This allows you to inspect and use the output in systems that are driving the deployment, such as scripts and other deployment tools.
```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string"
        },
        "networkInterfaceName": {
            "type": "string"
        },
        "networkSecurityGroupName": {
            "type": "string"
        },
        "networkSecurityGroupRules": {
            "type": "array"
        },
        "subnetName": {
            "type": "string"
        },
        "virtualNetworkName": {
            "type": "string"
        },
        "addressPrefixes": {
            "type": "array"
        },
        "subnets": {
            "type": "array"
        },
        "publicIpAddressName": {
            "type": "string"
        },
        "publicIpAddressType": {
            "type": "string"
        },
        "publicIpAddressSku": {
            "type": "string"
        },
        "virtualMachineName": {
            "type": "string"
        },
        "virtualMachineComputerName": {
            "type": "string"
        },
        "virtualMachineRG": {
            "type": "string"
        },
        "osDiskType": {
            "type": "string"
        },
        "virtualMachineSize": {
            "type": "string"
        },
        "adminUsername": {
            "type": "string"
        },
        "adminPassword": {
            "type": "secureString"
        },
        "patchMode": {
            "type": "string"
        }
    },
    "variables": {
        "nsgId": "[resourceId(resourceGroup().name, 'Microsoft.Network/networkSecurityGroups', parameters('networkSecurityGroupName'))]",
        "vnetId": "[resourceId(resourceGroup().name,'Microsoft.Network/virtualNetworks', parameters('virtualNetworkName'))]",
        "subnetRef": "[concat(variables('vnetId'), '/subnets/', parameters('subnetName'))]"
    },
    "resources": [
        {
            "name": "[parameters('networkInterfaceName')]",
            "type": "Microsoft.Network/networkInterfaces",
            "apiVersion": "2018-10-01",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[concat('Microsoft.Network/networkSecurityGroups/', parameters('networkSecurityGroupName'))]",
                "[concat('Microsoft.Network/virtualNetworks/', parameters('virtualNetworkName'))]",
                "[concat('Microsoft.Network/publicIpAddresses/', parameters('publicIpAddressName'))]"
            ],
            "properties": {
                "ipConfigurations": [
                    {
                        "name": "ipconfig1",
                        "properties": {
                            "subnet": {
                                "id": "[variables('subnetRef')]"
                            },
                            "privateIPAllocationMethod": "Dynamic",
                            "publicIpAddress": {
                                "id": "[resourceId(resourceGroup().name, 'Microsoft.Network/publicIpAddresses', parameters('publicIpAddressName'))]"
                            }
                        }
                    }
                ],
                "networkSecurityGroup": {
                    "id": "[variables('nsgId')]"
                }
            }
        },
        {
            "name": "[parameters('networkSecurityGroupName')]",
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2019-02-01",
            "location": "[parameters('location')]",
            "properties": {
                "securityRules": "[parameters('networkSecurityGroupRules')]"
            }
        },
        {
            "name": "[parameters('virtualNetworkName')]",
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2019-09-01",
            "location": "[parameters('location')]",
            "properties": {
                "addressSpace": {
                    "addressPrefixes": "[parameters('addressPrefixes')]"
                },
                "subnets": "[parameters('subnets')]"
            }
        },
        {
            "name": "[parameters('publicIpAddressName')]",
            "type": "Microsoft.Network/publicIpAddresses",
            "apiVersion": "2019-02-01",
            "location": "[parameters('location')]",
            "properties": {
                "publicIpAllocationMethod": "[parameters('publicIpAddressType')]"
            },
            "sku": {
                "name": "[parameters('publicIpAddressSku')]"
            }
        },
        {
            "name": "[parameters('virtualMachineName')]",
            "type": "Microsoft.Compute/virtualMachines",
            "apiVersion": "2020-06-01",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[concat('Microsoft.Network/networkInterfaces/', parameters('networkInterfaceName'))]"
            ],
            "properties": {
                "hardwareProfile": {
                    "vmSize": "[parameters('virtualMachineSize')]"
                },
                "storageProfile": {
                    "osDisk": {
                        "createOption": "fromImage",
                        "managedDisk": {
                            "storageAccountType": "[parameters('osDiskType')]"
                        }
                    },
                    "imageReference": {
                        "publisher": "MicrosoftWindowsServer",
                        "offer": "WindowsServer",
                        "sku": "2019-Datacenter",
                        "version": "latest"
                    }
                },
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[resourceId('Microsoft.Network/networkInterfaces', parameters('networkInterfaceName'))]"
                        }
                    ]
                },
                "osProfile": {
                    "computerName": "[parameters('virtualMachineComputerName')]",
                    "adminUsername": "[parameters('adminUsername')]",
                    "adminPassword": "[parameters('adminPassword')]",
                    "windowsConfiguration": {
                        "enableAutomaticUpdates": true,
                        "provisionVmAgent": true,
                        "patchSettings": {
                            "patchMode": "[parameters('patchMode')]"
                        }
                    }
                },
                "diagnosticsProfile": {
                    "bootDiagnostics": {
                        "enabled": true
                    }
                }
            }
        }
    ],
    "outputs": {
        "adminUsername": {
            "type": "string",
            "value": "[parameters('adminUsername')]"
        }
    }
}
```
After deployment of a VM configured in the Azure Portal, you can download the template and parameters used. The values of the parameters are in the parameters file.  
The `dependsOn` list defines which resources need to be deployed before that resource can be deployed.

##### Creating and Running Containers in Azure
Azure Container Registry is a managed container registry service based on the open source Docker Registry, allowing you to build, store, and manage container images for your container‑based application deployments.

Azure Container Instances is an Azure service that allows us to run one or more containers in Azure with no servers or infrastructure to manage.

A running instance of a container image is called a container, and inside that container is your running application.

Generally speaking, you'll run one application inside of a container.

Container images are generally going to be very small and very portable. And a key way that container images become portable is via container registries, which allow for container images to be easily shared and used. 

A Dockerfile is a sequence of commands or instructions used to build a container image. In there you run commands to copy a compiled application binary and to craft the environment inside the container. You proceed executing the build command to create your binary container stored locally. You can run it locally to test it and you can push the container to Azure Container Registry, where it can be shared or combined, and be pulled to the Azure Container Instance to publish it.

Example Dockerfile
```dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1

RUN mkdir /app
WORKDIR /app

COPY ./webapp/bin/Release/netcoreapp3.1/publish ./
COPY ./config.sh ./

RUN bash config.sh

EXPOSE 80
ENTRYPOINT ["dotnet", "webapp.dll"]
```
`FROM` instruction: defines base container image  
`RUN` instruction: executes a command inside the container for a command that exists inside the container  
`WORKDIR` instruction: sets the working directory  
`COPY` instruction: copies application binary into the container image  
`EXPOSE` instructions: tells the container's runtime which port the application running in the container is listening on
`ENTRYPOINT` instruction: defines which script or binary we want to start when the container is started from this cointainer image

You build the container with `docker build --progress plain -t <name>:<tag> .`.

Demo: create local container
```sh
#####################################################################################
# Requirements for this demo
# 1. Install docker          - https://docs.docker.com/desktop/#download-and-install 
# 2. Install dotnet core sdk - https://dotnet.microsoft.com/download/dotnet-core
#####################################################################################
cd ./m2/demos

#This is our simple hello world web app and will be included in the course downloads.
ls -ls ./webapp

#Step 1 - Build our web app first and test it prior to putting it in a container
dotnet build ./webapp
dotnet run --project ./webapp #Open a new terminal to test.
curl http://localhost:5000

#Step 2 - Let's publish a local build...this is what will be copied into the container
dotnet publish -c Release ./webapp

#Step 3 - Time to build the container and tag it...the build is defined in the Dockerfile
docker build -t webappimage:v1 .

#In docker 3.0, by default output from commands run in the container during the build aren't written to console
#Add you need to add --progress plain to see the output from commands running during the build
#If you already built the image you'll need to delete the image and your build cache so new layers are built
#docker rmi webappimage:v1 && docker builder prune --force && docker image prune --force
#docker build --progress plain -t webappimage:v1 .

#Step 4 - Run the container locally and test it out
docker run --name webapp --publish 8080:80 --detach webappimage:v1
curl http://localhost:8080

#Delete the running webapp container
docker stop webapp
docker rm webapp

#The image is still here...let's hold onto this for the next demo.
docker image ls webappimage:v1
```

Azure Container Registry is a managed Docker Registry service based on the open source Docker Registry, allowing you to build, store, and manage container images for container deployments. Container orchestrators, such as Kubernetes, and serverless platforms, like Azure Container Instances, can be configured to pull images from Azure Container Registry. Further, you can use Azure Container Registry tasks to streamline building, testing, pushing, and deploying images in Azure.

Azure Container Registry supports two types of identities for logging in, Azure Active Directory Identities, including both user and service principals, and a service‑specific ACR admin account.

Orchestrators, toosl and applications should use 'headless' authentication and is only supported using Azure Active Directory Identities (service principals for applications). Don't use ACR Admin for this.

You can log into ACR from the command line with `az acr login` or `docker login`. Role based access controls define what you are allowed to configure. The roles can be divided into 2 gropus:
- Users: Owner, contributor, reader. These roles also have acces to the resource manager.
- Service principals: ArcPush, ArcPull, ArcDelete. These roles are for tools, pipelines and orchestrators.

Demo: create and push to ACR
```sh
#Login interactively and set a subscription to be the current active subscription
az login
az account set --subscription "Demonstration Account"


#Step 0 - Create Resource Group for our demo if it doesn't exist 
az group create \
    --name psdemo-rg \
    --location centralus


#Step 1 - Create an Azure Container Registry
#SKUs include Basic, Standard and Premium (speed, replication, adv security features)
#https://docs.microsoft.com/en-us/azure/container-registry/container-registry-skus#sku-features-and-limits
ACR_NAME='psdemoacr'  #<---- THIS NEEDS TO BE GLOBALLY unique inside of Azure
az acr create \
    --resource-group psdemo-rg \
    --name $ACR_NAME \
    --sku Standard 

#Step 2 - Log into ACR to push containers...this will use our current azure cli login context
az acr login --name $ACR_NAME

#Step 3 - Get the loginServer which is used in the image tag
ACR_LOGINSERVER=$(az acr show --name $ACR_NAME --query loginServer --output tsv)
echo $ACR_LOGINSERVER

#Step 4 - Tag the container image using the login server name. This doesn't push it to ACR, that's the next step.
#[loginUrl]/[repository:][tag]
docker tag webappimage:v1 $ACR_LOGINSERVER/webappimage:v1
docker image ls $ACR_LOGINSERVER/webappimage:v1
docker image ls

#Step 5 - Push image to Azure Container Registry
docker push $ACR_LOGINSERVER/webappimage:v1

#Step 6 - Get a listing of the repositories and images/tags in our Azure Container Registry
az acr repository list --name $ACR_NAME --output table
az acr repository show-tags --name $ACR_NAME --repository webappimage --output table

####
#We don't have to build locally then push, we can build in ACR with Tasks.
####
#Step 1 - use ACR build to build our image in azure and then push that into ACR
az acr build --image "webappimage:v1-acr-task" --registry $ACR_NAME .

#Both images are in there now, the one we built locally and the one build with ACR tasks
az acr repository show-tags --name $ACR_NAME --repository webappimage --output table
```

Azure Container Instances gives you a serverless Platform as a Service offering, enabling you to run containers in Azure without having to set up any virtual machines or other infrastructure services. It's most effectively used for simple applications, task automation, and also build jobs.

When you deploy containers in Azure Container Instances, you can access your applications over the internet via an automatically provisioned, fully qualified domain name. Or optionally, containers deployed in ACI can be connected directly to an Azure Virtual Network for private, secure communications.

When defining your containers in ACI, you can specify an amount of CPU or memory needed to run your application.

If your ACI‑deployed containers need persistent data for data files or even databases, containers can directly mount Azure file shares backed by Azure Storage. Azure Container Instances also supports scheduling of multi‑group containers that share a host machine, local network, storage, and lifecycle.

Container restart policy tells ACI what to do if an application running inside of the container stops. And your options are restart always, restart on failure, and restart never.

We can deploy from ACR, but also from Docker Hub and other container registries. We can use both public and private registries. Private registries require authentication or are not directly attached to the internet. ACR uses authentication by default.

Demo: Running in ACI
```sh
#Login interactively and set a subscription to be the current active subscription
az login
az account set --subscription "Demonstration Account"

#Demo 0 - Deploy a container from a public registry.
az container create \
    --resource-group psdemo-rg \
    --name psdemo-hello-world-cli \
    --dns-name-label psdemo-hello-world-cli \
    --image mcr.microsoft.com/azuredocs/aci-helloworld \
    --ports 80

#Show the container info
az container show --resource-group 'psdemo-rg' --name 'psdemo-hello-world-cli' 

#Retrieve the URL, the format is [name].[region].azurecontainer.io
URL=$(az container show --resource-group 'psdemo-rg' --name 'psdemo-hello-world-cli' --query ipAddress.fqdn | tr -d '"') 
echo "http://$URL"

#Demo 1 - Deploy a container from Azure Container Registry with authentication
#Step 0 - Set some environment variables and create Resource Group for our demo
ACR_NAME='psdemoacr' #<---change this to match your globally unique ACR Name

#Step 1 - Obtain the full registry ID and login server which well use in the security and create sections of the demo
ACR_REGISTRY_ID=$(az acr show --name $ACR_NAME --query id --output tsv)
ACR_LOGINSERVER=$(az acr show --name $ACR_NAME --query loginServer --output tsv)

echo "ACR ID: $ACR_REGISTRY_ID"
echo "ACR Login Server: $ACR_LOGINSERVER"

#Step 2 - Create a service principal and get the password and ID, this will allow Azure Container Instances to Pull Images from our Azure Container Registry
SP_NAME=acr-service-principal
SP_PASSWD=$(az ad sp create-for-rbac \
    --name http://$ACR_NAME-pull \
    --scopes $ACR_REGISTRY_ID \
    --role acrpull \
    --query password \
    --output tsv)

SP_APPID=$(az ad sp show \
    --id http://$ACR_NAME-pull \
    --query appId \
    --output tsv)

echo "Service principal ID: $SP_APPID"
echo "Service principal password: $SP_PASSWD"

#Step 3 - Create the container in ACI, this will pull our image named
#$ACR_LOGINSERVER is psdemoacr.azurecontainer.io. this should match *your* login server name
az container create \
    --resource-group psdemo-rg \
    --name psdemo-webapp-cli \
    --dns-name-label psdemo-webapp-cli \
    --ports 80 \
    --image $ACR_LOGINSERVER/webappimage:v1 \
    --registry-login-server $ACR_LOGINSERVER \
    --registry-username $SP_APPID \
    --registry-password $SP_PASSWD 

#Step 4 - Confirm the container is running and test access to the web application, look in instanceView.state
az container show --resource-group psdemo-rg --name psdemo-webapp-cli  

#Get the URL of the container running in ACI...
#This is our hello world app we build in the previous demo
URL=$(az container show --resource-group psdemo-rg --name psdemo-webapp-cli --query ipAddress.fqdn | tr -d '"') 
echo $URL
curl $URL

#Step 5 - Pull the logs from the container
az container logs --resource-group psdemo-rg --name psdemo-webapp-cli

#Step 6 - Delete the running container
az container delete  \
    --resource-group psdemo-rg \
    --name psdemo-webapp-cli \
    --yes

#Step 7 - Clean up from our demos, this will delete all of the ACIs and the ACR deployed in this resource group.
#Delete the local container images
az group delete --name psdemo-rg --yes
docker image rm psdemoacr.azurecr.io/webappimage:v1
docker image rm webappimage:v1
```

#### Microsoft Learn

##### Introduction to Azure virtual machines

The main difference between Iaas and Paas is that with IaaS you have more control over the OS configuration.

With Iaas you still need to maintain the virtual machines:
- Operating system
- Patches
- Performance
- Disk space usage

Each VM is composed out of different components, which are software objects (as opposed to hardware objects in your own data center).

All VMs have at least 2 disks:
- Disk for OS
- Temporary Disk. This disk loses all data as soon as the VM is shut down, but you can add permanent data to your VMs with additional virtual disks, or managed disks, which are higher performing data disks.

Resource group is a group of resources that are needed together from initiation to termination.

Checklist of things to think about:
- Start with the network
By default, services outside of the VN cannot connect to services within the virtual network. Network addresses and subnets are not trivial to change once you have them set up, and if you plan to connect your private company network to the Azure services, you will want to make sure you consider the topology before putting any VMs into place. NOTE: Azure reserves the first four addresses and the last address in each subnet for its use.
- Name the VM
You can specify a name of up to 15 characters on a Windows VM and 64 characters on a Linux VM. A good convention is to include the following information in the name: Environment, Location, Instance, Product or Service, Role. For example, `devusc-webvm01`. 
- Decide the location for the VM
This lets you place your VMs as close as possible to your users to improve performance and to meet any legal, compliance, or tax requirements. Two things to consider. First, the location can limit your available options. First, the location can limit your available options
- Determine the size of the VM
Workload options are classified as follows on Azure:
| Option | Description |  
| ------ | ----------- |  
| General purpose | General-purpose VMs are designed to have a balanced CPU-to-memory ratio. Ideal for testing and development, small to medium databases, and low to medium traffic web servers. |  
| Compute optimized | Compute optimized VMs are designed to have a high CPU-to-memory ratio. Suitable for medium traffic web servers, network appliances, batch processes, and application servers. |  
| Memory optimized | Memory optimized VMs are designed to have a high memory-to-CPU ratio. Great for relational database servers, medium to large caches, and in-memory analytics. |  
| Storage optimized | Storage optimized VMs are designed to have high disk throughput and IO. Ideal for VMs running databases. |  
| GPU | GPU VMs are specialized virtual machines targeted for heavy graphics rendering and video editing. These VMs are ideal options for model training and inferencing with deep learning. |  
| High performance computes | High performance compute is the fastest and most powerful CPU virtual machines with optional high-throughput network interfaces. |  
Changing a running VM size will automatically reboot the machine to complete the request. Be careful about resizing production VMs - they will be rebooted automatically which can cause a temporary outage and change some configuration settings such as the IP address.
- Understanding the pricing model
Compute costs: Compute expenses are priced on a per-hour basis but billed on a per-minute basis. Storage costs: You are charged separately for the storage the VM uses. You're able to choose from two payment options for compute costs.
| Option | Description |
| -----  | ----------- |
| Pay as you go | With the pay-as-you-go option, you pay for compute capacity by the second, with no long-term commitment or upfront payments. |
| Reserved Virtual Machine Instances | The Reserved Virtual Machine Instances (RI) option is an advance purchase of a virtual machine for one or three years in a specified region. |
- Storage for the VM
Best practice is that all Azure virtual machines will have at least two virtual hard disks (VHDs). The first disk stores the operating system, and the second is used as temporary storage. Separating out the data to different VHDs allows you to manage the security, reliability, and performance of the disk independently. The data for each VHD is held in Azure Storage as page blobs, which allows Azure to allocate space only for the storage you use. You pay for the storage you are consuming.
A storage account provides access to objects in Azure Storage for a specific subscription. VMs always have one or more storage accounts to hold each attached virtual disk.
When you create disks, you will have two options for managing the relationship between the storage account and each VHD.
| Option | Description |  
| ------ | ----------- |
| Unmanaged disks | With unmanaged disks, you are responsible for the storage accounts that are used to hold the VHDs that correspond to your VM disks. You pay the storage account rates for the amount of space you use. A single storage account has a fixed-rate limit of 20,000 I/O operations/sec. This means that a storage account is capable of supporting 40 standard virtual hard disks at full utilization. If you need to scale out with more disks, then you'll need more storage accounts, which can get complicated. |
| Managed disks | Managed disks are the newer and recommended disk storage model. They elegantly solve this complexity by putting the burden of managing the storage accounts onto Azure. You specify the size of the disk, up to 4 TB, and Azure creates and manages both the disk and the storage. You don't have to worry about storage account limits, which makes managed disks easier to scale out. |
- Select an operating system
Azure provides a variety of OS images that you can install into the VM, including several versions of Windows and flavors of Linux. If you are looking for more than just base OS images, you can search the Azure Marketplace for more sophisticated install images that include the OS and popular software tools installed for specific scenarios. Finally, if you can't find a suitable OS image, you can create your disk image with what you need, upload it to Azure storage, and use it to create an Azure VM. Keep in mind that Azure only supports 64-bit operating systems.

Let's look at some other ways to create and administer resources in Azure:
- Azure Resource Manager
Azure Resource Manager organizes resources into named resource groups that let you deploy, update, or delete all of the resources together. Resource Manager also enables you to create templates, which can be used to create and deploy specific configurations. Resource Manager templates are JSON files that define the resources you need to deploy for your solution.
- Azure PowerShell
PowerShell is a cross-platform shell that provides services like the shell window and command parsing. Azure PowerShell is an optional add-on package that adds the Azure-specific commands (referred to as cmdlets).
- Azure CLI
It's available for Windows, Linux and macOS, or in the browser using the Cloud Shell. Like Azure PowerShell, the Azure CLI is a powerful way to streamline your administrative workflow. Unlike Azure PowerShell, the Azure CLI does not need PowerShell to function.

Generally speaking, both Azure PowerShell and Azure CLI are good options if you have simple scripts to run and want to stick to command-line tools. When it comes to more complex scenarios, where the creation and management of VMs form part of a larger application with complex logic, another approach is needed.
- Azure REST API
Operations are exposed as URIs with corresponding HTTP methods (`GET`, `PUT`, `POST`, `DELETE`, and `PATCH`) and a corresponding response.
- Azure Client SDK
The Azure Client SDK encapsulates the Azure REST API, making it much easier for developers to interact with Azure. Here's an example snippet of C# code to create an Azure VM using the `Microsoft.Azure.Management.Fluent` NuGet package.
```csharp
var azure = Azure
    .Configure()
    .WithLogLevel(HttpLoggingDelegatingHandler.Level.Basic)
    .Authenticate(credentials)
    .WithDefaultSubscription();
// ...
var vmName = "test-wp1-eus-vm";

azure.VirtualMachines.Define(vmName)
    .WithRegion(Region.USEast)
    .WithExistingResourceGroup("TestResourceGroup")
    .WithExistingPrimaryNetworkInterface(networkInterface)
    .WithLatestWindowsImage("MicrosoftWindowsServer", "WindowsServer", "2012-R2-Datacenter")
    .WithAdminUsername("jonc")
    .WithAdminPassword("aReallyGoodPasswordHere")
    .WithComputerName(vmName)
    .WithSize(VirtualMachineSizeTypes.StandardDS1)
    .Create();
```
- Azure VM Extensions
Azure VM extensions are small applications that enable you to configure and automate tasks on Azure VMs after initial deployment. Azure VM extensions can be run with the Azure CLI, PowerShell, Azure Resource Manager templates, and the Azure portal.
- Azure Automation Services
Azure Automation enables you to integrate services that allow you to automate frequent, time-consuming, and error-prone management tasks with ease. These services include:
| Type | Description |  
| ---- | ----------- |
| Process Automation | Process automation enables you to set up watcher tasks that can respond to events that may occur in your datacenter. |
| Configuration Management | Configuration management enables you to track these updates, and take action as required |
| Update Management | This is used to manage updates and patches for your VMs. |

Managing availability of you Azure VMs
Availability is the percentage of time a service is available for use. Microsoft does not automatically update your VMs OS or software. You have complete control and responsibility for that. However, the underlying software host and hardware are periodically patched to ensure reliability and high performance at all times. To ensure your services aren't interrupted and avoid a single point of failure, it's recommended to deploy at least two instances of each VM. This feature is called an availability set. When you place VMs into an availability set, Azure guarantees to spread them across Fault Domains and Update Domains. A fault domain is a logical group of hardware in Azure that shares a common set of hardware components, and that share a single point of failure. You can think of it as a rack within an on-premises datacenter. An update domain is a logical group of hardware that can undergo maintenance, or be rebooted at the same time.

Azure Site Recovery replicates workloads from a primary site to a secondary location.
- Site Recovery enables the use of Azure as a destination for recovery, thus eliminating the cost and complexity of maintaining a secondary physical datacenter.
- Site Recovery makes it incredibly simple to test failovers for recovery drills without impacting production environments. This makes it easy to test your planned or unplanned failovers. After all, you don’t have a good disaster recovery plan if you’ve never tried to failover.

Azure Site Recovery works with Azure resources, or Hyper-V, VMware, and physical servers in your on-premises infrastructure and can be a key part of your organization’s business continuity and disaster recovery (BCDR) strategy by orchestrating the replication, failover, and recovery of workloads and applications if the primary location fails.

Azure Backup is a backup as a service offering that protects physical or virtual machines no matter where they reside: on-premises or in the cloud. Azure Backup was designed to work in tandem with other Azure services and provides several distinct benefits.
- Automatic storage management
- Unlimited scaling
- Multiple storage options
- Unlimited data transfer
- Data encryption
- Application-consistent backup
- Long-term retention

Azure Backup uses a Recovery Services vault for storing the backup data. A vault is backed by Azure Storage blobs, making it a very efficient and economical long-term storage medium. With the vault in place, you can select the machines to back up, and define a backup policy (when snapshots are taken and for how long they’re stored).

##### Create a Linux VM in Azure

Selecting an image is one of the first and most important decisions you'll make when creating a VM. An image is a template that's used to create a VM. These templates include an OS and often other software, such as development tools or web hosting environments.

The temporary disk (VHD) on Linux virtual machines is `/dev/sdb` and is formatted and mounted to `/mnt` by the Azure Linux Agent. It is sized based on the VM size and is used to store the swap file.

An interesting capability is to create a VHD image from a real disk. This allows you to easily migrate existing information from an on-premises computer to the cloud.

Although SSH provides an encrypted connection, using passwords with SSH connections leaves the VM vulnerable to brute-force attacks of passwords. A more secure and preferred method of connecting to a Linux VM with SSH is a public-private key pair, also known as SSH keys.

Azure Cloud Shell stores the generated keys in Azure in your private storage account.

When you add a passphrase to your SSH key, it encrypts the private key using 128-bit AES so that the private key is useless without the passphrase to decrypt it. To apply the SSH key while creating a new Linux VM, you will need to copy the contents of the public key and supply it to the Azure portal, or supply the public key file to the Azure CLI or Azure PowerShell command.

After connecting to the VM with SSH, you notice (with command `lsblk`) that our Data drive (sdc) is present but not mounted into the file system. Azure added a VHD but didn't initialize it.
To initialize:
1. First, identify the disk that we just created. You could also use `dmesg | grep SCSI`, which will list all the messages from the kernel for SCSI devices.
2. After you know the drive (`sdc`) you need to initialize, you can use `fdisk` to do that. You'll need to run the command with `sudo`, and supply the disk you want to partition. We can use the following command to create a new primary partition. `(echo n; echo p; echo 1; echo ; echo ; echo w) | sudo fdisk /dev/sdc`.
3. Next, we need to write a file system to the partition with the mkfs command. `sudo mkfs -t ext4 /dev/sdc1`
4. Finally, we need to mount the drive to the file system. Let's assume we will have a `data` folder. Let's create the mount point folder and mount the drive. `sudo mkdir /data && sudo mount /dev/sdc1 /data`

We initialized the disk, and mounted it. If you want more details on this process, see the Add and size disks in Azure virtual machines module. This task is covered in more detail there.

Our virtual network is blocking the inbound request. We can change that through configuration.

By default, new VMs are locked down. Apps can make outgoing requests, but the only inbound traffic allowed is from the virtual network (for example, other resources on the same local network) and from Azure Load Balancer (probe checks).

Security groups can be associated to a network interface (for per host rules), a subnet in the virtual network (to apply to multiple resources), or both levels. NSGs use rules to allow or deny traffic moving through the network. Each rule identifies the source and destination address (or range), protocol, port (or range), direction (inbound or outbound), a numeric priority, and whether to allow or deny the traffic that matches the rule

For inbound traffic, Azure processes the security group associated to the subnet, and then the security group applied to the network interface. Outbound traffic is handled in the opposite order (the network interface first, followed by the subnet). Keep in mind that security groups are optional at both levels. If no security group is applied, then all traffic is allowed by Azure. If the VM has a public IP, this could be a serious risk, particularly if the OS doesn't provide a built-in firewall. The rules are evaluated in priority order, starting with the lowest priority rule. Deny rules always stop the evaluation. The last rule is always a Deny All rule. This is a default rule added to every security group for both inbound and outbound traffic with a priority of 65500. That means to have traffic pass through the security group, you must have an allow rule, or the final default rule will block it.

Always make sure to lock down ports used for administrative access. An even better approach is to create a VPN to link the virtual network to your private network, and only allow RDP or SSH requests from that address range. You can also change the port used by SSH to be something other than the default. Keep in mind that changing ports is not sufficient to stop attacks. It simply makes it a little harder to discover.

##### Create a Windows VM in Azure

Now that we have a new Windows virtual machine, we need to install our custom software onto it. There are several options to choose from:
- Remote Desktop Protocol (RDP)
- Custom scripts
- Custom VM images (with the software preinstalled)

When you connect, you'll typically receive two warnings. These are:
- Publisher warning - caused by the .rdp file not being publicly signed.
- Certificate warning - caused by the machine certificate not being trusted.

In test environments, these warnings can be ignored. In production environments, the .rdp file can be signed using RDPSIGN.EXE and the machine certificate placed in the client's Trusted Root Certification Authorities store.

The first time you connect to a Windows server VM, it will launch Server Manager. This allows you to assign a worker role for common web or data tasks. You can also launch the Server Manager through the Start Menu.

This is where we would add the Web Server role to the server. This will install IIS and as part of the configuration you would turn off HTTP requests and enable the FTP server. Or, we could ignore IIS and install a third-party FTP server. We'd then configure the FTP server to allow access to a folder on our big data drive we added to the VM.

Any additional drives you create from scratch will need to be initialized and formatted. The process for doing this is identical to a physical drive.
- Launch the Disk Management tool from the Start Menu. Y
- It will display a warning that it has detected an uninitialized disk.
- Click OK to initialize the disk. It will then show up in the list of volumes where you can format it and assign a drive letter.
- Open File Explorer and you should now see your data drive.

If we always need to install some software, you might consider automating the process using scripting.

##### Manage resources in Azure

The term private cloud should not be considered a rebranding of traditional on-premises datacenters. A private cloud uses on-premises infrastructure and services to provide similar benefits of the public cloud. It uses an abstraction platform to provide cloud-like services such as Kubernetes clusters or a complete cloud environment like Azure Stack.

Infrastructure as a service (IaaS) is an instant computing infrastructure, provisioned and managed over the Internet.

Like IaaS, PaaS includes infrastructure such as servers, storage, and networking. In addition, it also includes middleware, development tools, and other services. PaaS supports the complete web application lifecycle: building, testing, deploying, managing, and updating. PaaS removes the need to manage software licenses, middleware, and infrastructure of the services. Your developers deploy the website files to the cloud and your website is available on the Internet.

Software as a service (SaaS) allows users to connect to and use cloud-based apps over the Internet. Users can run most SaaS apps directly from their web browser without needing to download and install any software.

##### Control Azure services with the CLI

Use `az find`, the AI robot that uses the Azure documentation to tell you more about commands, the CLI and more.
- `az find blob`
- `az find "az vm"`

To login: `az login`

The Azure CLI group create command creates a resource group: `az group create --name <name> --location <location>`, then `az group list --output table` to display the resource groups.

Run az appservice plan create --help to see the pricing tiers.

Deploy code from a GitHub repository to the web app: `az webapp deployment source config --name $AZURE_WEB_APP --resource-group $RESOURCE_GROUP --repo-url "https://github.com/Azure-Samples/php-docs-hello-world" --branch master --manual-integration`

##### Automate Azure tasks using scripts with PowerShell

Azure provides three administration tools to choose from:
- The Azure portal
- The Azure CLI
- Azure PowerShell

Azure PowerShell is a module that you add to Windows PowerShell or PowerShell Core to let you connect to your Azure subscription and manage resources. Azure PowerShell requires PowerShell to function. PowerShell provides services like the shell window, command parsing, and so on. Azure PowerShell adds the Azure-specific commands.

PowerShell lets you write commands and execute them immediately. This is known as interactive mode.

A PowerShell command is called a cmdlet (pronounced "command-let"). A cmdlet is a command that manipulates a single feature. The term cmdlet is intended to imply "small command". By convention, cmdlet authors are encouraged to keep cmdlets simple and single-purpose.

Cmdlets follow a verb-noun naming convention; for example, Get-Process, Format-Table, and Start-Service. There is also a convention for verb choice: "get" to retrieve data, "set" to insert or update data, "format" to format data, "out" to direct output to a destination, and so on.

Cmdlets are shipped in modules. A PowerShell Module is a DLL that includes the code to process each available cmdlet. You load cmdlets into PowerShell by loading the module they are contained in. You can get a list of loaded modules using the `Get-Module` command.

`Az` is the formal name for the Azure PowerShell module containing cmdlets to work with Azure features. It contains hundreds of cmdlets that let you control nearly every aspect of every Azure resource. Install Azure Powershell: `Install-Module Az -AllowClobber`. You can also use the `Update-Module` command to re-install a module if you are having trouble with it.

> Example: How to create a resource group with Azure PowerShell
1. Import the Azure cmdlets. `Import-Module Az`
2. Connect to your Azure subscription. 
    - `Connect-AzAccount`
    - `Get-AzSubscription`
    - `Select-AzSubscription -SubscriptionId '53dde41e-916f-49f8-8108-558036f826ae'`
3. Create the resource group. 
    - `Get-AzResourceGroup`
    - `Get-AzResourceGroup | Format-Table`
    - `New-AzResourceGroup -Name <name> -Location <location>`
4. Verify that creation was successful (see the following).
    - `Get-AzResource`
    - `Get-AzResource | ft`
    - `Get-AzResource -ResourceGroupName ExerciseResources`

Create Azure VM:  
```powershell
New-AzVm 
       -ResourceGroupName <resource group name> 
       -Name <machine name> 
       -Credential <credentials object> 
       -Location <location> 
       -Image <image name>
```
Credential: An object containing the username and password for the VM admin account. We will use the Get-Credential cmdlet. This cmdlet will prompt for a username and password and package it into a credential object.

> Getting information for a VM
- `$vm = Get-AzVM  -Name MyVM -ResourceGroupName ExerciseResources`
```powershell
$ResourceGroupName = "ExerciseResources"
$vm = Get-AzVM  -Name MyVM -ResourceGroupName $ResourceGroupName
$vm.HardwareProfile.vmSize = "Standard_DS3_v2"

Update-AzVM -ResourceGroupName $ResourceGroupName  -VM $vm
```

The `Remove-AzVM` command just deletes the VM. To delete all resources:
- `$vm | Remove-AzNetworkInterface –Force`
- `Get-AzDisk -ResourceGroupName $vm.ResourceGroupName -DiskName $vm.StorageProfile.OSDisk.Name | Remove-AzDisk -Force`
- `Get-AzVirtualNetwork -ResourceGroup $vm.ResourceGroupName | Remove-AzVirtualNetwork -Force`
- `Get-AzNetworkSecurityGroup -ResourceGroup $vm.ResourceGroupName | Remove-AzNetworkSecurityGroup -Force`
- `Get-AzPublicIpAddress -ResourceGroup $vm.ResourceGroupName | Remove-AzPublicIpAddress -Force`

PowerShell has many features found in typical programming languages. You can define variables, use branches and loops, capture command-line parameters, write functions, add comments, and so on. We will need three features for our script: variables, loops, and parameters. Variables can hold objects.

Loop example
```powershell
For ($i = 1; $i -lt 3; $i++)
{
    $i
}
```

Parameters:  
`.\setupEnvironment.ps1 -size 5 -location "East US"`. Inside the script you capture the values into variables: `param([string]$location, [int]$size)`

Example script to deploy 3 vms:
```powershell
param([string]$resourceGroup)

$adminCredential = Get-Credential -Message "Enter a username and password for the VM administrator."

For ($i = 1; $i -le 3; $i++)
{
    $vmName = "ConferenceDemo" + $i
    Write-Host "Creating VM: " $vmName
    New-AzVm -ResourceGroupName $resourceGroup -Name $vmName -Credential $adminCredential -Image UbuntuLTS
}
```
Execute the script: `./ConferenceDailyReset.ps1 learn-46df0bb4-a295-48ed-a766-316b5c3a81d9`
Verify: `Get-AzResource -ResourceType Microsoft.Compute/virtualMachines`

### 1.2 Create Azure App Service Web Apps {#1.2}

#### Pluralsight notes

##### Creating Azure App Service Web Apps
Azure App Service is an HTTP‑based service for hosting web apps, and it's just basically a very advanced version of web hosting. A lot of different frameworks can be used. Since it is a managed service, Microsoft does pretty much all the heavy lifting: security, load balancing, automation.

There are two types of App Service plans:
- Non-isolated: the App Service plan is not inside a virtual network. Free-tier (F1) and shared plan (D1). With shared plans, under the hood other apps can run on the same VM as your app. Basic tier (B1, B2, B3): no advanced network requirements, no auto-scaling, etc. Robust, but not production level. Standard (S1, S2, S3): allow autoscaling. Premium V2 and V3: higher speeds. 
- Isolated: isolation with app service environments (ASE). The ASE can be deployed into an Azure VN. ASE is fully isolated and has a dedicated environment for running web apps. It is highly scalable and has high memory utilazation. It allows for secure network access through network security groups for fine-grained control over network traffic. Apps can connect over VPN to on-premises resources.

Choose for Isolated App Service plan if you need isolation at the network level.

Demo: create app service via Azure CLI
```sh
az group create -n webapps-dev-rg -l westus2

az appservice plan create --name webapps-dev-plan \
  --resource-group webapps-dev-rg \
  --sku s1 \
  --is-linux
  
az webapp create -g webapps-dev-rg \
  -p webapps-dev-plan \
  -n mp10344884 \
  --runtime "node|10.14"
```

Best way to automate deployment is through ARM templates

##### Configuring Azure App Service
Securing a Web App with SSL.  
Web Apps are published under the domain <web app name>.azurewebsites.net which provides secure https connection availability. We can also use custom domains. So, for example, we could take the pluralsight.com domain if we wanted to, and we could make that resolve over to a web application in App Service. And we could even map an SSL certificate to that web application, and that would ensure that external users going to that domain would get a secure connection, and they wouldn't get any warnings in a browser window because we could upload a certificate that has the pluralsight.com domain as one of the subject names. Considerations when you want to secure a domain with SSL/TLS binding.
- Use Basic, Standard, Premium, or Isolated plan types
- Public (bought) vs. private (not all clients accept these) certificates
- Managed (bought in the Azure portal) vs. un-managed (self-signed and commercially bought)certificates
- You can enforce HTTPS and TLS versions

Configure a custom domain  
Add the Domain Verification ID for this web app to the domain as a TXT record. re, the type of resource record is going to be a TXT record. This is going to be one of the records that gets queried by app service to prove that we actually own the domain.

We can go into the App Service web application configuration and set up a connection string, which we can call it whatever we want. That connection string is just an environment variable that's available in the compute environment there in App Service. So your application code could query for an environment variable called SQLServerDB, and that'll resolve to the actual connection string to the SQL Database.

You can copy the connection string of the Azure database in the connection string section, in configuration, under settings, of the app service web application.

Enabling Diagnostic Logging of the App Service under.  
- Application logging: set the level of app logs that you want to store.
- Web server logging
- Detailed error messages: by default this is turned off
- Failed request tracing
- Deployment logging

Deploying code to App Service Web Apps.
- App service pulls code from GitHub, Bitbucket, and Azure Repos. You can do continuous deployment through code updates, or create CI/CD pipelines with, for example, ADO.
- Authorize Azure App Service to retrieve the code
- Enable/disable continuous deployment

##### Scaling Azure App Service
Scaling up vertically: make the VM bigger.
Scaling out/in horizantally: use an Azure load balancer for this. You can use trigger an auto-scale event that adds a VM if, for example, the average CPU usage is > 70% for 5 minutes straight. You can also scale based on other rules, such as a time-schedule. You can also manually scale through the Azure Portal.

Auto scale is available for standard, premium and isolated pricing tiers. It is managed through Azure Monitor. The autoscale profile consists of:
- Capacity settings (minimum, maximum, default number of VMs)
- Rules
- Notifications

We can use Azure Monitor Metrics to configure scaling: metrics (resources usage/custom) and time (schedule). Based on the rules, you can set up different actions, such as VM scaling, email, webhook.

#### Microsoft Learn

##### Host a web application with Azure App Service
Azure App Service is PaaS

Using the Azure portal, you can easily add deployment slots to an App Service web app. For instance, you can create a staging deployment slot where you can push your code to test on Azure.

With Azure DevOps, you can define your own build and release process that compiles your source code, runs the tests, builds a release, and finally deploys the release into your web app every time you commit the code.

App Service also supports FTP-based publishing for more traditional workflows.

You can deploy your application to App Service as code or as a ready-to-run Docker image. Selecting Docker image will activate the Docker tab of the wizard, where you provide information about the Docker registry from which App Service will retrieve your image. If you choose to deploy your application as code, App Service needs to know what runtime your application uses (examples include Node.js, Python, Java, and .NET).

Selecting the Windows operating system activates the Monitoring tab, where you have the option to enable Application Insights.

An App Service plan is a set of virtual server resources that run App Service apps. A plan's size (sometimes referred to as its sku or pricing tier) determines the performance characteristics of the virtual servers that run the apps assigned to the plan and the App Service features that those apps have access to. Every App Service web app you create must be assigned to a single App Service plan that runs it.

App Service plans are the unit of billing for App Service. The size of each App Service plan in your subscription, in addition to the bandwidth resources used by the apps deployed to those plans, determines the price that you pay. The number of web apps deployed to your App Service plans has no effect on your bill.

Automated deployment, or continuous integration, is a process used to push out new features and bug fixes in a fast and repetitive pattern with minimal impact on end users. There are a few options that you can use to manually push your code to Azure:
- Git
- `az webapp up`
- `az webapp deployment source config-zip` to send a ZIP of the application files.
- WAR deploy: App Service deployment mechanism specifically designed for deploying Java web applications using WAR packages.
- Visual Studio
- FTP/S

##### Deploy a website to Azure with Azure App Service
A Visual Study workload is a pre-configured bundle of tools within Visual Studio that are grouped to enable developers to build certain types of applications, use certain development languages, or develop for specific platforms.

The Azure development workload in Visual Studio installs the latest Azure SDK for .NET and tools for Visual Studio. Once these items are installed, you can view resources in Cloud Explorer, create resources using Azure Resource Manager tools, build applications for Azure web and Cloud Services, and perform big data operations using Azure Data Lake tools.

Azure App Service is a service for hosting web applications, REST APIs, and backend services. App Service supports code written in .NET Core, .NET Framework, Java, Ruby, Node.js, PHP, and Python. App Service is ideal for most websites, particularly if you don't need tight control over the hosting infrastructure.

You can define an App Service plan in advance in the Azure portal with PowerShell or the Azure CLI, or set one up as you publish your application in Visual Studio. Each App Service plan defines:
- Region (West US, East US, and so on)
- Number of VM instances
- Size of VM instances (small, medium, large)
- Pricing tier (Free, Shared, Basic, Standard, Premium, Premium V2, Isolated)

Visual Studio will remember where the app is published, which makes updating and changing your app a two-click process.

One slot is the production slot. This slot is the web app that users see when they connect. Make sure that the app deployed to this slot is stable and well tested.


##### Stage a web app deployment for testing and rollback by using App Service deployment slots
Use additional slots to host new versions of your web app. Against these instances, you can run tests such as integration tests, acceptance tests, and capacity tests. Unlike a code deployment, a slot swap is instantaneous. When you swap slots, the slot hostnames are exchanged, immediately sending production traffic to the new version of the app. When you use slot swaps to deploy, your app is never exposed to the public web in a partially deployed state. If you find that, in spite of your careful testing, the new version has a problem, you can roll back the version by swapping the slots back.

The slots are treated as separate instances, and resources, of that web app. However, each slot shares the resources of the App Service plan, including virtual machine memory and CPU as well as disk space.

Deployment slots are available only when your web app uses an App Service plan in the Standard, Premium, or Isolated tier.

Many of the technologies that developers use to create web apps require final compilation and other actions on the server before they deliver a page to a user. The initial delay is called a cold start. You can avoid a cold start by using slot swaps to deploy to production. When you swap a slot into production, you "warm-up" the app because your action sends a request to the root of the site.

Unless you register the slot with a search engine or link to it from a crawled page, the slot won't appear in search engine indexes. You can control access to a slot by using IP address restrictions.

Example using deployment slots
1. Create web app from portal
2. Select Web App resource
3. Go to deployment center and choose code source, save
4. Configure the git client
5. Use the git clone url from the Overview Essentials pane and locally push the code to remote with `git remote add production <git-clone-url>` and `git push production`.
6. Go to the App Service Web App, go to deployment slots and add a slot
7. Go to resources, select the new slot / resource and repeat step 3 and 5
8. Modify your project, then `git add .`, `git commit -m "New version of web app."`, `git push staging`.

Suppose, for example, you have two databases. You use one for production, and the other for acceptance testing. You always want the app version in the staging slot to use the testing database. The app version in the production slot should always use the production database. To achieve this, you can configure the database connection string as a slot setting.

When you swap slots, the settings in the target slot (which is typically the production slot) are applied to the app version in the source slot before the hostnames are swapped. You might discover problems at this point. For example, if the database connection string is configured as a slot setting, the new version of the web app will use the existing production database. If you forgot to upgrade the database schema in the production database before the swap, you could see errors and exceptions when the new app version attempts to use the old schema.

To help you discover problems before your app goes live into production, Azure App Service offers a swap-with-preview feature. When you choose this option, the swap proceeds in two phases:
- Phase 1: Slot settings from the target slot are applied to the web app in the source slot. Then Azure warms up the staging slot. At this point, the swap operation pauses so you can test the app in the source slot to make sure it works with the target slot configuration
- Phase 2: The hostnames for the two sites are swapped. 

When you use auto swap, you can't test the new app version in the staging slot before the swap. Auto swap mainly benefits users who want zero-downtime deployments and simple automated deployment pipelines. Auto swap is not available in App Service on Linux.

Suppose that deploying version 3 of your app to production revealed an unexpected problem. To quickly resolve it, you can roll back to the previous version of the site by swapping the slots again.
- Go to the Deployment slots page of the production slot's web app.
- Swap the staging and production slots.
- When the swap finishes, on the Overview page, select Browse to view the app one last time. You'll see that version 2 has been redeployed to production.

##### Scale an App Service web app to efficiently meet demand with App Service scale up and scale out
The key to scaling effectively is knowing when to scale, and by how much. You monitor the performance of a web app by using the metrics available for the App Service. The simplest way to do this is to use the Azure portal. If you notice a steady increase in resource use, such as CPU utilization, memory occupancy, or disk queue length, you should consider scaling out before these metrics hit a critical point. You should also monitor the average response time of requests and the number of failing requests. If both of these figures are high, the system might be running close to (or beyond) capacity. You might need to scale out immediately.

When you hit scale out limits, you must first scale up in order to extend the scale out limitations.

##### Deploy and run a containerized web app with Azure App Service
Security is an important reason to choose Container Registry instead of Docker Hub:
- You have much more control over who can see and use your images.
- You can sign images to increase trust and reduce the chances of an image becoming accidentally (or intentionally) corrupted or otherwise infected.
- All images stored in a container registry are encrypted at rest.

Demo:
1. Create a container registry resource group
2. Build the container directly in the ACR with the command `az acr build --resource-group learn-deploy-container-acr-rg --registry calibali --image webimage .`
3. In the container registry resource, under Services, go to repositories and find the webimage

When you create a web app from a Docker image, you configure the following properties:
- Registry that contains the image:
- Image: This item is the name of the repository.
- Tag: This item indicates which version of the image to use from the repository. By convention, the most recent version is given the tag latest when it's built.
- Startup file: This item is the name of an executable file or a command to be run when the image is loaded. It's equivalent to the command that you can supply to Docker when running an image from the command line by using docker run. If you're deploying a ready-to-run, containerized app that already has the ENTRYPOINT and/or COMMAND values configured, you don't need to fill this in.

After you've configured the web app, the Docker image is pulled, and runs as a cold start operation the first time a user attempts to visit the site.

Continue Demo:
4. Go to Acces Keys under Settings of the Container Registry and enable Admin user
5. Create a webapp in the same resource group and publish from Docker Container (Linux). In the Docker tab, choose ACR under Image Source.

Azure App Service supports continuous deployment using webhooks. A webhook is a service offered by Azure Container Registry. Services and applications can subscribe to the webhook to receive notifications about updates to images in the registry. A web app that uses App Service can subscribe to an Azure Container Registry webhook to receive notifications about updates to the image that contains the web app. When the image is updated, and App Service receives a notification, your app automatically restarts the site and pulls the latest version of the image.

You use the tasks feature of Container Registry to rebuild your image whenever its source code changes automatically. You configure a Container Registry task to monitor the GitHub repository that contains your code and trigger a build each time it changes. If the build finishes successfully, Container Registry can store the image in the repository. If your web app is set up for continuous integration in App Service, it receives a notification via the webhook and updates the app.

Container Registry tasks must be created from the command line. Unlike the `az acr build` command that we ran earlier to build our image, the `az acr task create` command creates and registers a long-lived task.

Before running this command, you need to create a GitHub personal access token with permissions to create a webhook in your repository. For private repositories, the token will also need full repository read permissions.

Continue Demo:
6. In the Container Settings of the Web App, set continuous deployment to On and click Save.
7. Modify the project and rebuild with the command from step 2.
8. Go to the Container Registry resource, click on Webhooks under Services, and select the webhook in the list

### 1.3 Implement Azure Functions {#1.3}

#### Pluralsight notes

##### Introducing Azure Functions
Azure Functions is a serverless application platform, which provides a simple way to run small pieces of code or functions in the cloud. You might hear it referred to as a Functions as a Service platform. Serverless means that rather than having to provision and maintain servers ourselves, we delegate that responsibility to the cloud provider, we simply provide our code.

Azure Functions are grouped into an Azure function app, which is a collection of one or more related Azure functions that are developed, deployed, and hosted as a group.

You can create functions in different languages (most commonly used are C# and JS).

Hosting choices:
1. Consumption Plan. This is the most commonly used and is a serverless pricing plan (bills only for usage) which offers automatic sclain. It also has a 5 minute time limit for each function invocation.
2. App Service Plan. Pay monthly.
3. Premium Plan. Offers higher speeds, more security options and provides reserved instances.
4. Docker container. Allows you to use Azure Functions on premises or in any cloud.
5. Locally. For development and testing

Development environment choices:
1. Azure Portal
2. Visual Studio
3. Cross-platform CLI together with VS Code.

##### Implementing Function Triggers

Every Azure Function has exactly one trigger, which is the event that causes the function to run.

There are three types of triggers:
1. HTTP Request Trigger (Webhooks) - use for APIs and webhooks
2. Timer Trigger - use for scheduled tasks
3. Data operations
    - Queue Trigger - run in response to a message on a queue
    - Cosmos DB Trigger - run when a document is created or updated
    - Blob Trigger - run when a new file is uploaded to Blob Storage

There are also other triggers, such as Event Grid and Microsoft graph.

###### HTTP Request Trigger
HTTP Request triggers are the most used.  
Customization:
    - HTTP method (GET, POST)
    - Route, to control the URL of the function

Three authorization methods for secure Azure Functions:
- Anonymous: no key required
- Function: key per function. The authorization key is in the function URL which you can copy from the Azure Portal.
- Admin: key per function app

There is also a `function.json` file created with each Azure Function. It defines not only the trigger, but also any additional bindings used by your function. This file can be modified if needed.

##### Timer Trigger
Run scheduled tasks

You define a CRON expression, which determins when your function should run

##### Blob Trigger
For the path, you need to indicate the location in the storage account that you want to monitor, for example `samples-workitems/{name}` as the path is looking for any file within the samples‑workitems container, and this special name in curly bracket syntax is called a binding expression, which will allow our function to access the name of the file that triggered the function.

##### Implementing Input and Output bindings
A binding is a connection to data.
- Input bindings provide read-access to data
- Output bindings let us write to an external system

Functions can have multiple input and output bindings

##### Implementing Azure Durable Functions
Azure Durable Functions are an extension to Azure Functions that create stateful, serverless workflows ("orchestrations").

There are three types of Function
1. Client ("Starter") Function. Initiate a new orchestration and use any trigger.
2. Orchestrator Function. Defines the ordered steps in the workflow and handles errors.
3. Activity Function. Implements a step in the workflow and use any bindings.

Orchestration patterns:
- Function chaining: execute a sequence of activity functions in a specified order.
- Fan-out, fan-in: runs multiple activity functions in parallel and then waits for all the activities to finish.
- Asynchronous HTTP APIs: this pattern is useful when you have an API that you need to repeatedly pull for progress. You start off a long‑running operation, and then your orchestrator function can manage the pulling until the operation has completed or timed out.
- The monitor pattern implements a recurring process in a workflow, possibly looking for a change in state.
- Human interaction: This pattern allows workflows to wait for certain events to be raised and then optionally perform a mitigating action if there's a timeout waiting for the human interaction.

Example Durable Function
```csharp
using System.Collections.Generic;
using System.Net.Http;
using System.Threading.Tasks;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.DurableTask; // must be added via the NuGet package manager
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;

namespace DurableFunctionApp
{
    public static class MyOrchestration
    {
        [FunctionName("MyOrchestration")]
        public static async Task<List<string>> RunOrchestrator(
            [OrchestrationTrigger] IDurableOrchestrationContext context)
        {
            var outputs = new List<string>();

            // Replace "hello" with the name of your Durable Activity Function.
            outputs.Add(await context.CallActivityAsync<string>("MyOrchestration_Hello", "Tokyo"));
            outputs.Add(await context.CallActivityAsync<string>("MyOrchestration_Hello", "Seattle"));
            outputs.Add(await context.CallActivityAsync<string>("MyOrchestration_Hello", "London"));

            // returns ["Hello Tokyo!", "Hello Seattle!", "Hello London!"]
            return outputs;
        }

        [FunctionName("MyOrchestration_Hello")]
        public static string SayHello([ActivityTrigger] string name, ILogger log)
        {
            log.LogInformation($"Saying hello to {name}.");
            return $"Hello {name}!";
        }

        [FunctionName("MyOrchestration_HttpStart")]
        public static async Task<HttpResponseMessage> HttpStart(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post")] HttpRequestMessage req,
            [DurableClient] IDurableOrchestrationClient starter,
            ILogger log)
        {
            // Function input comes from the request content.
            string instanceId = await starter.StartNewAsync("MyOrchestration", null);

            log.LogInformation($"Started orchestration with ID = '{instanceId}'.");

            return starter.CreateCheckStatusResponse(req, instanceId);
        }
    }
}
```

Sub-orchestrations allow one orchestration to trigger another orchestration.

##### Implementing Custom Handlers
Implement Azure Functions using your language or runtime of choice

It's the Azure Functions host that receives this trigger along with any associated payload data. However, to execute your custom handler, it makes an ongoing HTTP request to a web server that's running your custom handler code.

Creating a custom handler:
1. Create a Function App, with `func init` selecting "Custom" as the language
2. Create an Azure Function
3. Create a web server using your custom language of choice
4. Compile your custom handler
5. Update host.json (`defaultExecutablePath` and `enableForwardingHttpRequest`)
6. Test locally or publish to Azure

#### Microsoft Learn

##### Choose the best Azure service to automate your business processes
Business processes modeled in software are often called workflows. Azure includes four different technologies that you can use to build and implement workflows that integrate multiple systems:
- Design-first technologies
    - Logic Apps. This tool is designed for users who have a good understanding of the business process but no coding skills. One reason why Logic Apps is so good at integration is that over 200 connectors are included. A connector is a Logic Apps component that provides an interface to an external service.
    - Microsoft Power Automate. This tool is designed for people with development skills. There are four different types of flow that you can create:
        - Automated: A flow that is started by a trigger from some event. For example, the event could be the arrival of a new tweet or a new file being uploaded.
        - Button: Use a button flow to run a repetitive task with a single click from your mobile device.
        - Scheduled: A flow that executes on a regular basis such as once a week, on a specific date, or after 10 hours.
        - Business process: A flow that models a business process such as the stock ordering process or the complaints procedure. The flow process can have: notification to required people; with their approval recorded; calendar dates for steps; and recorded time of flow steps.
- Code-first technologies. The Azure App Service is a cloud-based hosting service for web applications, mobile back-ends, and RESTful APIs.
    - WebJobs. WebJobs are a part of the Azure App Service that you can use to run a program or script automatically and only support C# in Microsoft Windows. With Web Jobs, you pay for the entire VM or App Service Plan that hosts the job. The SDK includes a range of classes, such as `JobHostConfiguration` and `HostBuilder`, which reduce the amount of code required to interact with the Azure App Service. There are two kinds of WebJob:
        - Continuous: these WebJobs run in a continuous loop.
        - Triggered. These WebJobs run when you manually start them or on a schedule.
    - Azure Functions

You choose WebJobs over Functions when:
- You want the code to be a part of an existing App Service application and to be managed as part of that application, for example in the same Azure DevOps environment.
- You need close control over the object that listens for events that trigger the code. This object in question is the `JobHost` class, and you have more flexibility to modify its behavior in WebJobs.

These four technologies have some similarities. For example:
- They can all accept inputs. An input is a piece of data or a file that is supplied to the workflow.
- They can all run actions. An action is a simple operation that the workflow executes and may often modify data or cause another action to be performed.
- They can all include conditions. A condition is a test, often run against an input, that may decide which action to execute next.
- They can all produce outputs. An output is a piece of data or a file that is created by the workflow.

##### Create serverless logic with Azure Functions
Serverless compute can be thought of as a function as a service (FaaS), or a microservice that is hosted on a cloud platform.

Azure Functions is a serverless application platform. It allows developers to host business logic that can be executed without provisioning infrastructure.

Benefits of a serverless compute solution:
- Avoids over-allocation of infrastructure
- Stateless logic (but, if state is required, it can be stored in an associated storage service)
- Functions can be used in traditional compute environments

Drawbacks of a serverless compute solution:
- Execution time: by default, functions have a timeout of 5 minutes. There's also an option called Durable Functions that allows you to orchestrate the executions of multiple functions without any timeout.
- Execution frequency

Functions are hosted in an execution context called a function app. You define function apps to logically group and structure your functions and a compute resource in Azure.

Service plans:
- The Consumption service plan provides automatic scaling and bills you when your functions are running.
- Azure App Service plan. This plan allows you to avoid timeout periods by having your function run continuously on a VM that you define.

When you create a function app, it must be linked to a storage account. The function app uses this storage account for internal operations such as logging function executions and managing execution triggers. On the Consumption service plan, this is also where the function code and configuration file are stored.

Azure supports triggers for the following services:
| Service | Trigger description |
| ------- | ------------------- |
| Blob Storage | Starts a function when a new or updated blob is detected. |
| Azure Cosmos DB | Start a function when inserts and updates are detected. |
| Event Grid | Starts a function when an event is received from Event Grid. |
| HTTP | Starts a function with an HTTP request. |
| Microsoft Graph Events | Starts a function in response to an incoming webhook from the Microsoft Graph. Each instance of this trigger can react to one Microsoft Graph resource type. |
| Queue Storage | Starts a function when a new item is received on a queue. The queue message is provided as input to the function. |
| Service Bus | Starts a function in response to messages from a Service Bus queue. |
| Timer	 | Starts a function on a schedule. |

A trigger is a special type of input binding that has the additional capability of initiating execution.

For PowerShell functions, output bindings are explicitly written to with the `Push-OutputBinding` cmdlet.

Test your Azure Function:
- Manual execution
- Test in the Azure portal

The Application Insights dashboard provides a quick way to view the history of function executions, and displays the timestamp, result code, duration, and operation ID populated by Application Insights.

You're also able to add logging statements to your function for debugging in the Azure portal.
```powershell
Write-Host "Enter your logging statement here"
```

When using the function or admin authorization method, we will need to supply the key when we send the HTTP request. You can send it as a query string parameter named code, or as an HTTP header (preferred) named x-functions-key.

##### Execute an Azure Function with triggers

###### Timer trigger
To create a timer trigger, you need to supply two pieces of information.
- A Timestamp parameter name, which is simply an identifier to access the trigger in code.
- A Schedule, which is a CRON expression that sets the interval for the timer.

A CRON expression is a string that consists of six fields that represent a set of times. The order of the six fields in Azure is: `{second}` `{minute}` `{hour}` `{day}` `{month}` `{day of the week}`.

| Special character | Meaning | Example |
| ----------------- | ------- | ------- |
| * | Selects every value in a field | An asterisk "*" in the day of the week field means every day. |
| , | Separates items in a list | A comma "1,3" in the day of the week field means just Mondays (day 1) and Wednesdays (day 3). |
| - | Specifies a range | A hyphen "10-12" in the hour field means a range that includes the hours 10, 11, and 12. |
| /	| Specifies an increment | A slash "*/10" in the minutes field means an increment of every 10 minutes. |

###### HTTP Trigger
HTTP triggers have many capabilities and customizations, including:
- Provide authorized access by supplying keys. If your Authorization level is set to Function, you can use either a function or a host key. If your Authorization level is set to Admin, you must supply a host key.
- Restrict which HTTP verbs are supported.
- Return data back to the caller.
- Receive data through query string parameters or through the request body.
- Support URL route templates to modify the function URL.

After you have the URL for your function, you can send HTTP requests. If your function receives data, remember that you can use either query string parameters or supply the data through the request body.

###### Execute an Azure function when a blob is created
Azure Blob storage is great at doing things like:
- Storing files
- Serving files
- Streaming video and audio
- Logging data

There are three types of blobs: block blobs, append blobs, and page blobs. Block blobs are the most common type. They allow you to store text or binary data efficiently. Append blobs are like block blobs, but they're designed more for append operations like creating a log file that's being constantly updated. Finally, page blobs are made up of pages and are designed for frequent random read and write operations.

One setting that you'll want to look at is the Path. The Path tells the blob trigger where to monitor to see if a blob is uploaded or updated. By default, the Path value is: `samples-workitems/{name}`. Let's break down this concept into two pieces: samples-workitems and {name}. The first part, samples-workitems, represents the blob container that the trigger monitors. The second part, {name} means that every type of file will cause the trigger to invoke the function. The function is invoked because there's no filter. For example, we could make the trigger invoke the function only when a PNG file is added by using syntax like: `samples-workitems/{name}.png`. The name represents a parameter in your Azure function that receives the name of the added file. For example, if we upload a file named resume.txt, my Azure function receives that value as a string through a parameter called name.

##### Chain Azure Functions together using input and output bindings
In Azure Functions, bindings provide a declarative way to connect to data from within your code.

There are two kinds of bindings you can use with functions:
- Input binding - Connects to a data source. Our function can read data from these inputs.
- Output binding - Connects to a data destination. Our function can write data to these destinations.

There are also triggers, which are special types of input bindings that cause a function to run.

Three properties are required in all bindings. You may have to supply additional properties based on the type of binding and storage you are using.
- Name - Defines the function parameter through which you access the data.
- Type - Identifies the type of binding.
- Direction - Indicates the direction data is flowing.

Additionally, most binding types also need a fourth property:
- Connection - Provides the name of an app setting key that contains the connection string.

Example code:
```powershell
using namespace System.Net

# Input bindings are passed in via param block.
param($Request, $TriggerMetadata)

# Write to the Azure Functions log stream.
Write-Host "PowerShell HTTP trigger function processed a request."

# Interact with query parameters or the body of the request.
$name = $Request.Query.Name
if (-not $name) {
    $name = $Request.Body.Name
}

$body = "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response."

if ($name) {
    $body = "Hello, $name. This HTTP triggered function executed successfully."
}

# Associate values to output bindings by calling 'Push-OutputBinding'.
Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = [HttpStatusCode]::OK
    Body = $body
})
```

example binding:
```json
{
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "Request",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "Response"
    }
  ]
}
```

In the preceding code for our function, we saw how we accessed the payload of the incoming HTTP request through our Request parameter. Similarly, we sent an HTTP response simply by setting our Response parameter. Bindings really do take care of some of the burdensome work for us.

There are a number of types of binding expressions, including:
- App settings
- Trigger filename
- Trigger metadata
- JSON payloads
- New GUID
- Current date and time

Most expressions are identified by being wrapped in curly braces. However, app setting binding expressions are wrapped in percent signs rather than curly braces. For example, if the blob output binding path is %Environment%/newblob.txt and the Environment app setting value is Development, a blob will be created in the Development container.

How do you connect to a database from a function and read data? In the world of functions, you configure an input binding for that job.

##### Create a long-running serverless workflow with Durable Functions

Durable Functions is an extension of Azure Functions that enables you to perform long-lasting, stateful operations in Azure. Azure provides the infrastructure for maintaining state information. You can use Durable Functions to orchestrate a long-running workflow. Using this approach, you get all the benefits of a serverless hosting model, while letting the Durable Functions framework take care of activity monitoring, synchronization, and runtime concerns.

Durable Functions scales as needed, and provides a cost effective means of implementing complex workflows in the cloud. Some benefits of using Durable Functions include:
- They enable you to write event driven code. A durable function can wait asynchronously for one or more external events, and then perform a series of tasks in response to these events.
- You can chain functions together. You can implement common patterns such as fan-out/fan-in, which uses one function to invoke others in parallel, and then accumulate the results.
- You can orchestrate and coordinate functions, and specify the order in which functions should execute.
- The state is managed for you. You don't have to write your own code to save state information for a long-running function.

Durable functions allows you to define stateful workflows using an orchestration function. An orchestration function provides these extra benefits:
- You can define the workflows in code. You don't need to write a JSON description or use a workflow design tool.
- Functions can be called both synchronously and asynchronously. Output from the called functions is saved locally in variables and used in subsequent function calls.
- Azure checkpoints the progress of a function automatically when the function awaits. Azure may choose to dehydrate the function and save its state while the function waits, to preserve resources and reduce costs. When the function starts running again, Azure will rehydrate it and restore its state.

When setting a timer for delay, you should always use `currentUtcDateTime` to obtain the current date and time, instead of `Date.now` or `Date.UTC`.

##### Create and run Azure Functions locally by using the Core Tools

Once you start using a local development workflow based on the Core Tools, don't expect to be able to use the portal to make changes to your functions.

Every function published to Azure belongs to a function app: a collection of functions that are published together into the same environment. All of the functions in an app share a common set of configuration values, and must all be built for the same language runtime. Each function app is an Azure resource that can be configured and managed independently.

A functions project on your computer is equivalent to a function app in Azure, and can contain multiple functions that use the same language runtime.

Create a new functions project with `func init`.

Create a new function with `func new`.

To start the functions host locally, run `func start`.

Before you can use the Core Tools to publish a project, you need to create a function app in Azure. This is not a capability of the Core Tools: creating function apps is one of the responsibilities of the Azure management tools, which include the Azure portal, Azure CLI and Azure PowerShell. In the next exercise, we'll use the Azure CLI's `az functionapp` create command to create a function app to which we can publish our code. If you already have a local functions project you want to publish, make sure to create the function app with the same language runtime.

When you publish, any functions already present in the target app are stopped and deleted before the contents of your project are deployed.

##### Develop, test, and deploy an Azure Function with Visual Studio
Use v2 triggers of the runtime environment required to run Azure Functions wherever possible.

An Azure Function is implemented as a static class. The class provides a static, asynchronous method named `Run`, which acts as the entry point for the class. The parameters passed to the `Run` method provide the context for the trigger. In the case of an HTTP trigger, the function receives an HttpRequest object. The function returns a value containing any output data and results, wrapped in an IActionResult object. The value is returned in the body of the HTTP response for the request.

Different types of trigger receive different input parameters and return types.

In all cases, an Azure Function is passed an ILogger parameter. The function can use this parameter to write log messages, which the function app will write to storage for later analysis.

An Azure Function also contains metadata that specify the type of the trigger and any other specific information and security requirements.

Deployment from Visual Studio is great feature for developers. If developers have access to an Azure subscription, they can create an Azure Functions app and publish code to Azure, to perform testing in an environment that is similar to production environment.

Azure Functions integrates with BitBucket, Dropbox, GitHub, and Azure DevOps. This enables a workflow where function code updates made by using one of these integrated services triggers deployment to Azure.

You can configure continuous deployment from the Azure portal, using the Deployment Center feature of an Azure Functions app. Deployment is configured on a per-function app basis. Azure Functions can also be deployed from a zip file using the push deployment technique: `az functionapp deployment source config-zip -g <resource-group> -n <function-app-name> --src <zip-file>`

##### Monitor GitHub events by using a webhook with Azure Functions

The Gollum event allows you to listen for wiki updates; specifically creation and updates for a wiki page.

Webhooks are user-defined HTTP callbacks. When the event occurs, the source site makes an HTTP request to the URL configured for the webhook. With Azure Functions, we can define logic in a function that can be run when a webhook message is received.

Setting up a webhook is a two-step process. You specify how you want your webhook to behave through GitHub and what events it should listen to. Then you set up your function in Azure Functions to receive and manage the payload received from the webhook. To set up the webhook, go to the settings page of your repository in the GitHub portal. Select Webhooks, and then select Add webhook.

Webhooks can be delivered using two different content types:
- The application/json content type delivers the JSON payload directly as the body of the POST request.
- The application/x-www-form-urlencoded content type sends the JSON payload as a form parameter, called payload.

The payload for the Gollum event contains the following items:
- `pages` Pages that were updated. Each page includes the following information:
- `page_name` Name of the page.
- `title` Current page title.
- `action` Action that was performed on the page. Can be created or edited.
- `html_url` HTML wiki page. repository information about the repository containing the wiki page, including:
    - `name` Name of the repository.
    - `owner` Details of the owner of the repository.
    - `html_url` Address of the repository. sender information about the user that raised the event that caused the webhook to fire.

Setting a webhook secret allows you to ensure that POST requests sent to the payload URL are from GitHub. When you set a secret, you'll receive the `x-hub-signature` header in the webhook POST request. In GitHub, you can set the secret field by going to the repository where you have setup your webhook, and then editing the webhook. When your secret token is set, GitHub uses it to create a hash signature for each payload. This hash signature is passed along with each request in the headers as `x-hub-signature`. When your function receives a request, you need to compute the hash using your secret, and ensure that it matches the hash in the request header. GitHub uses an HMAC SHA1 hex digest to compute the hash, so you must calculate your hash in this same way, using the key of your secret and your payload body. The hash signature starts with the text `sha1=`.

##### Enable automatic updates in a web application using Azure Functions and SignalR Service

In contrast to polling, a more favorable design features persistent connections between the client and server. Establishing a persistent connection allows the server to push data to the client at will.

SignalR is an abstraction for a series of technologies that allows your app to enjoy two-way communication between the client and server. SignalR handles connection management automatically, and lets you broadcast messages to all connected clients simultaneously, like a chat room. You can also send messages to specific clients. The connection between the client and server is persistent, unlike a classic HTTP connection, which is re-established for each communication.

The abstraction layer offered by SignalR provides two benefits to your application.
- The first advantage is future-proofing your app. As the web evolves and APIs superior to WebSockets become available, your application doesn't need to change. You could update to a version of SignalR that supports any new APIs and your application code won't need an overhaul.
- The second benefit is that SignalR allows your application to gracefully degrade depending on supported technologies of the client. If it doesn't support WebSockets, then Server Sent Events are used. If the client can't handle Server Sent Events, then it uses Ajax long polling, and so on.

The first step is to run the following command in the Cloud Shell to create a new SignalR account in the sandbox resource group. For SignalR Service to work properly with Azure Functions, you need to set its service mode to Serverless.

In order for the (Azure Cosmos) database to push changes, the DB still has to monitor the changes. When you build a Function with an Azure Cosmos DB trigger, you can add a parameter `feedPollDelay` to the input binding in `function.json`. The `feedPollDelay` refers to how the internals of Azure Cosmos DB recognize changes, not how your web application exposes changes to the data.

##### Expose multiple Azure Function apps as a consistent API by using Azure API Management

When you build an application as a collection of microservices, you create many different small services. Each service has a defined domain of responsibility, and is developed, deployed, and scaled independently. This modular architecture results in an application that is easier to understand, improve, and test. It also makes continuous delivery easier, because you change only a small part of the whole application when you deploy a microservice.

Azure API Management (APIM) is a fully managed cloud service that you can use to publish, secure, transform, maintain, and monitor APIs. It helps organizations publish APIs to external, partner, and internal developers to unlock the potential of their data and services. API Management handles all the tasks involved in mediating API calls, including request authentication and authorization, rate limit and quota enforcement, request and response transformation, logging and tracing, and API version management. APIM enables you to create and manage modern API gateways for existing backend services no matter where they are hosted.

Because you can publish Azure Functions through API Management, you can use them to implement a microservices architecture; each function implements a microservice. By adding several functions to a single API Management product, you can build those microservices into an integrated distributed application. Once the application is built, you can use API Management policies to implement caching or ensure security requirements.

When you choose a usage plan for API Management, you can choose the consumption tier, because it's especially suited to microservice-based architectures and event-driven systems.

From the Function App, on the left hand side, under API, select API management, click create new.

The microservices approach to architecture creates a modular application in which each part is loosely coupled to the others. This method makes it easier to implement continuous delivery because new versions of each service can be deployed independently of the others. If bugs are not detected during testing and make it through to production, their impact is reduced and it is easier to roll back to a stable version. Also, you can create small, autonomous teams of developers for each microservice. This division fits well with modern Agile practices. However, microservices architectures can also present challenges, such as:
- Client apps are coupled to microservices. If you want to change the location or definition of the microservice, you may have to reconfigure or update the client app.
- Each microservice may be presented under different domain names or IP addresses. This presentation can give an impression of inconsistency to users and can negatively affect your branding.
- It can be difficult to enforce consistent API rules and standards across all microservices. For example, one team may prefer to respond with XML and another may prefer JSON.
- You're reliant on individual teams to implement security in their microservice correctly. It's difficult to impose these requirements centrally.

By adding multiple APIs, functions, and other services to API Management, you can assemble those components into an integrated product that presents a single entry point to client applications. Composing an API using API Management has advantages that include:
- Client apps are coupled to the API expressing business logic, not the underlying technical implementation with individual microservices. You can change the location and definition of the services without necessarily reconfiguring or updating the client apps.
- API Management acts as an intermediary. It forwards requests to the right microservice, wherever it is located, and returns responses to users. Users never see the different URIs where microservices are hosted.
- You can use API Management policies to enforce consistent rules on all microservices in the product. For example, you can transform all XML responses into JSON, if that is your preferred format.
- Policies also enable you to enforce consistent security requirements.

API Management also includes helpful tools - you can test each microservice and its operations to ensure that they behave in accordance with your requirements. You can also monitor the behavior and performance of deployed services.

Azure API Management supports importing Azure Function Apps as new APIs or appending them to existing APIs. The process automatically generates a host key in the Azure Function App, which is then assigned to a named value in Azure API Management.

We can also access them both by using the same subscription key, because that key grants access to the API Management gateway. In other Learn modules, you can learn how to apply policies, security settings, external caches, and other features to all the functions in an API Management Gateway. The gateway provides you with a central control point, where you can manage multiple microservices without altering their code.

## 2 Develop for Azure Storage (15-20%) {#2}

### 2.1 Develop solutions that use Cosmos DB storage {#1.1}

#### Pluralsight

##### Creating Cosmos DB Containers
Relational databases don't scale well. NoSQL Databases tended to be very well suited for working in a distributed manner in the cloud.

Microsoft says, "If you're transactional volumes are reaching extreme levels, such as many thousands of transactions per second, you should consider a distributed NoSQL database."

Azure Cosmos DB:
- Provides extremely low latency (single digit millisecond)
- Provides SLA throughput, latency, availability, and consistency
- Support multi-region replication at any point
- Provides five-nines of high-availability for both reads and writes
- Enables elastic scalability
- Pricing is for the throughput you provision, but now we also have serverless solutions
- Supports multiple consistency options

Additional features:
- Integrated analytics (has Spark)
- Region support
- Schema-agnostic
- Automatic indexing
- Supports multiple SDK's

Cosmos DB organization:
1. You start with a Cosmos DB Account. This is the high level container that has to be created first. You have to select an API at the account level.
2. Inside, you have databases.
3. Within a database, you have a container, which are similar to tables in relational databases
4. Inside the containers, you have items.

Supported Cosmos DB API's
- SQL. It enables you to use a SQL-like query syntax for querying your data.
- Cassandra.
    - You want to leverage the CQL to query data
    - Company already uses Cassandra tools
    - You have existing Cassandra databases you want to migrate to the cloud
    - You want to store data in a wide-column format
- MongoDB
    - Similar as first three reasons as for Cassandra
    - You want to store data in JSON format
- Gremlin
    - You need to store graph relations between data (e.g. for recommender systems)
    - Leverage Apache Tinerpop's Gremlin language for querying relationships
- Azure Table API
    - You have experience with Azure Table Storage
    - You have applications and data to migrate from Azure Table Storage
    - You want to query data using OData or LINQ queries
- SQL API (this is the core API)
    - Leverage SQL-like language to query data
    - Store data as JSON documents
    - If no other use cases fit, choose the SQL API

Organization of the different APIs
| API | Database Entity | Container Entity |
| --- | --------------- | ---------------- |
| SQL | Database | Container |
| Cassandra | Keyspace | Table |
| MongoDB | Database | Collection |
| Gremlin | Database | Graph |
| Azure Table | Not Applicable | Table |

Create a Cosmos DB Container example
```bash
# create a sql api db account
az cosmosdb create --name pluralsight --resource-group pluralsight
# create a sql database
az cosmosdb sql database create --account-name pluralsight --name sampledb
# create a sql database container
az cosmosdb sql container create --resource-group pluralsight --account-name pluralsight --database-name sampledb --name samplecontainer --partition-key-path "/employeeid"
```

Selecting an SDK
- When using the SQL API, utilize the latest Cosmos DB SDK for your platform
- When using MongoDB, Cassandra, and Gremlin use current SDK's for those API's
- When leveraging the Azure Table API, leverage the current Table Storage SDK. But note, as mentioned earlier, there are some differences between using Azure Table storage and using Cosmos DB's Table Storage API, so make sure that you understand the impact of those differences.

##### Cosmos DB Performance

Traditional database scaling is done through vertical and horizontal scaling. But Cosmos DB doesn't work that way, because it is a managed service. Here, we work with Request Units (RUs), which encapsulates many of the resources for the database into a single unit. As a baseline, one RU is equal to a 1kb item read operation from a Cosmos DB container.

Resources encapsulated in RU's
- Processing power (CPU)
- Memory
- IOPS (input/output operations per second)

Managing Cosmos DB throughput
- Provisioned
    - ideal for always-on production implementation
    - can be configured at the database or container
    - throughput is evenly distribitued to partitions
    - requires 10 RU's per GB of storage
    - once RU's are consumed for a partition, future requests will be rate limited
    - by default, it requires a manual scaling approach to acquire more RU's
    - autocaling: with provisioned throughput, you can sepcify a maximum RU throughput amount, and Cosmos DB will ensure that your data is available up to that throughput amount. The minimum througput is calculated as 10% of the maximum.
- Serverless
    - pay only for the request units consumed and storage used
    - ideal for off and on workloads such as development workloads
    - requires a new account type

Best practices
- Use a partition strategy to evenly spread throughput on partitions
- Provision throughput at the container for predictable performance
- Use the serverless account type for development workloads
- Understand the link between consistency types and the amount of RU's confirmed

Data consistency levels: distributed databases that rely on replication for high availability, low latency, or both, must make a fundamental tradeoff between the read consistency, availability, latency, and througput.

Read consistency: when we make a write operation to the database, what data do we get back if we immediatly read that value. This is best explained by looking at the consistency level spectrum:

Eventual (consistency) <--- Consistent prefix --- Session --- Bounded Staleness ---> Strong  
Lower latency <---> Higher latency  
Higher throughput <---> Lower throughput  
Higher availability <---> Lower availability  

Consistency levels:
- Strong consistency: guarantees that reads get most recent version of item
- Bounded staleness: guarantees that a read has a max lag (either a bound on versions or time)
- Session consistency: guarantees that a client will read its own writes
- Consistent prefix consistency: guarantees that updates are returned in order
- Eventual consistency: provides no guarantee for order or that you can read your own writes. The data written on a node nearby will 'eventually' arrive at a node further away for reads.

Consistency levels and API's
- SQL API: consistency level is set at the account default, but can be overriden at the request-level.
- Gremlin and Azure Table API's, Cosmos DB uses account default consistency level
- Cassandra & MongoDB:
    - writes: the account default consistency level is used
    - reads: the client consistency is mapped to a Cosmos DB level

Throughput considerations:
Both strong and bounded staleness reads will consume twice the normal amount of RU's for a request, as Cosmos DB will need te query two replicas to meet the criteria of the consistency level.

Cosmos DB partitions can have a logical partition or a physical partition. Also, two important concepts are 1) the use of the partition key and 2) the replica set.

Logical partition:  
A logical partition is a set of items within a container that share the same partition key. A container can have as many logical partitions as it needs, but eacht partition is limited to 20GB of storage. Logical partitions are managed by Cosmos DB, but their use is governed by your partition key strategy. Microsoft Cosmos DB Docs: "A container is scaled by distributing data and throughput across physical partitions. Internally, one or more logical partitions are mapped to a single physical partition. They are entirely managed by Azure Cosmos DB."

Replica set:  
A physical partition contains multiple replicas of the data, known as a replica set. By having this data replicated, you enable your storage to be durable and fault tolerant. These replica sets are managed by Cosmos DB.

Partition key:
- serves as the means of routing your request to the correct partition
- made up of both the key and the value of the defined partition key
- should be a value that does not change for the item
- should have many different values represented in the container

Microsoft Cosmos DB Docs: "Azure Cosmos DB uses hash-based partitioning to spread logical partitions across physical partitions. Azure Cosmos DB hashes the partition key value of an item. Then, Azure Cosmos DB allocates the key space of partition key hashes evenly across the physical partitions."

Strategy considerations:
- throughput is distributed evenly across all of your physical partitions. This means that you have to minimize the chance of 'hot partitions', by evenly distributing your requested data across physical partitions.
- multi-item transactions require triggers or stored procedures
- you will want to minimize cross-partition queries for heavier workloads
- decide upon a partition strategy before creating your container

Example, if we look at a value like employeeId, this becomes an ideal choice. Number one, because we won't need to change it. We're also going to have a wide variety in the IDs that are chosen. That's going to enable it to give that proper key space by hashing the value to several different partitions. Now Cosmos DB will handle splitting of partitions for you. You don't have to manage that, but you do need to be sure that there's enough difference in the values so you don't end up with a hot partition.

##### Server-side Programming with Cosmos DB

Cosmos DB provides a robust approach for executing code in response to actions taken on the data stored in Cosmos DB. While in some cases these mirror traditional database constructs, they are fundamentally different in implementation and have unique limitations.

Cosmos DB server-side concepts:
- Internal to the database engine
    - Stored Procedures
    - Triggers
    - User defined functions (UDF's)
- External to the database engine
    - Change feed

Server-side execution environment
- If you write a stored procedure, trigger or UDF, it has to be written in JavaScript
- Can be created and managed via the portal and via the SDK

Stored procedures:
- Must be defined in JS
- Executes on a single partition, and it only has acces to that partition. Throws an error if you try to execute the stored procedure on a different partition.
- Partition key must be provided with the execution request
- Supports a transaction model as all statements will be removed if it fails

Triggers:
- Must be defined in JS
- Triggers can be executed either before (pre) and (post) data is written
    - Pre triggers can handle data transformation and validation
    - Post triggers can handle aggregation and change notifications
- Triggers are not guaranteed to execute, as they have to be specified in a request.
- Errors in either the pre or post trigger will result in data being rolled back

UDF's
- Must be defined in JS
- Enables you to define a custom function that can be leveraged in a query
- Enables encapsulation of common logic in query conditions

Change Feed Processing: while all other server-side programming approaches enable executioni on the Cosmos DB engine, change feed processing enables you to react to data changes using server-side code outside of the Cosmos DB engine.

Change Feed:
- Enables you to be notified for any insert and update on your data
- Deletes are not directly supported, but you can leverage a soft-delete flag
    - you can still support deletes with TTL's
- A change will appear exactly once in the change feed
- Reading data from the database will consume throughput
- Partition updates will be in order, but between partitions there is no guarantee
- Not supported for the Azure Table API

Change feed approaches:
- Azure functions
- Change feed processor, which is supported with some SDK's.

In reality the two approaches are the same thing, except with Azure Functions it's got the change feed processor built in and you don't have to manage the provisioning of workers to go out and deal with the changes that rol in.

#### Microsoft Learn

##### Choose a data storage approach in Azure

Kinds of data
1. Structured data, sometimes referred to as relational data, is data that adheres to a strict schema, so all of the data has the same fields or properties.
2. Semi-structured data is less organized than structured data, and is not stored in a relational format, as the fields do not neatly fit into tables, rows, and columns. Semi-structured data contains tags that make the organization and hierarchy of the data apparent - for example, key/value pairs. Semi-structured data is also referred to as non-relational or NoSQL data. The expression and structure of the data in this style is defined by a serialization languagem, like xml, json and yaml.
    - Key-value databases:
        - Use key-value pairs
        - No query language
        - Simple commands, like get, put and delete
    - Graph databases
        - Point to other nodes
        - Links of relationships
    - Document database
        - Use mark-up language, like XML, JSON
3. Photos, videos, and other similar files are classified as unstructured data.

Once you've identified the kind of data you're dealing with (structured, semi-structured, or unstructured), the next step is to determine how you'll use the data. Ask yourself these questions:
- Will you be doing simple lookups using an ID?
- Do you need to query the database for one or more fields?
- How many create, update, and delete operations do you expect?
- Do you need to run complex analytical queries?
- How quickly do these operations need to complete?

A transaction is a logical group of database operations that execute together. Transactions are often defined by a set of four requirements, referred to as ACID guarantees. ACID stands for Atomicity, Consistency, Isolation, and Durability:
- Atomicity means a transaction must execute exactly once and must be atomic; either all of the work is done, or none of it is. Operations within a transaction usually share a common intent and are interdependent.
- Consistency ensures that the data is consistent both before and after the transaction.
- Isolation ensures that one transaction is not impacted by another transaction.
- Durability means that the changes made due to the transaction are permanently saved in the system. Committed data is saved by the system so that even in the event of a failure and system restart, the data is available in its correct state.

Transactional databases are often called OLTP (Online Transaction Processing) systems. OLTP systems commonly support lots of users, have quick response times, and handle large volumes of data. They are also highly available (meaning they have very minimal downtime), and typically handle small or relatively simple transactions.

On the contrary, OLAP (Online Analytical Processing) systems commonly support fewer users, have longer response times, can be less available, and typically handle large and complex transactions.

##### Create an Azure Storage account

Azure provides many ways to store your data. There are multiple database options like Azure SQL Database, Azure Cosmos DB, and Azure Table Storage. Azure offers multiple ways to store and send messages, such as Azure Queues and Event Hubs. You can even store loose files using services like Azure Files and Azure Blobs. Azure selected four of these data services and placed them together under the name Azure Storage. The four services are Azure Blobs, Azure Files, Azure Queues, and Azure Tables. These four were given special treatment because they are all primitive, cloud-based storage services and are often used together in the same application.

A storage account is a container that groups a set of Azure Storage services together. Combining data services into a storage account lets you manage them as a group. A storage account is an Azure resource and is included in a resource group.

A storage account defines a policy that applies to all the storage services in the account. The settings that are defined by a storage account are:
- Subscription: The Azure subscription that will be billed for the services in the account.
- Location: The datacenter that will store the services in the account.
- Performance: Determines the data services you can have in your storage account and the type of hardware disks used to store the data. Standard allows you to have any data service (Blob, File, Queue, Table) and uses magnetic disk drives. Premium introduces additional services for storing data. For example, storing unstructured object data as block blobs or append blobs, and specialized file storage used to store and create premium file shares. These storage accounts use solid-state drives (SSD) for storage.
- Replication: Determines the strategy used to make copies of your data to protect against hardware failure or natural disaster. At a minimum, Azure will automatically maintain three copies of your data within the data center associated with the storage account. This is called locally-redundant storage (LRS), and guards against hardware failure but does not protect you from an event that incapacitates the entire datacenter. You can upgrade to one of the other options such as geo-redundant storage (GRS) to get replication at different datacenters across the world.
- Access tier: Controls how quickly you will be able to access the blobs in this storage account. Hot gives quicker access than Cool, but at increased cost. This applies only to blobs, and serves as the default value for new blobs.
- Secure transfer required: A security feature that determines the supported protocols for access. Enabled requires HTTPs, while disabled allows HTTP.
- Virtual networks: A security feature that allows inbound access requests only from the virtual network(s) you specify.

You need one storage account for every group of settings that you want to apply to your data. The number of storage accounts you need is typically determined by your:
- data diversity: where it is consumed, how sensitive it is, which group pays the bills, etc.
- cost sensitivity: a storage account by itself has no financial cost; however, the settings you choose for the account do influence the cost of services in the account.
- tolerance for management overhead: each storage account requires some time and attention from an administrator to create and maintain. It also increases complexity for anyone who adds data to your cloud storage; everyone in this role needs to understand the purpose of each storage account so they add new data to the correct account.

A typical strategy is to start with an analysis of your data and create partitions that share characteristics like location, billing, and replication strategy, and then create one storage account for each partition.

Account settings (storage account itself, not the resources):
- Name: must be globally unique within Azure, use only lowercase letters and digits and be between 3 and 24 characters.
- Deployment model:
    - Resource Manager: the current model that uses the Azure Resource Manager API and adds the concept of a resource group, which is not available in the classic model. Microsoft recommends this model for all new resources.
    - Classic: a legacy offering that uses the Azure Service Management API
- Account kind: a set of policies that determine which data services you can include in the account and the pricing of those services. here are three kinds of storage accounts:
    - StorageV2 (general purpose v2): the current offering that supports all storage types and all of the latest features. Microsoft recommends that you use the General-purpose v2 option for new storage accounts.
    - Storage (general purpose v1): a legacy kind that supports all storage types but may not support all features
    - Blob storage: a legacy kind that allows only block blobs and append blobs
    Microsoft recommends that you use the General-purpose v2 option for new storage accounts.

##### Connect an app to Azure Storage

Microsoft Azure Storage is a managed service that provides durable, secure, and scalable storage in the cloud:

| Durable |	Redundancy ensures that your data is safe in the event of transient hardware failures. You can also replicate data across datacenters or geographical regions for extra protection from local catastrophe or natural disaster. Data replicated in this way remains highly available in the event of an unexpected outage. |
| Secure | All data written to Azure Storage is encrypted by the service. Azure Storage provides you with fine-grained control over who has access to your data. |
| Scalable | Azure Storage is designed to be massively scalable to meet the data storage and performance needs of today's applications. |
| Managed | Microsoft Azure handles maintenance and any critical problems for you. |

A single Azure subscription can host up to 200 storage accounts, each of which can hold 500 TB of data.

Azure storage includes four types of data:
- Blobs: A massively scalable object store for text and binary data. Can include support for Azure Data Lake Storage Gen2.
- Files: Managed file shares for cloud or on-premises deployments.
- Queues: A messaging store for reliable messaging between application components.
- Table Storage: A NoSQL store for schema-less storage of structured data. Table Storage is not covered in this module.

Blob storage is ideal for:
- Serving images or documents directly to a browser, including full static websites.
- Storing files for distributed access.
- Streaming video and audio.
- Storing data for backup and restoration, disaster recovery, and archiving.
- Storing data for analysis by an on-premises or Azure-hosted service.

Azure Storage supports three kinds of blobs:
| Blob type | Description |
| --------- | ----------- |
| Block blobs |	Block blobs are used to hold text or binary files up to ~5 TB (50,000 blocks of 100 MB) in size. The primary use case for block blobs is the storage of files that are read from beginning to end, such as media files or image files for websites. They are named block blobs because files larger than 100 MB must be uploaded as small blocks. These blocks are then consolidated (or committed) into the final blob. |
| Page blobs | Page blobs are used to hold random-access files up to 8 TB in size. Page blobs are used primarily as the backing storage for the VHDs used to provide durable disks for Azure Virtual Machines (Azure VMs). They are named page blobs because they provide random read/write access to 512-byte pages. |
| Append blobs | Append blobs are made up of blocks like block blobs, but they are optimized for append operations. These blobs are frequently used for logging information from one or more sources into the same blob. For example, you might write all of your trace logging to the same append blob for an application running on multiple VMs. A single append blob can be up to 195 GB. |

Azure Files enables you to set up highly available network file shares that can be accessed using the standard Server Message Block (SMB) protocol. This means that multiple VMs can share the same files with both read and write access. You can also read the files using the REST interface or the storage client libraries. You can also associate a unique URL to any file to allow fine-grained access to a private file for a set period of time. File shares can be used for many common scenarios:
- Storing shared configuration files for VMs, tools, or utilities so that everyone is using the same version.
- Log files such as diagnostics, metrics, and crash dumps.
- Shared data between on-premises applications and Azure VMs to allow migration of apps to the cloud over a period of time.

The Azure Queue service is used to store and retrieve messages. Queue messages can be up to 64 KB in size, and a queue can contain millions of messages. Queues are used to store lists of messages to be processed asynchronously.

You can use queues to loosely connect different parts of your application together. For example, we could perform image processing on the photos uploaded by our users. Perhaps we want to provide some sort of face detection or tagging capability, so people can search through all the images they have stored in our service. We could use queues to pass messages to our image-processing service to let it know that new images have been uploaded and are ready for processing. This sort of architecture would allow you to develop and update each part of the service independently.

Azure Storage provides a REST API to work with the containers and data stored in each account. There are independent APIs available to work with each type of data you can store. The Storage REST APIs are accessible from anywhere on the Internet, by any app that can send an HTTP/HTTPS request and receive an HTTP/HTTPS response.

For example, if you wanted to list all the blobs in a container, you would send something like: `GET https://[url-for-service-account]/?comp=list&include=metadata`. This would return an XML block with data specific to the account:
```xml
<?xml version="1.0" encoding="utf-8"?>  
<EnumerationResults AccountName="https://[url-for-service-account]/">  
  <Containers>  
    <Container>  
      <Name>container1</Name>  
      <Url>https://[url-for-service-account]/container1</Url>  
      <Properties>  
        <Last-Modified>Sun, 24 Sep 2018 18:09:03 GMT</Last-Modified>  
        <Etag>0x8CAE7D0C4AF4487</Etag>  
      </Properties>  
      <Metadata>  
        <Color>orange</Color>  
        <ContainerNumber>01</ContainerNumber>  
        <SomeMetadataName>SomeMetadataValue</SomeMetadataName>  
      </Metadata>  
    </Container>  
    <Container>  
      <Name>container2</Name>  
      <Url>https://[url-for-service-account]/container2</Url>  
      <Properties>  
        <Last-Modified>Sun, 24 Sep 2018 17:26:40 GMT</Last-Modified>  
        <Etag>0x8CAE7CAD8C24928</Etag>  
      </Properties>  
      <Metadata>  
        <Color>pink</Color>  
        <ContainerNumber>02</ContainerNumber>  
        <SomeMetadataName>SomeMetadataValue</SomeMetadataName>  
      </Metadata>  
    </Container>  
    <Container>  
      <Name>container3</Name>  
      <Url>https://[url-for-service-account]/container3</Url>  
      <Properties>  
        <Last-Modified>Sun, 24 Sep 2018 17:26:40 GMT</Last-Modified>  
        <Etag>0x8CAE7CAD8EAC0BB</Etag>  
      </Properties>  
      <Metadata>  
        <Color>brown</Color>  
        <ContainerNumber>03</ContainerNumber>  
        <SomeMetadataName>SomeMetadataValue</SomeMetadataName>  
      </Metadata>  
    </Container>  
  </Containers>  
  <NextMarker>container4</NextMarker>  
</EnumerationResults>
```

Client libraries can save a significant amount of work for app developers because the API is tested and it often provides nicer wrappers around the data models sent and received by the REST API.
```csharp
string containerName = "...";
BlobContainerClient container = new BlobContainerClient(connectionString, containerName);

var blobs = container.GetBlobs();
foreach (var blob in blobs)
{
    Console.WriteLine($"{blob.Name} --> Created On: {blob.Properties.CreatedOn:YYYY-MM-dd HH:mm:ss}  Size: {blob.Properties.ContentLength}");
}
```

The client libraries are just thin wrappers over the REST API. They are doing exactly what you would do if you used the web services directly. These libraries are also open source, making them very transparent. For links to the source code for these libraries on GitHub, at the end of this module, see the Additional Resources section.

You'll want to add the Azure.Storage.Blobs package to your .NET or .NET Core applications: `dotnet add package Azure.Storage.Blobs`

To work with data in a storage account, your app will need two pieces of data:
- Access key. Each storage account has two unique access keys that are used to secure the storage account. If your app needs to connect to multiple storage accounts, your app will require an access key for each storage account.
- REST API endpoint. The REST endpoint is a combination of your storage account name, the data type, and a known domain. For example: `https://[name].blob.core.windows.net/`

The simplest way to handle access keys and endpoint URLs within applications is to use storage account connection strings. A connection string provides all needed connectivity information in a single text string. Azure Storage connection strings look similar to the following example, but with the access key and account name of your specific storage account: `"DefaultEndpointsProtocol=https;AccountName={your-storage};AccountKey={your-access-key};EndpointSuffix=core.windows.net"`

Each storage account has two access keys. The reason for this is to allow keys to be rotated (regenerated) periodically as part of security best practice in keeping your storage account secure. This process can be done from the Azure portal or the Azure CLI/PowerShell command line tool. Rotating a key will invalidate the original key value immediately and will revoke access to anyone who obtained the key inappropriately. With support for two keys, you can rotate keys without causing downtime in your applications that use them. Your app can switch to using the alternate access key while the other key is regenerated. If you have multiple apps using this storage account, they should all use the same key to support this technique. Here's the basic idea:
- Update the connection strings in your application code to reference the secondary access key of the storage account.
- Regenerate the primary access key for your storage account using the Azure portal or command line tool.
- Update the connection strings in your code to reference the new primary access key.
- Regenerate the secondary access key in the same manner.

It's highly recommended that you periodically rotate your access keys to ensure they remain private, just like changing your passwords. If you are using the key in a server application, you can use an Azure Key Vault to store the access key for you. Key Vaults include support to synchronize directly to the Storage Account and automatically rotate the keys periodically. Using a Key Vault provides an additional layer of security, so your app never has to work directly with an access key.

Storage accounts offer a separate authentication mechanism called shared access signatures that support expiration and limited permissions for scenarios where you need to grant limited access. You should use this approach when you are allowing other users to read and write data to your storage account.

Add the following command to Program.cs: `using Azure.Storage.Blobs;`

To create and manage containers in your storage account from your .NET application, you use a BlobContainerClient object. To instantiate a BlobContainerClient client object, you must provide the connection string to your storage account and the container name. The container name must be between 3 and 63 characters long and may only contain lowercase letters and the dash (-) character. For this application, we will simply use the name photos.

###### Example code

`appsettings.json`
```json
{
    "ConnectionStrings": {
        "StorageAccount": "DefaultEndpointsProtocol=https;EndpointSuffix=core.windows.net;AccountName=calibali12321;AccountKey=HKu0lPPWYatAhYt/vskcweGt3KtfWy+I+XPTpETefIpyox+ShCsyYdiKYJkdYYsfuMgNHlewHZEBasZ6t9JJ5g=="
    }
}
```

`Program.cs`
```csharp
using System;
using Microsoft.Extensions.Configuration;
using System.IO;
using Azure.Storage.Blobs;

namespace PhotoSharingApp
{
    class Program
    {
        static void Main(string[] args)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddJsonFile("appsettings.json");

            var configuration = builder.Build();

            // Get a connection string to our Azure Storage account.
            var connectionString = configuration.GetConnectionString("StorageAccount");
            
            // Get a reference to the container client object so you can create the "photos" container
            string containerName = "photos";
            BlobContainerClient container = new BlobContainerClient(connectionString, containerName);
            container.CreateIfNotExists(); // this container is created inside the storage account

            // Uploads the image to Blob storage.  If a blb already exists with this name it will be overwritten
            string blobName = "docs-and-friends-selfie-stick";
            string fileName = "docs-and-friends-selfie-stick.png";
            BlobClient blobClient = container.GetBlobClient(blobName);
            blobClient.Upload(fileName, true);

            // List out all the blobs in the container
            var blobs = container.GetBlobs();
            foreach (var blob in blobs)
            {
                Console.WriteLine($"{blob.Name} --> Created On: {blob.Properties.CreatedOn:yyyy-MM-dd HH:mm:ss}  Size: {blob.Properties.ContentLength}");
            }
        }
    }
}
```

`PhotoSharingApp.csproj`
```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp3.1</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Azure.Storage.Blobs" Version="12.9.0" />
    <None Update="appsettings.json">
        <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
    <PackageReference Include="Microsoft.Extensions.Configuration.Json" Version="5.0.0" />
  </ItemGroup>

</Project>
```

##### Secure your Azure Storage account

###### Security features
All data written to Azure Storage is automatically encrypted by Storage Service Encryption (SSE) with a 256-bit Advanced Encryption Standard (AES) cipher, and is FIPS 140-2 compliant. SSE automatically encrypts data when writing it to Azure Storage. When you read data from Azure Storage, Azure Storage decrypts the data before returning it. This process incurs no additional charges and doesn't degrade performance. It can't be disabled. For virtual machines (VMs), Azure lets you encrypt virtual hard disks (VHDs) by using Azure Disk Encryption. This encryption uses BitLocker for Windows images, and it uses dm-crypt for Linux. Azure Key Vault stores the keys automatically to help you control and manage the disk-encryption keys and secrets. So even if someone gets access to the VHD image and downloads it, they can't access the data on the VHD.

Keep your data secure by enabling transport-level security between Azure and the client.

Azure Storage supports cross-domain access through cross-origin resource sharing (CORS). CORS uses HTTP headers so that a web application at one domain can access resources from a server at a different domain. By using CORS, web apps ensure that they load only authorized content from authorized sources. CORS support is an optional flag you can enable on Storage accounts. The flag adds the appropriate headers when you use HTTP GET requests to retrieve resources from the Storage account.

Azure Storage supports Azure Active Directory and role-based access control (RBAC) for both resource management and data operations. For security principals, you can assign RBAC roles that are scoped to the storage account. Use Active Directory to authorize resource management operations, such as configuration. Active Directory is supported for data operations on Blob and Queue storage.

Storage Analytics logs every operation in real time, and you can search the Storage Analytics logs for specific requests. Filter based on the authentication mechanism, the success of the operation, or the resource that was accessed.

###### Storage account keys
Azure Storage accounts can create authorized apps in Active Directory to control access to the data in blobs and queues. This authentication approach is the best solution for apps that use Blob storage or Queue storage.

For other storage models, clients can use a shared key, or shared secret. This authentication option is one of the easiest to use, and it supports blobs, files, queues, and tables. The client embeds the shared key in the HTTP Authorization header of every request, and the Storage account validates the key.

###### Shared access signatures
For untrusted clients, use a shared access signature (SAS). A SAS is a string that contains a security token that can be attached to a URI. Use a SAS to delegate access to storage objects and specify constraints, such as the permissions and the time range of access.

You can use a service-level SAS to allow access to specific resources in a storage account. You'd use this type of SAS, for example, to allow an app to retrieve a list of files in a file system, or to download a file. Use an account-level SAS to allow access to anything that a service-level SAS can allow, plus additional resources and abilities. For example, you can use an account-level SAS to allow the ability to create file systems.

You'd typically use a SAS for a service where users read and write their data to your storage account. Accounts that store user data have two typical designs:
- Clients upload and download data through a front-end proxy service, which performs authentication. This front-end proxy service has the advantage of allowing validation of business rules. But, if the service must handle large amounts of data or high-volume transactions, you might find it complicated or expensive to scale this service to match demand.
- A lightweight service authenticates the client, as needed. Next, it generates a SAS. After receiving the SAS, the client can access storage account resources directly. The SAS defines the client's permissions and access interval. It reduces the need to route all data through the front-end proxy service.

###### Control network access to your storage account

By default, storage accounts accept connections from clients on any network. To limit access to selected networks, you must first change the default action. You can restrict access to specific IP addresses, ranges, or virtual networks. Changing network rules can affect your application's ability to connect to Azure Storage. If you set the default network rule to deny, you'll block all access to the data unless specific network rules grant access. Before you change the default rule to deny access, be sure to use network rules to grant access to any allowed networks.

###### Understand advanced threat protection for Azure Storage

Azure Defender for Storage provides an extra layer of security intelligence that detects unusual and potentially harmful attempts to access or exploit storage accounts. Security alerts are triggered when anomalies in activity occur. These security alerts are integrated with Azure Security Center, and are also sent via email to subscription administrators, with details of suspicious activity and recommendations on how to investigate and remediate threat.

Azure Defender for Storage is currently available for Blob storage, Azure Files, and Azure Data Lake Storage Gen2. Account types that support Azure Defender include general-purpose v2, block blob, and Blob storage accounts. Azure Defender for Storage is available in all public clouds and US government clouds, but not in other sovereign or Azure Government cloud regions. Accounts with hierarchical namespaces enabled for Data Lake Storage support transactions using both the Azure Blob storage APIs and the Data Lake Storage APIs. Azure file shares support transactions over SMB.

When storage activity anomalies occur, you receive an email notification with information about the suspicious security event. Details of the event include:
- Nature of the anomaly
- Storage account name
- Event time
- Storage type
- Potential causes
- Investigation steps
- Remediation steps

Email also includes details about possible causes and recommended actions to investigate and mitigate the potential threat. You can review and manage your current security alerts from Azure Security Center's Security alerts tile. Selecting a specific alert provides details and actions for investigating the current threat and addressing future threats.

###### Explore Azure Data Lake Storage security features

Azure Data Lake Storage Gen2 provides a first-class data lake solution that enables enterprises to consolidate their data. It's built on Azure Blob storage, so it inherits all of the security features we've reviewed in this module.

Along with role-based access control (RBAC), Azure Data Lake Storage Gen2 provides access control lists (ACLs) that are POSIX-compliant, and that restrict access to only authorized users, groups, or service principals. It applies restrictions in a way that's flexible, fine-grained, and manageable. Azure Data Lake Storage Gen2 authenticates through Azure Active Directory OAuth 2.0 bearer tokens. This allows for flexible authentication schemes, including federation with Azure AD Connect and multifactor authentication that provides stronger protection than just passwords.

More significantly, these authentication schemes are integrated into the main analytics services that use the data. These services include Azure Databricks, HDInsight, and Azure Synapse Analytics. Management tools, such as Azure Storage Explorer, are also included. After authentication finishes, permissions are applied at the finest granularity to ensure the right level of authorization for an enterprise's big-data assets.

The Azure Storage end-to-end encryption of data and transport layer protections complete the security shield for an enterprise data lake. The same set of analytics engines and tools can take advantage of these additional layers of protection, resulting in complete protection of your analytics pipelines.

##### Create an Azure Cosmos DB database built to scale

An Azure Cosmos DB account is an Azure resource that acts as an organizational entity for your databases. It connects your usage to your Azure subscription for billing purposes. When creating an account, choose an ID that is meaningful to you; it is how you identify your account. Further, create the account in the Azure region that's closest to your users to minimize latency between the datacenter and your users. You can optionally set up virtual networks and geo-redundancy during account creation, but you can configure those options later.

In Azure Cosmos DB, you provision throughput for your containers to run writes, reads, updates, and deletes. You can provision throughput for an entire database and have it shared among containers within the database. You can also provision throughput dedicated to specific containers.

###### What is a request unit?
Setting provisioned throughput at a container is the most frequently used option. Throughput is reserved only for that container and it's evenly distributed among its physical partitions.

A single request unit, one RU, is equal to the approximate cost of performing a single GET request on a 1-KB document using a document's ID. Performing a GET by using a document's ID is an efficient means for retrieving a document, and thus the cost is small. Creating, replacing, or deleting the same item requires additional processing by the service, and therefore requires more request units.

You can obtain the request unit charge for any operation on your Azure Cosmos DB containers. Multiply the number of consumed RUs of each operation by the estimated number of times each operation (write, read, update, and delete) will be executed per second. By summing the number of consumed RUs for each operation, you will be able to accurately estimate how many RUs to provision. You provision the number of RUs on a per-second basis and you can change the value at any time in increments or decrements of 100 RUs.

While you estimate the number of RUs per second to provision, consider the following factors:
- Item size: As the size of an item increases, the number of RUs consumed to read or write the item also increases.
- Item indexing: By default, each item is automatically indexed. Fewer RUs are consumed if you choose not to index some of your items in a container.
- Item property count: Assuming the default indexing is on all properties, the number of RUs consumed to write an item increases as the item property count increases.
- Indexed properties: An index policy on each container determines which properties are indexed by default. To reduce the RU consumption for write operations, limit the number of indexed properties.
- Data consistency: The strong and bounded staleness consistency levels consume approximately two times more RUs on read operations when compared to that of other relaxed consistency levels.
- Query patterns: The complexity of a query affects how many RUs are consumed for an operation. Factors that affect the cost of query operations include:
    - The number of query results
    - The number of predicates
    - The nature of the predicates
    - The number of user-defined functions
    - The size of the source data
    - The size of the result set
    - Projections
    
    Azure Cosmos DB guarantees that the same query on the same data always costs the same number of RUs on repeated executions.
- Script usage: As with queries, stored procedures and triggers consume RUs based on the complexity of their operations. As you develop your application, inspect the request charge header to better understand how much RU capacity each operation consumes.

If you attempt to use throughput higher than the one provisioned, your request will be rate-limited. The .NET SDK will automatically retry your request after waiting the amount of time specified in the retry-after header.

When you create an account, you can provision a minimum of 400 RU/s, or a maximum of 250,000 RU/s in the portal. If you need even more throughput, fill out a ticket in the Azure portal.

###### Choose a partition key in Azure Cosmos DB
A partitioning strategy enables you to add more partitions to your database when you need them. This scaling strategy is called scale out or horizontal scaling. A partition key defines the partition strategy, it's set when you create a container and can't be changed. Selecting the right partition key is an important decision to make early in your development process. A partition key is the value by which Azure organizes your data into logical divisions. It should aim to evenly distribute operations across the database to avoid hot partitions. The storage space for the data associated with each partition key can't exceed 20 GB, which is the size of one physical partition in Azure Cosmos DB.

When you're trying to determine the right partition key and the solution isn't obvious, here are a few tips to keep in mind.
- Don't be afraid of choosing a partition key that has a large number of values. The more values your partition key has, the more scalability you have.
- To determine the best partition key for a read-heavy workload, review the top three to five queries you plan on using. The value most frequently included in the WHERE clause is a good candidate for the partition key.
- For write-heavy workloads, you'll need to understand the transactional needs of your workload, because the partition key is the scope of multi-document transactions.

For each Azure Cosmos DB container, you should specify a partition key that satisfies the following core properties:
- Have a high cardinality. This option allows data to distribute evenly across all physical partitions.
- Evenly distribute requests. Remember the total number of RU/s is evenly divided across all physical partitions.
- Evenly distribute storage. Each partition can grow up to 20 GB in size.

Example code to build a Cosmos DB database and container:
```csharp
using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Azure.Cosmos;

namespace myApp
{
    public class Program
    {
        private static readonly string _endpointUri = "YOUR_URI";
        private static readonly string _primaryKey = "YOUR_KEY";
        public static async Task Main(string[] args)
        {         
            using (CosmosClient client = new CosmosClient(_endpointUri, _primaryKey))
            {        
                DatabaseResponse databaseResponse = await client.CreateDatabaseIfNotExistsAsync("Products");
                Database targetDatabase = databaseResponse.Database;
                await Console.Out.WriteLineAsync($"Database Id:\t{targetDatabase.Id}");
                IndexingPolicy indexingPolicy = new IndexingPolicy
                {
                    IndexingMode = IndexingMode.Consistent,
                    Automatic = true,
                    IncludedPaths =
                    {
                        new IncludedPath
                        {
                            Path = "/*"
                        }
                    }
                };
                var containerProperties = new ContainerProperties("Clothing", "/productId")
                {
                    IndexingPolicy = indexingPolicy
                };
                var containerResponse = await targetDatabase.CreateContainerIfNotExistsAsync(containerProperties, 10000);
                var customContainer = containerResponse.Container;
                await Console.Out.WriteLineAsync($"Custom Container Id:\t{customContainer.Id}");
            }
        }
    }
}
```

##### Choose the appropriate API for Azure Cosmos DB
At the lowest level, Azure Cosmos DB stores data in atom-record-sequence (ARS) format. The data is then abstracted and projected as an API, which you specify when you are creating your database.

Decision criteria

| Criteria Core |  (SQL) | MongoDB | Cassandra | Azure Table | Gremlin |
| ------------- | ------ | ------- | --------- | ----------- | ------- |
| New projects being created from scratch | ✔ | | | | |		
Existing MongoDB, Cassandra, Azure Table, or Gremlin data | | ✔ | ✔| ✔ | ✔ |
Analysis of the relationships between data | | | | | ✔ |
All other scenarios	| ✔ | | | | |

1. Does the schema change a lot? A traditional document database is a good fit in these scenarios, making Core (SQL) a good choice.
2. Is there important data about the relationships between items in the database? Relationships that require metadata to be stored for them are best represented in a graph database.
3. Does the data consist of simple key-value pairs? Before Azure Cosmos DB existed, Redis or the Table API might have been a good fit for this kind of data; however, Core (SQL) API is now the better choice, as it offers a richer query experience, with improved indexing over the Table API.

###### Core
Main difference in choice between Core (SQL) and MongoDB is the preferred language choice of the existing project, since Core (SQL) provides you with a view of your data that resembles a traditional NoSQL document store.

###### Gremlin
The data store needs to be able to assign weight values to the relationships between products. For example, you might store a count of the number of times that a relationship occurs. With that in mind, each time that a person buys "Product A" and "Product B", the relationship link between these two products needs to be incremented. This relationship counter is meta-data that needs to be stored in a database.

###### MongoDB
The operations team has semi-structured data that needs the flexibility to store many different supplier order formats. Based on your research, you have determined that both Core (SQL) and MongoDB are good options. However, your operations team wants to reduce downtime during their app migration, so your best bet would be to find a way to allow your operations team to continue to use the MongoDB wire protocol.

###### Cassandra
Your web analytics application is based on Cassandra, and your web analytics team has valuable experience with CQL.

###### Azure Table
The IoT data consists of key-value pairs with no relationship information, and the existing dataset is currently deployed in Azure Table Storage.

##### Insert and query data in your Azure Cosmos DB database

Azure Cosmos DB provides the Data Explorer tool in the Azure portal that you can use to perform all these operations: adding data, modifying data, and creating and running stored procedures. The Data Explorer can be used in the Azure portal or it can be undocked and used in a standalone browser window, providing additional space to work with your data.

Every SQL query consists of a `SELECT` clause and optional `FROM` and `WHERE` clauses. You can also add other clauses like `ORDER BY` and `JOIN` to get the information you need.

A SQL query has the following format:
```sql
SELECT <select_list>
    [FROM <optional_from_specification>]
    [WHERE <optional_filter_condition>]
    [ORDER BY <optional_sort_specification>]
    [JOIN <optional_join_specification>]
```

A value of `SELECT *` indicates that the entire JSON document is returned. Or, you can limit the output to include only certain properties by including a list of properties in the SELECT clause.

The `FROM` clause specifies the data source upon which the query operates. You can make the whole container the source of the query or you can specify a subset of the container instead.

A container can be aliased with the AS keyword: `SELECT p.id FROM Products AS p`. ou can also omit the AS keyword as shown in this equivalent statement: `SELECT p.id FROM Products p`.

The `WHERE` clause specifies the conditions that the source JSON documents must satisfy to be included as part of the result.

The `ORDER BY` clause enables you to order the results in ascending or descending order.

The `JOIN` clause lets you perform inner joins with the document and the document subroots. In the following query, the product IDs are returned for each product that has a shipping method. If you added a third product that didn't have a shipping property, the result would be the same because the third item would be excluded for not having a shipping property.
```sql
SELECT p.productId
  FROM Products p
  JOIN p.shipping
```

Geospatial queries enable you to perform spatial queries using GeoJSON Points. Using the coordinates in the database, you can calculate the distance between two points and determine whether a Point, Polygon, or LineString is within another Point, Polygon, or LineString.

For product catalog data, this would enable your users to enter their location information and determine whether there were any stores within a 50-mile radius that have the item they're looking for.

Stored procedures are the only way to ensure ACID (Atomicity, Consistency, Isolation, Durability) transactions because they are run on the server, and are thus referred to as server-side programming. UDFs are also stored on the server and are used during queries to perform computational logic on values or documents within the query. Stored procedures perform complex transactions on documents and properties. Stored procedures are written in JavaScript and are stored in a container on Azure Cosmos DB.

UDFs are used to extend the Azure Cosmos DB SQL query language grammar and implement custom business logic, such as calculations on properties and documents. UDFs can be called only from inside queries and, unlike stored procedures, they do not have access to the context object, so they cannot read or write documents. The following sample creates a UDF to calculate tax on a product in a fictitious company based the product cost:
```javascript
function producttax(price) {
    if (price == undefined) 
        throw 'no input';

    var amount = parseFloat(price);

    if (amount < 1000) 
        return amount * 0.1;
    else if (amount < 10000) 
        return amount * 0.2;
    else
        return amount * 0.4;
}
```
The SQL query used to execute a UDF looks like:
```sql
SELECT c.id, c.productId, c.price, udf.producttax(c.price) AS producttax FROM c
```

#####  Store and access graph data in Azure Cosmos DB with the Graph API

In order to understand graph databases, you first need to know what we mean by a graph. A graph is a structure that's composed of vertices and edges. Both vertices and edges can have an arbitrary number of properties.

| Component | Description |
| --------- | ----------- |
| Vertices or Nodes | Vertices represent objects. For example: a person, a place, or a product. Nodes can be assigned roles or types by using one or more labels in order to group or categorize them. |
| Edges or Relationships | Edges denote relationships between vertices. For example: a person might know another person, or have visited a place. You can often determine relationships for the graph model by identifying actions or verbs in your domain. |
| Properties | Properties express information about the vertices and edges. For example, vertices' properties might include the name and age of a person. Edge properties might include a time stamp of a purchase or a hierarchical affiliation between coworkers. |

Because relationships are treated as data rather than schema structure, there are many scenarios where graph databases are useful, as they let you model and store efficiently in a non-tabular format.

Strengths and weaknesses of graph databases fall under the following criteria:
- Performance: With traditional relational databases, the performance of relationship queries decreases as the number of relationships increase. With graph databases, the performance stays constant even as the data and complexity continues to grow.
- Flexibility: In graph databases, it's easy to add to an existing structure without affecting functionality. This flexibility allows the graph database model to dictate change, rather than being forced to adapt to a tabular way of seeing your data in a standard relational database.
- Deficiencies: Graph databases are not as efficient at processing high volumes of transactions, nor are they as effective at handling queries that traverse an entire database.

Graph databases do not create better relationships; instead, they provide rapid data retrieval for connected data. This increases the need for efficient data design, because any performance gains from graph searches can be reduced by failing to model the relationships between your nodes efficiently.

When you have identified your nodes and relationships, your data model begins to take shape. For example:
| Node | Relationship | Node |
| ---- | ------------ | ---- |
| A customer | likes | a product |
| A product | belongs to | a category |
| A customer | places | an order |
| A product | belongs to | an order |
| An employee | submits	| an invoice |
| An order | belongs to | an invoice |

Properties are name-value pairs that you can define for nodes or relationships, which enable you to store relevant data about the node or relationship you're describing. To determine what kind of properties you can use, you could ask some relevant questions about the data you are capturing:
- When did the customer place the order?
- What was the product price?
- When did the employee submit the invoice?
- Which invoice does the order belong to?

The flexible nature of a graph database enables you to grow and change your model over time, and to add or remove relationships, nodes, and properties.

The Azure Cosmos DB Gremlin API is used to store and operate on graph data. Gremlin API supports modeling graph data and provides APIs to traverse through the graph data. Azure Cosmos DB is a fully managed graph database that offers global distribution, elastic scaling of storage and throughput, automatic indexing and query, tunable consistency levels, and support for the TinkerPop standard. The Gremlin Graph Traversal Language, or Gremlin is a high-performance language used to traverse and query data from graph databases. It's described as a functional, data-flow language that enables users to express complex traversals for their application's property graph.

Example .NET core app code:
```json
{
  "AzureConfig": {
   "HostName" : "wss://calibali12321.gremlin.cosmos.azure.com:443/",
   "Port" : "443",
   "AuthKey" : "tayzZ1Q1t350YkmqhYtW6S1aqJls4pB2s10EjtxMJj9MgLpjh32M400F7QfbN80fyGefRcVvRtGL0r8j0biLpQ==",
   "Database" : "sample-database",
   "Collection" : "sample-graph"
   }
}
```

```csharp
using System;
using System.Threading.Tasks;
using Gremlin.Net.Driver;
using Gremlin.Net.Driver.Exceptions;
using Gremlin.Net.Structure.IO.GraphSON;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.Configuration.Json;
using Newtonsoft.Json;

namespace GremlinApp
{
    class Program
    {
        static void Main(string[] args)
        {
            try
            {
                if (args.Length!=1)
                {
                    Console.WriteLine("Please enter a Gremlin/Graph Query.");
                }
                else
                {
                    var azureConfig = new ConfigurationBuilder()
                        .SetBasePath(Environment.CurrentDirectory)
                        .AddJsonFile("appsettings.json", optional: false, reloadOnChange: false)
                        .Build()
                        .GetSection("AzureConfig");
                    var hostname = azureConfig["HostName"];
                    var port = Convert.ToInt32(azureConfig["Port"]);
                    var authKey = azureConfig["AuthKey"];
                    var database = azureConfig["Database"];
                    var collection = azureConfig["Collection"];
                    var gremlinServer = new GremlinServer(
                        hostname, port, enableSsl: true,
                        username: $"/dbs/" + database + "/colls/" + collection,
                        password: authKey);
                    using (var gremlinClient = new GremlinClient(gremlinServer, new GraphSON2Reader(), new GraphSON2Writer(), GremlinClient.GraphSON2MimeType))
                    {
                        var resultSet = AzureAsync(gremlinClient, args[0]);
                        Console.WriteLine("\n{{\"Returned\": \"{0}\"}}", resultSet.Result.Count);
                        foreach (var result in resultSet.Result)
                        {
                            string jsonOutput = JsonConvert.SerializeObject(result);
                            Console.WriteLine("{0}",jsonOutput);
                        }
                    }
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine("EXCEPTION: {0}", ex.Message);
            }
        }
        private static Task<ResultSet<dynamic>> AzureAsync(GremlinClient gremlinClient, string query)
        {
            try
            {
                return gremlinClient.SubmitAsync<dynamic>(query);
            }
            catch (ResponseException ex)
            {
                Console.WriteLine("EXCEPTION: {0}", ex.StatusCode);
                throw;
            }
        }
    }
}
```

To retrieve all the vertices/nodes in your graph, run the following command: `dotnet run "g.V()"`. To retrieve all the edges/relationships in your graph, run the following command: `dotnet run "g.E()"`

More example commands:
- `dotnet run "g.V().count()"`
- `dotnet run "g.V().hasLabel('Category')"`
- `dotnet run "g.V().hasLabel('Product').has('id','p1')"`
- `dotnet run "g.V().hasLabel('Category').values('name')"`
- `dotnet run "g.V().hasLabel('Product').values('name','price')"`
- `dotnet run "g.V().hasLabel('Product').order().by('name', incr).values('name','price')"`
- `dotnet run "g.V().hasLabel('Product').order().by('price', decr).values('name','price')"`
- `dotnet run "g.V('p1').property('price', 15.99)"`

##### Store and Access NoSQL Data with Azure Cosmos DB and the Table API

Storage tables are an example of a NoSQL database. Such databases don't impose a strict schema on each table like a SQL database does. Instead, each entity in the table can have a different set of properties. It's up to you to ensure that these properties are organized, and to ensure that apps that query the data can work with results that may have different values. A primary advantage of this semi-structured approach to data is that the database can evolve more quickly to meet changing business requirements. An entity, in a NoSQL table, is the equivalent of a row in a relational database table.

With geo-redundant storage, data in the secondary region is not readable unless an outage causes Azure to fail-over to that region. Under normal circumstances, all clients must connect to the primary region to access data. You can also opt for the 'Read-access geo-redundant storage' replication type when configuring the storage account.

Azure Cosmos DB includes the Tables API, which means that if you move your data from Azure Storage tables into Azure Cosmos DB, you don't have to rewrite your apps. Instead, you just change their connection strings.

There are some differences in behavior between Azure Storage tables and Azure Cosmos DB tables to remember if you are considering a migration. For example:
- You are charged for the capacity of an Azure Cosmos DB table as soon as it is created, even if that capacity isn't used. This charging structure is because Azure Cosmos DB uses a reserved-capacity model to ensure that clients can read data within 10 ms. In Azure Storage tables, you are only charged for used capacity, but read access is only guaranteed within 10 seconds.
- Query results from Azure Cosmos DB are not sorted in order of partition key and row key as they are from Storage tables.
- Row keys in Azure Cosmos DB are limited to 255 bytes.
- Batch operations are limited to 2 MBs.
- Cross-Origin Resource Sharing (CORS) is supported by Azure Cosmos DB.
- Table names are case-sensitive in Azure Cosmos DB. They are not case-sensitive in Storage tables.

While these differences are small, you should take care to review your apps to ensure that a migration does not cause unexpected problems.

HOW TO CHOOSE A STORAGE LOCATION
| Priority | Azure Storage tables | Azure Cosmos DB tables |
| -------- | -------------------- | ---------------------- |
| Latency | Responses are fast, but there is no guaranteed response time. | < 10 ms for reads, < 15 ms for writes. |
| Throughput | Maximum 20,000 operations/sec | 	No upper limit on throughput. Over 10 million operations/sec/table. |
| Global distribution | Single region for writes. A secondary read-only region is possible with read-access geo-redundant replication. | Replication of data for read and write to more than 30 regions. | 
| Indexes | A single primary key on the partition key and the row key. No other indexes. | Indexes are created automatically on all properties. | 
| Data consistency | Strong in the primary region. If you are using read-access geo-redundant replication, it may take time for changes to reach the secondary region. | You can choose from five different consistency levels depending on your needs for availability, latency, throughput, and consistency. | 
| Pricing | Optimized for storage. | Optimized for throughput. | 
| SLAs | 99.99% availability. | 99.99% availability for single region and relaxed consistency databases. 99.999% availability for multi-region databases. | 

How to migrate an app to Azure Cosmos DB:
- Azure Cosmos DB Data Migration Tool. This open source tool is built specifically to import data into Azure Cosmos DB from many different sources, including tables in Azure Storage, SQL databases, MongoDB, text files in JSON and CSV formats, HBase, and other databases. The tool has both a command-line version and a GUI version. You supply the connection strings for the data source and the Azure Cosmos DB target, and you can filter the data before migration.
- AzCopy. This command-line only tool is designed to enable developers to copy data to and from Azure Storage accounts. The process has two stages:
    - Export the data from the source to a local file.
    - Import from that local file to a database in Cosmos, specifying the destination database by using its URL and access key.

##### Build a .NET Core app for Azure Cosmos DB in Visual Studio Code

See `Demo's/NetCoreAppForCosmosDB/Program.cs` for code.

LINQ is a .NET programming model that expresses computations as queries on streams of objects. You can create an IQueryable object that directly queries Azure Cosmos DB, which translates the LINQ query into an Azure Cosmos DB query. The query is then passed to the Azure Cosmos DB server to retrieve a set of results in JSON format. The returned results are deserialized into a stream of .NET objects on the client side.

##### Optimize the performance of Azure Cosmos DB by using partitioning and indexing strategies

n RU is the amount of CPU, disk I/O, and memory required to read 1 KB of data in 1 second.

The number of RUs that a specific operation uses depends on the following factors:
- How the data is distributed across the physical resources in Azure
- Volume of data that's read and written
- Whether the operation is a read or a write
- Number of fields in your database that are indexed, and the indexing mode
- Complexity of the operation for queries
- Data consistency for geographically replicated collections

To operate an Azure Cosmos DB database efficiently, you need to:
- Configure enough throughput to meet performance demands.
- Minimize unused excess throughput to keep your costs down.
- Regularly review metrics to maximize the efficiency of your provisioned throughput.

RUs are billed on an hourly basis, whether you consume them. The amount that you're charged for your Azure Cosmos DB collection is fixed. It's based only on the configured capacity in RUs.

Azure Cosmos DB concepts consist of:
- Resources. An Azure Cosmos DB account is a container for one or more databases. An Azure Cosmos DB database is a container for one or more collections. A collection contains documents. A document is an unstructured set of key/value pairs, read and written in JSON format.
- Partitioning. Partitioning is the distribution and grouping of your data across the underlying resources. Documents are grouped in a partition based on the value of the partition key. You specify the partition key when you create the collection. Querying documents from within the same partition is less expensive than querying across partitions.
- Indexing. An index is a catalog of document properties and their values. It includes links to documents that contain properties equal to each property value. Indexing makes searching a collection more efficient. But the search efficiency is balanced with the resources required to insert or change a document. When a document is inserted or changed, Azure Cosmos DB has to update the index. The optimal indexing strategy for your collection depends on your workload. Unlike partitioning, you can change indexing at runtime.

The HTTP 429 error code indicates that there are too many requests. Future requests will be rate-limited. When you see HTTP 429 responses from Azure Cosmos DB, that means you've exceeded the allocated capacity. If the demand on one or more of your collections exceeds its allocated capacity, you have the following options to fix an overloaded collection:
- Increase the overloaded collection's capacity.
- Reduce the demand on your collection.
- Increase the efficiency of the operations that overload your collection.

To look at the performance costs of detailed operations, like reading a document directly, use the ExerciseCosmosDB utility.

Reading a document directly from your Azure Cosmos DB collection by using its `id` and `partition key` values is the least expensive operation.

Remember that data in an Azure Cosmos DB is stored in collections. Collections are distributed across partitions based on the value of a collection's partition key.

The partition key is a document property. Documents with the same partition key value are always located on the same logical partition. A partition supports a fixed maximum amount of storage and Request Units (RUs). When the capacity of a logical partition gets close to the maximum storage, Azure Cosmos DB allocates another physical partition. Azure Cosmos DB seamlessly splits the logical partitions, the groups of documents with the same partition key value, among the physical partitions.

Designing a partitioning strategy requires you to understand your data and its operational workloads. As you consider your design, we recommend that you consider the following requirements.

1. Estimate the scale of your data needs
    - What's the approximate size of your documents, or range of sizes?
    - What's the required throughput in number of reads per second and writes per second?
    - What's the volume of documents being queried?
2. Understand the workload
- Do you have a read-heavy or write-heavy workload, or both?
- If it's read-heavy, what are the top five queries?
- If it's write-heavy, do you need transactions?
3. Propose some partition key options
- Does the key choice have a large number of possible values or large cardinality?
- Do the values have a consistent spread across the data?
- Are some values accessed more than others?
- For read-heavy workloads, can the query be within a single partition?
- For write-heavy transactional workloads, can the transaction be within a single partition?

What causes a hot partition in a distributed NoSQL database collection? A partition key that doesn't distribute requests evenly over storage and time.

What's the best way to maximize the efficiency of an individual database query? 
Design a partition key strategy so that your most frequent queries don't cross partitions.

An index is extra information that sits alongside a collection to make querying more efficient. Queries use the index to locate documents.

The index is updated every time a document is updated or added to a collection. That update adds to the RUs used for each write operation.

The right indexing strategy depends on the patterns of access to your collections. Read-intensive workloads call for a different indexing strategy from write-intensive ones. Unlike the partitioning configuration, you can change the Azure Cosmos DB indexing configuration after a collection is provisioned. When your database is up and running, measure the performance of your index configuration and tune it.

By default, all document properties in Azure Cosmos DB are indexed.

The three indexing modes you can use with Azure Cosmos DB are:
- Consistent: The index is updated synchronously every time a new document is written to the collection. New queries on the collection use the updated index immediately. Query results are consistent with the updated documents in the collection.
- Lazy: The index is updated at a lower priority. The reads and writes from the collection take a higher priority. In lazy mode, writes are cheaper because the index isn't updated immediately. When the index is fully updated depends on the demands on the collection. Query results don't include the updated documents until the index is consistent with the collection.
- None: No index is created. Queries are expensive on collections that aren't indexed. If you're using your Azure Cosmos DB collection to read records directly rather than querying the collection, it's possible to avoid the overhead of indexing.

If you add properties to the index configuration, Azure Cosmos DB parses your collections, and updates the index. The index uses more storage. If you reduce the number of properties from the index, Azure Cosmos DB removes this information from your index. The index uses less storage.

To include a document property in the index, you add its path to the index configuration. For example:
- To include the Customer Email property in the index, add Customer/Email/? to the includedPaths array.
- To exclude a property, add it to the excludedPaths array.

Example configuration:
```json
{
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/Item/id/?"
        },
        {
          "path": "/Customer/email/?"
        },
        {
          "path": "/OrderTime/?"
        },
        {
          "path": "/Merchant/?"
        },
        {
          "path": "/OrderStatus/?"
        }
    ],
    "excludedPaths": [
        {
            "path": "/"
        }
    ]
}
```

As indexing complexity goes up, the write consumption goes up, and the read consumption goes down. The indexing strategy that you choose depends on your data and the workloads that it supports.

##### Distribute your data globally with Azure Cosmos DB

Global distribution enables you to replicate data from one region into multiple Azure regions. You can add or remove regions in which your database is replicated at any time, and Azure Cosmos DB ensures that when you add an additional region, your data is available for operations within 30 minutes, assuming your data is 100 TBs or less.

There are two common scenarios for replicating data in two or more regions:
- Delivering low-latency data access to end users no matter where they are located around the globe
- Adding regional resiliency for business continuity and disaster recovery (BCDR)

When a database is replicated, the throughput and storage are replicated equally as well.

When creating the Azure Cosmos DB resource, the multi-region writes can only be enabled during account configuration.

Multi-master support is an option that can be enabled on new Azure Cosmos DB accounts. Once the account is replicated in multiple regions, each region is a master region that equally participates in a write-anywhere model, also known as an active-active pattern.

The benefits of multi-master support are:
- Single-digit write latency – Multi-master accounts have an improved write latency of <10 ms for 99% of writes, up from <15 ms for non-multi-master accounts.
- 99.999% read-write availability - The write availability multi-master accounts increases to 99.999%, up from the 99.99% for non-multi-master accounts.
- Unlimited write scalability and throughput – With multi-master accounts, you can write to every region, providing unlimited write scalability and throughput to support billions of devices.
- Built-in conflict resolution – Multi-master accounts have three methods for resolving conflicts to ensure global data integrity and consistency.

With the addition of multi-master support comes the possibility of encountering conflicts for writes to different regions. Conflicts are rare in Azure Cosmos DB and can only occur when an item is simultaneously changed in multiple regions, before propagation between the regions has happened. There are three conflict resolution modes offered by Azure Cosmos DB.
- Last-Writer-Wins (LWW), in which conflicts are resolved based on the value of a user-defined integer property in the document. By default _ts is used to determine the last written document. Last-Writer-Wins is the default conflict handling mechanism.
- Custom - User-defined function, in which you can fully control conflict resolution by registering a User-defined function to the collection. A User-defined function is a special type of stored procedure with a specific signature. If the User-defined function fails or does not exist, Azure Cosmos DB will add all conflicts into the read-only conflicts feed they can be processed asynchronously.
- Custom - Async, in which Azure Cosmos DB excludes all conflicts from being committed and registers them in the read-only conflicts feed for deferred resolution by the user’s application. The application can perform conflict resolution asynchronously and use any logic or refer to any external source, application, or service to resolve the conflict.

Once you've replicated your data in multiple regions, you can take advantage of the automated failover solutions Azure Cosmos DB provides. Automated failover is a feature that comes into play when there's a disaster or other event that takes one of your read or write regions offline, and it redirects requests from the offline region to the next most prioritized region.

If you enable multi-region writes when creating the Azure Cosmos DB account, every region is both read and write. So if a regional failure happens the SDK will redirect to the next closest region and this region supports both read and write requests. So the concept of automatic and manual failover is applicable to single-region write account only.

What happens if a read region has an outage?
- Azure Cosmos DB accounts with a read region in one of the affected regions are automatically disconnected from their write region and marked offline. The Azure Cosmos DB SDKs implement a regional discovery protocol that allows them to automatically detect when a region is available and redirect read calls to the next available region in the preferred region list. If none of the regions in the preferred region list is available, calls automatically fall back to the current write region.

What happens if a write region has an outage?
- If the affected region is the current write region and automatic failover is enabled for the Azure Cosmos DB account, then the region is automatically marked as offline. Then, an alternative region is promoted as the write region for the affected Azure Cosmos DB account.

You can configure the default consistency level on your Azure Cosmos DB account (and later override the consistency on a specific read request) in the Azure portal. Internally, the default consistency level applies to data within the partition sets, which may span regions.

Strong consistency
- Strong consistency offers a linearizability guarantee with the reads guaranteed to return the most recent version of an item.
- Strong consistency guarantees that a write is only visible after it is committed durably by the majority quorum of replicas. A write is either synchronously committed durably by both the primary and the quorum of secondaries, or it is aborted. A read is always acknowledged by the majority read quorum, a client can never see an uncommitted or partial write and is always guaranteed to read the latest acknowledged write.
- You can have an Azure Cosmos account with strong consistency and multiple write regions. However, the benefits such as low write latency, high write availability that are available to multiple write regions are not applicable to Cosmos accounts configured with strong consistency, because of synchronous replication across regions. For more information, see Consistency and Data Durability in Azure Cosmos DB article.
-The cost of a read operation (in terms of request units consumed) with strong consistency is higher than session and eventual, but the same as bounded staleness.

Bounded staleness consistency
-Bounded staleness consistency guarantees that the reads may lag behind writes by at most K versions or prefixes of an item or t time-interval.
Therefore, when choosing bounded staleness, the "staleness" can be configured in two ways: number of versions K of the item by which the reads lag behind the writes, and the time interval t.
- Bounded staleness offers total global order except within the "staleness window." The monotonic read guarantees exist within a region both inside and outside the "staleness window."
- Bounded staleness provides a stronger consistency guarantee than session, consistent-prefix, or eventual consistency. For globally distributed applications, we recommend you use bounded staleness for scenarios where you would like to have strong consistency but also want 99.99% availability and low latency.
- Azure Cosmos DB accounts that are configured with bounded staleness consistency can associate any number of Azure regions with their Azure Cosmos DB account.
- The cost of a read operation (in terms of RUs consumed) with bounded staleness is higher than session and eventual consistency, but the same as strong consistency.

Session consistency
- Unlike the global consistency models offered by strong and bounded staleness consistency levels, session consistency is scoped to a client session.
- Session consistency is ideal for all scenarios where a device or user session is involved since it guarantees monotonic reads, monotonic writes, and read your own writes (RYW) guarantees.
- Session consistency provides predictable consistency for a session, and maximum read throughput while offering the lowest latency writes and reads.
- Azure Cosmos DB accounts that are configured with session consistency can associate any number of Azure regions with their Azure Cosmos DB account.
- The cost of a read operation (in terms of RUs consumed) with session consistency level is less than strong and bounded staleness, but more than eventual consistency.

Consistent prefix consistency
- Consistent prefix guarantees that in absence of any further writes, the replicas within the group eventually converge.
- Consistent prefix guarantees that reads never see out of order writes. If writes were performed in the order A, B, C, then a client sees either A, A,B, or A,B,C, but never out of order like A,C or B,A,C.
- Azure Cosmos DB accounts that are configured with consistent prefix consistency can associate any number of Azure regions with their Azure Cosmos DB account.

Eventual consistency
- Eventual consistency guarantees that in absence of any further writes, the replicas within the group eventually converge.
- Eventual consistency is the weakest form of consistency where a client may get the values that are older than the ones it had seen before.
- Eventual consistency provides the weakest read consistency but offers the lowest latency for both reads and writes.
- Azure Cosmos DB accounts that are configured with eventual consistency can associate any number of Azure regions with their Azure Cosmos DB account.
- The cost of a read operation (in terms of RUs consumed) with the eventual consistency level is the lowest of all the Azure Cosmos DB consistency levels.

### 2.2 Develop solutions that use blob storage {#2.2}

#### Pluralsight notes

##### Developing Solutions with Blob Storage

###### Understanding Azure Blob Storage

Azure Blob Storage is a massively scalable object storage for the cloud.
- Text files
	- text
	- log
- Binary files
	- images/videos
	- virtual disks

Accessible via Rest API over HTTP/HTTPs
	- Azure Portal
	- Azure Powershell
	- Azure CLI
	- Azure storage client library
		- .Net, JAVA, Python, Node.js

To use blob storage, you need to create a storage account. Name must be unique across all Azure Storage Accounts (3-24 characters, lower case, numbers). Here-in you make blob containers. Here-in you store your blobs. To access the files, use the follow URL for you requests: `https://<storageAccountName>.blob.core.windows.net/<blobContainerName>/<blobName>`. You can not make subfolders in blob containers, but you can make virtual directories by naming your blob somehting like: `design/logo.png`.

Performance of disks:
- Standard: magnetic disks
- Premium: SSD

Authorize requests:
- The connection string to your DB contains the Share Key (Storage Account Key).
- You can also use Shared Access Signature (SAS) for granular control over who can see what items in storage
- You can also use Azure Active Directory to control who has access to the storage
- Set Anonymous Public Read Access at the container level for the whole container, or just a blob.

Different Blob Types:
- Block Blob: for uploading a whole binary file (photos, videos, etc).
- Append Blob: for files like .log.
- Page Blob: for random read/write access to access a specific position in the blob. Useful for .vhdx.

Storage account kinds:
- Blob
- File
- Queue
- Table

Within StorageV2 (General Purpose V2) The Permium Performance storage account (SSD) does not support File, Queue and Table. It only supports Blob: page kind. This means it is only useful for storage hard disks. If you want Block and Append Blob on premium SSD, you should choose the 'BlockBlobStorage' account. For premium File-storage, you need the FileStorage storage account.

StorageV2 is great for high throughput and is recommended for most usecases. BlockBlobStorage is great for high frequency with small data.

Replication Strategy;
- Locally Redundant Storage (LRS): 3 copies within 1 datacenter (zone)
- Zone-redundant storage (ZRS): 3 copies within 3 zones, in 1 region.
- Geo-redundant region (GRS): same as LRS but another 3 copies in anohter zone within a region
- Geo-zone-redundant storage (GZRS): ZRS, but with another 3 copies in another zone within a region

After a fail-over, the secondary region becomes your primary region. The failover however, takes about an hour. However, you can enable read-access replication strategy to the secondary regions (RA-GRS, RA-GZRS). You can access via `https://<storageAccountName>-secondary.blob.core.windows.net/<blobContainerName>/<blobName>`

For the standard accounts:
- StorageV2 supports all replication strategies. 
- The Legacy accounts (storageV1 en BlobStorage) support LRS, GRS, RA-GRS.

The premium accounts support only LRS and ZRS:
- BlockBlobStorage
- FileStorage

ZRS is only available in availability zones  

###### Interacting with data using the Azure SDK for .NET

Azure SDK for .NET uses client libraries as NuGet Packages. `Azure.<service-category>.<service-name>`.

The Azure.Storage.Blobs NuGet package contains 3 important classes:
- BlobSercviceClient -> sotrage account
- BlobContainerClient -> blob containers
- BlobClient -> blobs

###### Setting properties and Metadata

A blob container has system properties:
- ETag: entity identifier. this changes every time a blob container gets updated
- LastModified: when the container is modified

A blob has properties:
- Etag
- Last modified
- Content-type
- Content-Length
- x-ms-blob-type

Both Blob Containers and Blobs can have user-defined metadata that are key-value pairs

Properties and UD-metadata are set through http-headers.

Look at properties and metadata.
- In the Azure Portal
- In the HTTP response headers of a Blob

When you set the properties with the SDK, the existing properties are overwritten. That's why it's good to first get the properties of the blob, and then set the properties.

###### Implementing data archiving and retention

Access tiers for block blobs:
- Hot: frequently accessed data
- Cool: infrequently accessed data stored for at least 30 days
- Archive: data can be archived for at least 180 days.

Hot < ---- Cool ---- > Archive
low < --- access and transaction costs --- > high
high < --- storage per GB --- > low

Storage account default access tier: Hot (but can be set to Cool). The block blob access tier is inferred from the storage account, but can be set to Hot, Cool and Archive. Archive can only be set at the block level, since it is an offline access tier. That means that you cannot access a Blob set to Archive. You can read the properties and metadata, but you can only read the content if you 'rehydrate' the archived blob (set to cool or hot). This may take several hours.

Only StorageV2 and Legacy BlobStorage support access tiers.

You can manage the blob lifecycle with rules. For example, you can move blobs or containers from Hot to the Cool access tier if the 'last modified' property is older than x days.

Turn on soft deletes (data marked as deleted). Soft delete does not only work for blobs, but also for snapshots and versions.

You can create a lease for a blob. This means that you can only update or delete a blob using the Lease ID. You can specify different tags for blobs

###### Move Items in Blob Storage

First copy, then delete. Use Azure CLI, AzCopy, .NET client library

When you use the azcopy command, you copy directly in the cloud.

#### Microsoft Learn

##### Store application data with Azure Blob Storage

Blobs are usually not appropriate for structured data that needs to be queried frequently. They have higher latency than memory and local disk and don't have the indexing features that make databases efficient at running queries.

In addition to public access, Azure has a shared access signature feature that allows fine-grained permissions control on containers. Precision access control enables scenarios that further improve scalability, so thinking about containers as security boundaries in general is a helpful guideline.

The following is the typical workflow for apps that use Azure Blob storage:
- Retrieve configuration: At startup, load the storage account configuration. This is typically a storage account connection string.
- Initialize client: To initialize the Azure Storage client library, use the connection string. This creates the objects the app will use to work with the Blob storage API.
- Use: To operate on containers and blobs, make API calls with the client library.

In .NET SDK
```csharp
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(connectionString); // or TryParse()
CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
CloudBlobContainer container = blobClient.GetContainerReference(containerName);
```

To create a container when your app starts or when it first tries to use it, call `CreateIfNotExistsAsync` on a `CloudBlobContainer`.

`CreateIfNotExistsAsync` won't throw an exception if the container already exists, but it does make a network call to Azure Storage. Call it once during initialization, not every time you try to use a container.

check out `git clone https://github.com/MicrosoftDocs/mslearn-store-data-in-azure.git`

You can get a list of the blobs in a container using `CloudBlobContainer`'s `ListBlobsSegmentedAsync` method. Segmented refers to the separate pages of results returned — a single call to `ListBlobsSegmentedAsync` is never guaranteed to return all the results in a single page. You may need to call it repeatedly using the `ContinuationToken` it returns to work our way through the pages. This makes the code for listing blobs a little more complex than the code for uploading or downloading, but there's a standard pattern you can use to get every blob in a container.

```csharp
BlobContinuationToken continuationToken = null;
BlobResultSegment resultSegment = null;

do
{
    resultSegment = await container.ListBlobsSegmentedAsync(continuationToken);

    // Do work here on resultSegment.Results

    continuationToken = resultSegment.ContinuationToken;
} while (continuationToken != null);
```

to get the results:
```csharp
// Get all blobs
var allBlobs = resultSegment.Results.OfType<ICloudBlob>();

// Get only block blobs
var blockBlobs = resultSegment.Results.OfType<CloudBlockBlob>();
```

To create a new blob, call one of the `Upload` methods on a reference to a blob that doesn't exist in storage. This does two things: creates the blob in storage, and uploads the data.

In the Azure Storage SDK for .NET Core, all methods that require network activity return `Task`s, so make sure you use `await` in your controller methods appropriately.

### 3 Implement Azure Security (20-25%) {#3}

#### 3.1 Implement user authentication and authorization {#3.1}

##### Pluralsight notes

###### Secure Azure Storage

Securing Azure Storage in 3 dimensions:
1. Management plane: user and permission management
2. Data plane: what are the different mechanisms to access the data
3. Encryption

__Management plane__
- RBAC: role based access controls
    - Security principal: somebody or something you want to grant access to.
        - User
        - Group
        - Service principal: headless process
        - Managed identity: service principal which is a Microsoft managed service principal
    - Role definition: json file with defined permissions
    - Scope: limit the actions to a scope
        - Management group
        - Subscription
        - Resource group
        - Resource

Role assignment: Attach role definition to a security principal on a scope

Multiple role assignments are additive

Deny assignments can block access, overruling role assignments

__Data plane__
Three ways to secure access to storage:
- Keys: storage account generates two 512 bit keys that a security principal can use to access the storage
- Shared Access Signature: least privilege mechanism when compared to keys. SAS are attached to a URL, can also be time-based.
- Azure AD: when you access to storage with AAD, you connect with standard OpenID mechanisms
    - No dependency on key, you use an access token

Storage account keys
- They come in pairs, so if you want to change one, your app can still continue
- Give root access
- Rotate

Good practice: automatically rotate the Keys using Azure KeyVault.

Shared Access Signatures: secure, delegated access, without sharing the key. Control what clients access and for how long. Three type of SAS's:
- User delegation SAS: this is when you use Azure AD
- Service SAS: secured with the storage account key, but gives access to only some things inside the storage account
- Account SAS: secured with the storage account key. It delegates access to resources in one or more of the storage services. Delegate read/write access to resources in the storage account.

A typical SAS token, with it's attributes, looks like [this](https://docs.microsoft.com/en-us/rest/api/storageservices/create-account-sas). The signature attribute is a `StringToSign`.

Another way to categorize SAS:
- Ad-hoc SAS: everything you need is inside the token
- Service SAS: with link to stored access policy

###### Authenticating using Azure AD

The Microsoft Identity Platform:
- Authentication service
    - Azure Active Directory
        - Azure AD Connect
        - ADFS, federate authentication, based on open standard
        - much more...
- Open-source libraries
    - MSAL, Microsoft authentication library (available for multiple frameworks)
    - Microsoft.Identity.Web, .NET EF core compatible
    - Open ID connect
- Application management tools
    - Gallery and non gallery apps: gallery apps are commonly used apps, like dropbox, google apps, etc. Non-gallery apps are apps that are not famous, but use open standard protocols.
    - Single tenant and multi tenant apps
    - Authorization
    - Consent
    - Logs
    - much more...

__Modern Authentication__

Identity:
- Legacy
    - Basic authentication: username-password in header
    - NLTM
    - Kerberos: domain controller issuing tickets, good for on-premises
- Modern (cloud, internet, cross-platfrom): no industry standard definition
    - WS-* and SAML
    - OAuth: delegation protocol, used as a pseudo authentication method
    - OpenID Connect: includes user information
        - User loging into an application, application connects to an API, and the API connects to another downstream API. With OpenID Connect you can authenticate to the downstream components with the user or the application or both.
        - OpenID Connect supports different flows.
        - All types of applications send out an Access Token to the API. The API validates the token with Azure AD.
            - OnBehalfOfFlow: let's me get a token for an API on behalf of a user.

Open ID Connect Tokens
- Access Token: what you present to an API. Who are you, what do you want, are you authorized?
- ID Token: user's identity. If you are validated, it allows a session between the server and the browser.
- Refresh token: used to get new access tokens when, for example, the existing access tokens are about to expire

You can register an app with Azure Active Directory. When you go to quickstart from the registered app in the Azure portal, you can create a project that uses Azure Active Directory for the login process.

###### Authorization using Azure AD

Authorization = what you can do.

Advice: do not overengineer authorization

Entities: things that can authenticate in Azure Active Directory
- Apps
    - Service principals
    - Managed identities, which are service principals of whom credentials are managed by Azure Active Directory and don't have a backing app registration
- User

Three different ways you can do authorization:
- Groups: user or app groups
- Custom Claims: information you can put on the ID Token or Access Token based on which you can drive authorization decisions.
- App Roles: something you define at the app level and then those rules can be assigned to a user or an app and they surface up in the necessary tokens.

#### Microsoft Learn

##### Protect against security threats on Azure

Azure Security Center is a monitoring service that provides visibility of your security posture across all of your services, both on Azure and on-premises. The term security posture refers to cybersecurity policies and controls, as well as how well you can predict, prevent, and respond to security threats.

In the Resource security hygiene section, you can see the health of its resources from a security perspective. To help prioritize remediation actions, recommendations are categorized as low, medium, and high.

Secure score is a measurement of an organization's security posture. Secure score is based on security controls, or groups of related security recommendations. Your score is based on the percentage of security controls that you satisfy. The more security controls you satisfy, the higher the score you receive. Your score improves when you remediate all of the recommendations for a single resource within a control.

Azure Sentinel enables you to:
- Collect cloud data at scale. Collect data across all users, devices, applications, and infrastructure, both on-premises and from multiple clouds.
- Detect previously undetected threats. Minimize false positives by using Microsoft's comprehensive analytics and threat intelligence.
- Investigate threats with artificial intelligence. Examine suspicious activities at scale, tapping into years of cybersecurity experience from Microsoft.
- Respond to incidents rapidly. Use built-in orchestration and automation of common tasks.

Azure Sentinel supports a number of data sources, which it can analyze for security events. These connections are handled by built-in connectors or industry-standard log formats and APIs.
- Connect Microsoft solutions. Connectors provide real-time integration for services like Microsoft Threat Protection solutions, Microsoft 365 sources (including Office 365), Azure Active Directory, and Windows Defender Firewall.
- Connect other services and solutions. Connectors are available for common non-Microsoft services and solutions, including AWS CloudTrail, Citrix Analytics (Security), Sophos XG Firewall, VMware Carbon Black Cloud, and Okta SSO.
- Connect industry-standard data sources. Azure Sentinel supports data from other sources that use the Common Event Format (CEF) messaging standard, Syslog, or REST API.

Built in analytics use templates designed by Microsoft's team of security experts and analysts based on known threats, common attack vectors, and escalation chains for suspicious activity. Custom analytics are rules that you create to search for specific criteria within your environment. You can preview the number of results that the query would generate (based on past log events) and set a schedule for the query to run. You can also set an alert threshold.

Use Azure Monitor Workbooks to automate responses to threats. For example, it can set an alert that looks for malicious IP addresses that access the network and create a workbook that does the following steps:
1. When the alert is triggered, open a ticket in the IT ticketing system.
2. Send a message to the security operations channel in Microsoft Teams or Slack to make sure the security analysts are aware of the incident.
3. Send all of the information in the alert to the senior network admin and to the security admin. The email message includes two user option buttons: Block or Ignore.

Azure Key Vault is a centralized cloud service for storing an application's secrets in a single, central location. It provides secure access to sensitive information by providing access control and logging capabilities.

Azure Key Vault can help you:
- Manage secrets
- Manage encryption keys
- Manage SSL/TLS certificates
- Store secrets backed by hardware security modules (HSMSs)

The benefits of Key Vault include:
- Centralized application secrets
- Securely stored secrets and keys
- Access monitoring and access control
- Simplified administration of application secrets
- Integration with other Azure services

On Azure, virtual machines (VMs) run on shared hardware that Microsoft manages. Although the underlying hardware is shared, your VM workloads are isolated from workloads that other Azure customers run. Some organizations must follow regulatory compliance that requires them to be the only customer using the physical machine that hosts their virtual machines. Azure Dedicated Host provides dedicated physical servers to host your Azure VMs for Windows and Linux. A dedicated host is mapped to a physical server in an Azure datacenter. A host group is a collection of dedicated hosts.

Azure Dedicated Host:
- Gives you visibility into, and control over, the server infrastructure that's running your Azure VMs.
- Helps address compliance requirements by deploying your workloads on an isolated server.
- Lets you choose the number of processors, server capabilities, VM series, and VM sizes within the same host.

For high availability, you can provision multiple hosts in a host group, and deploy your VMs across this group. VMs on dedicated hosts can also take advantage of maintenance control. This feature enables you to control when regular maintenance updates occur, within a 35-day rolling window.

##### Top 5 security items to consider before pushing to production