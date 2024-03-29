## Getting Started

The following instructions will get you up and running with Apache Guacamole in no time at all.

### Prerequisites

In order to deploy this Azure Resource Manager template you will need provide the following parameters.

| Parameter                 | Description  |
| :-------------------------|:-------------|
| **Name**                   | Name of your Apache Guacamole Virtual Machine. |
| **DNS Prefix**             | DNS prefix for the assigned the Public IP address. |
| **Size**                   | Compute instance size (e.g. Standard_A2_v2). |
| **Username**               | Apache Guacamole Virtual Machine Administrator username.|
| **SSH Public Key**         | SSH Public Key that will be used for SSH authentication.|
| **Vnet Name**              | Name of an existing Virtual Network.|
| **Vnet RG**                | Name of the Resource Group where your Virtual Network resides.|
| **Subnet Name**            | Name of an existing Subnet within your Virtual Network.|
| **Mysql Root Pwd**         | MySQL root password.|
 **Guac DB Pwd**            | Guacamole database password.|
| **Cert Email Address**     | Email address to be used with the generation of your Let's Encrypt SSL certificate.|

### Generate your SSH Public Key

The SSH protocol supports many authentication methods. Arguably one of the most important of these is public key authentication for ineractive and automated connections. As such, this Azure Resource Manager template requires the use of SSH public key authentication. In order to generate a Public and Private key, you can use a tool called PuTTYgen (Windows) or ssh-keygen (Linux).

References
* <a href="https://docs.microsoft.com/en-us/azure/virtual-machines/linux/ssh-from-windows">Create and use SSH keys on Windows</a>
* <a href="https://docs.microsoft.com/en-us/azure/virtual-machines/linux/mac-create-ssh-keys">Create and use SSH keys on Linux or macOS</a>

### Create an Azure Virtual Network

Most of you should already have an Azure Virtual Network with multiple subnets. However, if this is not the case, you can follow the references below to create an Azure Virtual Network.

```
Reminder: You will need to remember the Resource Group that your Virtual Network resides within.
```
References
* <a href="https://docs.microsoft.com/en-us/azure/virtual-network/quick-create-portal">Create a virtual network using the Azure portal</a>
* <a href="https://docs.microsoft.com/en-ca/azure/virtual-network/quick-create-powershell">Create a virtual network using PowerShell</a>

### Deploy to Azure

Once you have the answers to each of the rerequisites, you are ready to proceed with deploying this Azure Resource Manager template by clicking the **Deploy to Azure** button below.

The following will occur as part of the deployment to Microsoft Azure:
* Creation of a Ubuntu 16.04 virtual machine
* Creation of a Public IP address with your specified DNS prefix
* Creation of a Network Security Group that allows incoming traffic on ports 22, 80, 443.
  * Ports 8080 and 8443 are proxied via the Nginx server to Tomcat thus they are not required to be open.
* Installation of Nginx, Tomcat, MySQL, Guacamole, and a Let's Encrypt SSL certificate

<a href="https://portal.azure.us/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FBarryJNewman%2Fguacamole%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>
<a href="http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2Fbigredthelogger%2Fguacamole%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="http://armviz.io/visualizebutton.png"/>
</a>

### Login to Guacamole and change your password

The default username and password is **guacadmin**/**guacadmin**. Make sure that you login and change this.

### Customization

If you would like to customize this template to meet your own specific business requirments, you can do so very simply. 

* Make a copy of the entry point script <a href="https://raw.githubusercontent.com/bigredthelogger/guacamole/master/install.sh">install.sh</a>.
  * Make your desired changes
  * Upload your new script to a publicly available location (e.g. Github) or your Azure Storage Account
  * Override the **scriptActionUri_entrypoint** parameter with the URL to your customized entry point script.
* Make a copy of the Nginx install script <a href="https://raw.githubusercontent.com/bigredthelogger/guacamole/master/nginx-install.sh">nginx-install.sh</a>.
  * Make your desired changes
  * Upload your new script to a publicly available location (e.g. Github) or your Azure Storage Account
  * Override the **scriptActionUri_nginx** parameter with the URL to your customized Nginx script.
* Make a copy of the Guacamole install script <a href="https://raw.githubusercontent.com/bigredthelogger/guacamole/master/guac-install.sh">guac-install.sh</a>.
  * Make your desired changes
  * Upload your new script to a publicly available location (e.g. Github) or your Azure Storage Account
  * Override the **scriptActionUri_guacamole** parameter with the URL to your customized Guacamole install script.

## Lessons Learned

### 1. Custom Script Extensions 

Custom Script Extensions download and runs scripts on Azure virtual machines. These are useful for post-deployment configuration, software installation, or any other configuration/management task. You can download scripts from Azure Storage or another accessible internet location, or even provide them to the extension at runtime.
```
Tip: Only a single Custom Script Extension can be assigned to an Azure virtual machine.
Option 1: Create a single script that executes all your desired commands or
Option 2: Create an entry point script (reference install.sh in this project) that executes your individual scripts.
```
References
* <a href="https://docs.microsoft.com/en-us/azure/virtual-machines/windows/extensions-customscript">Custom Script Extension for Windows</a>
* <a href="https://docs.microsoft.com/en-us/azure/virtual-machines/linux/extensions-customscript">Custom Script Extension for Linux</a>

### 2. Network Security Groups

Be very careful when you want to apply Network Security Groups to both a Azure virtual machine NIC and the subnet at the same time: NSGs are evaluated independently, and an "allow" rule must exist at both levels, otherwise traffic will not be admitted.

References
* <a href="https://blogs.msdn.microsoft.com/igorpag/2016/05/14/azure-network-security-groups-nsg-best-practices-and-lessons-learned/">Azure Network Security Groups (NSG) – Best Practices and Lessons Learned</a>

### 3. Azure Resource Manager (ARM) Templates

You can inject a script or other metadata into a Microsoft Azure virtual machine at provision time using a feature called custom data. Custom data needs to be base64-encoded and can contain data such as variables, parameters, scripts, or even cloud-init syntax.

You can use cloud-init to install packages and write files, or to configure users and security on select Linux images (e.g. Ubuntu, CoreOS, and RHEL).

References
* <a href="https://azure.microsoft.com/en-us/blog/custom-data-and-cloud-init-on-windows-azure/">Custom Data and Cloud-Init on Microsoft Azure</a>
* <a href="https://docs.microsoft.com/en-us/azure/virtual-machines/linux/using-cloud-init">Cloud-init support for virtual machines in Azure</a>
* <a href="https://help.ubuntu.com/community/CloudInit">CloudInit Official Documentation</a>
* <a href="https://github.com/Azure/azure-quickstart-templates/blob/master/1-CONTRIBUTION-GUIDE/best-practices.md">Azure Resource Manager QuickStart Templates</a>

## Acknowledgments

* <a href="https://github.com/Azure/azure-quickstart-templates/tree/master/guacamole-rdp-vnc-gateway-existing-vnet">Azure Quickstart Templates</a>
* <a href="https://github.com/MysticRyuujin/guac-install">Chase Wright</a>
