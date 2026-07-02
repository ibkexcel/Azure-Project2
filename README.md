# Azure-Project2
  **ARM Template (JSON/parameters) File:**
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "metadata": {
    "_generator": {
      "name": "bicep",
      "version": "0.16.2.56959",
      "templateHash": "14427937023370378081"
    }
  },
  "parameters": {
    "adminUsername": {
      "type": "string",
      "metadata": {
        "description": "Username for the Virtual Machine."
      }
    },
    "adminPassword": {
      "type": "securestring",
      "minLength": 12,
      "metadata": {
        "description": "Password for the Virtual Machine."
      }
    },
    "dnsLabelPrefix": {
      "type": "string",
      "defaultValue": "[toLower(format('{0}-{1}', parameters('vmName'), uniqueString(resourceGroup().id, parameters('vmName'))))]",
      "metadata": {
        "description": "Unique DNS Name for the Public IP used to access the Virtual Machine."
      }
    },
    "publicIpName": {
      "type": "string",
      "defaultValue": "myPublicIP",
      "metadata": {
        "description": "Name for the Public IP used to access the Virtual Machine."
      }
    },
    "publicIPAllocationMethod": {
      "type": "string",
      "defaultValue": "Static",
      "metadata": {
        "description": "Allocation method for the Public IP used to access the Virtual Machine."
      }
    },
    "publicIpSku": {
      "type": "string",
      "defaultValue": "Standard",
      "metadata": {
        "description": "SKU for the Public IP used to access the Virtual Machine."
      }
    },
    "OSVersion": {
      "type": "string",
      "defaultValue": "2022-datacenter-azure-edition",
      "allowedValues": [
        "2016-datacenter-gensecond",
        "2016-datacenter-server-core-g2",
        "2016-datacenter-server-core-smalldisk-g2",
        "2016-datacenter-smalldisk-g2",
        "2016-datacenter-with-containers-g2",
        "2016-datacenter-zhcn-g2",
        "2019-datacenter-core-g2",
        "2019-datacenter-core-smalldisk-g2",
        "2019-datacenter-core-with-containers-g2",
        "2019-datacenter-core-with-containers-smalldisk-g2",
        "2019-datacenter-gensecond",
        "2019-datacenter-smalldisk-g2",
        "2019-datacenter-with-containers-g2",
        "2019-datacenter-with-containers-smalldisk-g2",
        "2019-datacenter-zhcn-g2",
        "2022-datacenter-azure-edition",
        "2022-datacenter-azure-edition-core",
        "2022-datacenter-azure-edition-core-smalldisk",
        "2022-datacenter-azure-edition-smalldisk",
        "2022-datacenter-core-g2",
        "2022-datacenter-core-smalldisk-g2",
        "2022-datacenter-g2",
        "2022-datacenter-smalldisk-g2"
      ],
      "metadata": {
        "description": "The Windows version for the VM. This will pick a fully patched image of this given Windows version."
      }
    },
    "vmSize": {
      "type": "string",
      "defaultValue": "Standard_D2s_v5",
      "metadata": {
        "description": "Size of the virtual machine."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    },
    "vmName": {
      "type": "string",
      "defaultValue": "class-vm",
      "metadata": {
        "description": "Name of the virtual machine."
      }
    },
    "securityType": {
      "type": "string",
      "defaultValue": "TrustedLaunch",
      "allowedValues": [
        "Standard",
        "TrustedLaunch"
      ],
      "metadata": {
        "description": "Security Type of the Virtual Machine."
      }
    }
  },
  "variables": {
    "storageAccountName": "[format('bootdiags{0}', uniqueString(resourceGroup().id))]",
    "nicName": "myVMNic",
    "addressPrefix": "10.0.0.0/16",
    "subnetName": "Subnet",
    "subnetPrefix": "10.0.0.0/24",
    "virtualNetworkName": "MyVNET",
    "networkSecurityGroupName": "default-NSG",
    "securityProfileJson": {
      "uefiSettings": {
        "secureBootEnabled": true,
        "vTpmEnabled": true
      },
      "securityType": "[parameters('securityType')]"
    },
    "extensionName": "GuestAttestation",
    "extensionPublisher": "Microsoft.Azure.Security.WindowsAttestation",
    "extensionVersion": "1.0",
    "maaTenantName": "GuestAttestation",
    "maaEndpoint": "[substring('emptyString', 0, 0)]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2022-05-01",
      "name": "[variables('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": {
        "defaultToOAuthAuthentication": true
      }
    },
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2022-05-01",
      "name": "[parameters('publicIpName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('publicIpSku')]"
      },
      "properties": {
        "publicIPAllocationMethod": "[parameters('publicIPAllocationMethod')]",
        "dnsSettings": {
          "domainNameLabel": "[parameters('dnsLabelPrefix')]"
        }
      }
    },
    {
      "type": "Microsoft.Network/networkSecurityGroups",
      "apiVersion": "2022-05-01",
      "name": "[variables('networkSecurityGroupName')]",
      "location": "[parameters('location')]",
      "properties": {
        "securityRules": [
          {
            "name": "default-allow-3389",
            "properties": {
              "priority": 1000,
              "access": "Allow",
              "direction": "Inbound",
              "destinationPortRange": "3389",
              "protocol": "Tcp",
              "sourcePortRange": "*",
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "*"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2022-05-01",
      "name": "[variables('virtualNetworkName')]",
      "location": "[parameters('location')]",
      "properties": {
        "addressSpace": {
          "addressPrefixes": [
            "[variables('addressPrefix')]"
          ]
        },
        "subnets": [
          {
            "name": "[variables('subnetName')]",
            "properties": {
              "addressPrefix": "[variables('subnetPrefix')]",
              "networkSecurityGroup": {
                "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
              }
            }
          }
        ]
      },
      "dependsOn": [
        "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
      ]
    },
    {
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2022-05-01",
      "name": "[variables('nicName')]",
      "location": "[parameters('location')]",
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipconfig1",
            "properties": {
              "privateIPAllocationMethod": "Dynamic",
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIpName'))]"
              },
              "subnet": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]"
              }
            }
          }
        ]
      },
      "dependsOn": [
        "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIpName'))]",
        "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworkName'))]"
      ]
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2022-03-01",
      "name": "[parameters('vmName')]",
      "location": "[parameters('location')]",
      "properties": {
        "hardwareProfile": {
          "vmSize": "[parameters('vmSize')]"
        },
        "osProfile": {
          "computerName": "[parameters('vmName')]",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPassword')]"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "[parameters('OSVersion')]",
            "version": "latest"
          },
          "osDisk": {
            "createOption": "FromImage",
            "managedDisk": {
              "storageAccountType": "StandardSSD_LRS"
            }
          },
          "dataDisks": [
            {
              "diskSizeGB": 1023,
              "lun": 0,
              "createOption": "Empty"
            }
          ]
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
            }
          ]
        },
        "diagnosticsProfile": {
          "bootDiagnostics": {
            "enabled": true,
            "storageUri": "[reference(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2022-05-01').primaryEndpoints.blob]"
          }
        },
        "securityProfile": "[if(equals(parameters('securityType'), 'TrustedLaunch'), variables('securityProfileJson'), null())]"
      },
      "dependsOn": [
        "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]",
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
      ]
    },
    {
      "condition": "[and(equals(parameters('securityType'), 'TrustedLaunch'), and(equals(variables('securityProfileJson').uefiSettings.secureBootEnabled, true()), equals(variables('securityProfileJson').uefiSettings.vTpmEnabled, true())))]",
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "apiVersion": "2022-03-01",
      "name": "[format('{0}/{1}', parameters('vmName'), variables('extensionName'))]",
      "location": "[parameters('location')]",
      "properties": {
        "publisher": "[variables('extensionPublisher')]",
        "type": "[variables('extensionName')]",
        "typeHandlerVersion": "[variables('extensionVersion')]",
        "autoUpgradeMinorVersion": true,
        "enableAutomaticUpgrade": true,
        "settings": {
          "AttestationConfig": {
            "MaaSettings": {
              "maaEndpoint": "[variables('maaEndpoint')]",
              "maaTenantName": "[variables('maaTenantName')]"
            }
          }
        }
      },
      "dependsOn": [
        "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
      ]
    }
  ],
  "outputs": {
    "hostname": {
      "type": "string",
      "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIpName')), '2022-05-01').dnsSettings.fqdn]"
    }
  }
}

**ARM Template (Structure Definition)**
The structure of an Azure Resource Manager (ARM) template includes several key elements that define how resources are deployed in Azure. Here's a breakdown of the main components:
$schema: This is a required element that specifies the location of the JSON schema file, which describes the version of the template language.
contentVersion: This required element indicates the version of the template, such as "1.0.0.0". It helps track significant changes in the template.
parameters: These are optional values provided during deployment to customize resource deployment. They allow for flexibility and reusability of the template.
variables: Optional values used as JSON fragments within the template to simplify expressions. They help reduce complexity in the template.
resources: This is a required section that defines the resource types to be deployed or updated in a resource group or subscription. It is the core part of the template.
outputs: Optional values that are returned after deployment, providing information about the deployed resources.
ARM Template Structure - You'll need to manually create the azuredeploy.json with these sections:
$schema - Template schema version
contentVersion - Your template version
parameters - Input values (VM name, location, admin credentials)
variables - Computed values and constants
resources - Azure resources to deploy
outputs - Return values after deployment
Key Resources to Define:
Virtual Network and Subnet
Public IP Address
Network Security Group (NSG)
Network Interface (NIC)
Virtual Machine
Dependencies - Use dependsOn arrays to ensure proper creation order

**Deployment Documentation**
Here is a brief document detailing the Azure CLI commands used to execute the deployment of an ARM template:
Azure CLI Deployment Commands
Create a Resource Group:
Before deploying an ARM template, create a resource group to contain the resources.
Shell
az group create --name 3MTT-PROJECT --location "Central US"
Deploy the ARM Template from a URI:
Deploy the template directly from a URI to the specified resource group.
Shell
az deployment group create --name classproject-deployment--resource-group ExampleGroup --template-file "C:\Users\user\Desktop\AZURE SUBMISSION PROJECTS\azuredeploytemplate.json file.txt -templates/master/quickstarts/microsoft.storage/storage-account-create/azuredeploy.json" --parameters storageAccountType=Standard_GRS
Deploy the ARM Template from a Local File:
Alternatively, deploy the template from a local file.
Shell
az deployment group create --resource-group ExampleGroup --template-file "./mainTemplate.json"
Create a Template Spec:
Create a template spec from a local ARM template file for reuse.
Shell
az ts create --name storageSpec --version "1.0" --resource-group templateSpecRG --location "westus2" --template-file "./mainTemplate.json"
   
Get the Template Spec ID:
Retrieve the ID of the created template spec for further deployments.
Shell
az ts show --name storageSpec --resource-group templateSpecRG --query id --output tsv
Deploy Using the Template Spec ID:
Deploy the ARM template spec to a specified resource group using its ID.
Shell
az deployment group create --resource-group demoRG --template-spec $id
This document outlines the key steps and commands used to deploy an ARM template using Azure CLI, providing a clear guide for executing deployments. Adjust the parameters as needed for your specific deployment scenario.

**Output Log (deployment outputs)**
public IP - 52.233.85.115
MyVNET/Subnet
DNS name - class-vm-hr7gkn7v3ccus.westus2.cloudapp.azure.com
Resource ID - /subscriptions/6d30b99d-00f1-48fa-aedc-1fd169f29504/resourceGroups/3MTT-PROJECT

**Deployment Overview**
My ARM Template deploys several Azure resources, including:
Virtual Machine (VM)
Virtual Network (VNET)
Network Interface Card (NIC)
Storage Account
Network Security Group (NSG)

**Deployment Commands**
To deploy these resources using Azure CLI, you can use the following commands:
Start a Deployment at Subscription Scope:
This command initiates a deployment using your ARM template.
Bash
az deployment sub create --name <deploymentName> --location <location> --template-file <templateFile> --parameters <parametersFile>
  --name: The name of the deployment.
--location: The location for the deployment.
--template-file: The path to your ARM template file.
--parameters: The path to your parameters file.
Translate ARM Template to CLI Scripts:
This command helps translate your ARM template into CLI scripts for a better understanding of the resources.
Bash
 az cli-translator arm translate --template <templateFile> --parameters <parametersFile> --resource-group <resourceGroup>

 **Verification Screenshots (Evidence of Deployment)**
<img width="1366" height="768" alt="login" src="https://github.com/user-attachments/assets/7799ab86-0590-4c16-bd4d-b6a212046222" />
<img width="1366" height="768" alt="loginCode" src="https://github.com/user-attachments/assets/c4729020-7d39-4012-8f00-dc28982f635f" />
<img width="1366" height="768" alt="Subscription" src="https://github.com/user-attachments/assets/eab3b7e4-39d2-42d2-a2e3-15d53fa7de8b" />
<img width="1366" height="768" alt="Provisioning" src="https://github.com/user-attachments/assets/4562198a-992a-4a57-96fb-6682a6ced3fe" />
<img width="1366" height="768" alt="Overview" src="https://github.com/user-attachments/assets/40d9b9af-94cc-491b-97c7-7ffec27efa79" />
<img width="1366" height="768" alt="vm-disk2" src="https://github.com/user-attachments/assets/39b773a5-27c9-4b18-95f0-9f46522adff7" />
<img width="1366" height="768" alt="Storage" src="https://github.com/user-attachments/assets/67a66d3b-60b8-4861-a938-fbdb3ec844ed" />
<img width="1366" height="768" alt="vm-OSDisk" src="https://github.com/user-attachments/assets/668c3cb8-bb13-4a64-92ba-2ef5e59f0419" />
<img width="1366" height="768" alt="classvm1" src="https://github.com/user-attachments/assets/8fdac825-d2b8-49bb-8f07-ef8ca333ad7b" />
<img width="1366" height="768" alt="classvm2" src="https://github.com/user-attachments/assets/ad25dc58-bd3c-48b6-960e-4291c8db2931" />
<img width="1366" height="768" alt="class-vmRunning" src="https://github.com/user-attachments/assets/2fec2ad4-1661-4f9a-a76b-ec62e58b847d" />
<img width="1366" height="768" alt="deploymentsucceed" src="https://github.com/user-attachments/assets/161a6031-8b64-42fc-b552-c65409ee5112" />
<img width="1366" height="768" alt="myVmNic" src="https://github.com/user-attachments/assets/f38f865a-59d5-41fd-8dec-36d542693f67" />
<img width="1366" height="768" alt="MyVnet" src="https://github.com/user-attachments/assets/af05ec04-e6f4-4115-bb79-9342194b059f" />
<img width="1366" height="768" alt="NSG" src="https://github.com/user-attachments/assets/34cae6c9-7000-460a-9410-cc6b6e4f513f" />
<img width="1366" height="768" alt="publicIP" src="https://github.com/user-attachments/assets/4949c2ca-eb4f-4f32-aeb9-f82fb862b26b" />
<img width="1366" height="768" alt="Storage" src="https://github.com/user-attachments/assets/c891380d-8ecf-4717-9276-86b470053a63" />

**Project Description Summary**
*To define a Virtual Machine (VM) resource within an ARM template, you need to specify several key components:*
Operating System: Specify the type of operating system for the VM. This can be either Windows or Linux. The osType property in the template allows you to define this.

VM Size: Define the size of the VM, which determines the number of CPUs, memory, and other resources allocated to the VM. This is specified in the hardwareProfile section of the template.
Disk Types: Configure the disk settings for the VM, including the OS disk and any data disks. You can specify the size, type (managed or unmanaged), and encryption settings for each disk.
Authentication Credentials: Set up the authentication method for accessing the VM. This can include specifying an admin username and password or using SSH keys for Linux VMs. The osProfile section of the template is used to define these credentials.
By configuring these elements in your ARM template, you can deploy a VM that meets your specific requirements for operating system, performance, storage, and security.

**To effectively implement parameterization, manage resource dependencies, and apply security controls in your ARM templates, consider the following key points:**
*Implement Parameterization*
Parameters: Use parameters to replace hardcoded values, allowing for dynamic inputs such as VM name, location, and admin credentials. This makes your template more flexible and reusable. For sensitive data like passwords, use the securestring type to ensure security.
*Manage Resource Dependencies*
Dependencies: Use the dependsOn attribute to specify the order of resource creation. This ensures that resources like a Network Interface (NIC) are created before the Virtual Machine (VM) that depends on them. Properly managing dependencies helps avoid deployment errors and ensures resources are available when needed.
*Apply Security Controls*
Network Security Group (NSG): Integrate an NSG to manage inbound and outbound traffic. Define security rules to control access to your resources, enhancing security by allowing only necessary traffic.
Authentication: Configure SSH keys or password-based authentication for accessing VMs. Use secure methods to store and manage these credentials, such as Azure Key Vault, to protect sensitive information.
By following these practices, you can create robust and secure ARM templates that are adaptable to different deployment scenarios.

**To define the networking infrastructure and compute resources for my ARM template deployment, I focus on the following key components:**
Networking Infrastructure
Virtual Network (VNet):
A VNet is a fundamental building block for your private network in Azure. It enables Azure resources to securely communicate with each other, the internet, and on-premises networks.
Define the address space and subnets within the VNet to segment the network.
Subnets:
Subnets are segments within a VNet that allow you to organize and secure your resources. Each subnet can have its own security policies and address range.
Network Interfaces (NIC):
NICs connect your virtual machines to a VNet. They provide the VM with an IP address and are essential for network communication.
Public IP Addresses:
Public IPs allow Azure resources to communicate with the internet. They are typically used for VMs that need to be accessible from outside the VNet.
Compute Resources
Virtual Machine (VM):
Define the VM resource within the template, specifying the operating system, VM size, and disk types.
Configure authentication credentials, such as admin username and password or SSH keys, to secure access to the VM.
By configuring these components in your ARM template, you can deploy a comprehensive infrastructure that supports your application needs. This setup ensures that your resources are well-organized, secure, and capable of communicating effectively within Azure and beyond.

**To troubleshoot and validate your Virtual Machine (VM) deployment, follow these steps:**
*Validate Template Syntax and Parameters:*
Run an ARM template validation using commands like az deployment group validate to ensure there are no syntax errors. Make sure parameters such as VM name, location, and admin credentials are correctly defined and used.
Verify Resource Dependencies:
Ensure that all dependent resources (e.g., Virtual Network, Subnet, Network Interface, Public IP, NSG, Disk) are created before the VM. Use the dependsOn attribute in your ARM template to manage the order of resource creation.
Apply Security Controls:
Attach a Network Security Group (NSG) to manage inbound and outbound traffic. Ensure that the NSG rules allow necessary traffic (e.g., SSH or RDP) from permitted sources. Configure authentication using SSH keys for Linux or strong passwords for Windows.
Configure Compute Resources:
Specify the operating system, VM size, disk types, and authentication credentials in the VM resource block. Parameterize these values for flexibility and reuse.
Troubleshoot Deployment or Connectivity Failures:
If the deployment succeeds but you encounter connection issues, consider redeploying the VM to a new node. This can resolve many connectivity problems. Use diagnostic tools like Azure Performance Diagnostics (PerfInsights) to check for issues.
Validate Network Connectivity:
Ensure a Public IP is attached to the NIC and that NSG rules allow management port access. Use VM Insights Map to observe connection metrics and diagnose any failed connections.
By following these steps, you can effectively troubleshoot and validate your VM deployment, ensuring successful connectivity and operation.

**Project Completion Checklists**
**Here's a project checklist for deploying resources using an ARM template, covering parameterization, resource dependencies, security controls, compute resource configuration, and troubleshooting:**
*Project Checklist for ARM Template Deployment*
Parameterization:
Define parameters for dynamic inputs such as VM name, location, and admin credentials.
Use securestring for sensitive data like passwords.
Ensure all parameters are correctly referenced in the template.
Resource Dependencies:
Use the dependsOn attribute to manage the order of resource creation.
Verify that all dependent resources (e.g., NIC, VNet) are created before the VM.
Security Controls:
Integrate a Network Security Group (NSG) to manage inbound and outbound traffic.
Configure NSG rules to allow necessary traffic (e.g., SSH or RDP) and block unnecessary traffic.
Set up authentication using SSH keys for Linux or strong passwords for Windows.
Compute Resource Configuration:
Specify the operating system, VM size, and disk types in the VM resource block.
Parameterize these values for flexibility and reuse.
Ensure the osProfile section includes admin username and authentication method.
Troubleshooting and Validation:
Validate the template syntax using tools like az deployment group validate.
Resolve any syntax or configuration errors identified during validation.
Verify successful deployment by connecting to the VM via its public IP.
Use diagnostic tools like Azure Performance Diagnostics (PerfInsights) for troubleshooting.
This checklist ensures a structured approach to deploying resources with ARM templates, focusing on flexibility, security, and successful deployment validation.







