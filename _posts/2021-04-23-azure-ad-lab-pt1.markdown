---
layout: post
title:  "It's just a lab in the cloud - Part 1"
date:   2021-04-23 14:21:21 -0400
categories: [Software]
tags: [azure, terraform, ansible]
permalink: /lab1
---

I wanted to have an Active Directory lab set up for playing around with different [Identity Providers](https://en.wikipedia.org/wiki/Identity_provider) (IDP) such as [Okta](https://www.okta.com/), [OneLogin](https://www.onelogin.com/), [Auth0](https://auth0.com/), etc. I thought what better way to do it if I can automate the provisioning of Active Directory with Azure and put my free $260 worth of credits to work! 

_I could have done some manual setup the old fashioned way by setting up a VMWare Workstation or Virtual Box then firing up a few Windows 10 ISOs but I already don't have enough resources on my PC/MAC._

So I thought, let's do a bit of research on how I can automate this because:
- I like getting things done with a _push of a button_
- maybe I'll learn a few tricks along the way

## **The Tools**
We'll be using the following tools for automating our lab: 
- [Azure](https://azure.microsoft.com/en-us/overview/what-is-azure/) - to take advantage of my credits, you can create your own account [here](https://azure.microsoft.com/en-ca/free/search/?&ef_id=Cj0KCQjw4ImEBhDFARIsAGOTMj9G2nJOszTaL69IeOf6hQ1lOchqN6HkGb-owKKq2f_O8IxHHuTIMd4aAlqxEALw_wcB:G:s&OCID=AID2100017_SEM_Cj0KCQjw4ImEBhDFARIsAGOTMj9G2nJOszTaL69IeOf6hQ1lOchqN6HkGb-owKKq2f_O8IxHHuTIMd4aAlqxEALw_wcB:G:s)
- [Terraform](https://www.terraform.io/intro/index.html) - I could have used ARM templates which are built-in to Azure but I wanted to learn Terraform's cloud-agnostic platform
- [Terraform Azure Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs) - we'll be using to define the resources we want to provision in Azure using Terraform
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-macos) - to manage Azure account and resources from your device
- [Ansible](https://www.ansible.com/) - I am learning the ropes with Ansible, but it easy enough to read and reference the YAML files we'll be using

## **The Requirements**
- A Windows Server domain controller which is domain joined
- One or more Windows 10 workstation(s) joined to the domain controller

## **How long does?**
If you are a _speed-demon_, by all means click away and set all of this up manually. Running a timer took me about 15-20 mins to provision everything in Azure from start to finish. The tear-down takes a little under 5 mins.

## **Install [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)**
_**Note:** I am using Mac OS Big Sur version 11.2.3_
1. Open the Terminal and run `brew update && brew install azure-cli`

2. Enter `az login` and sign in to your Azure account.

## **Configure Terraform in Azure**
1. You'll need Terraform. [Install](https://learn.hashicorp.com/tutorials/terraform/install-cli) it with Homebrew. 
2. Create a new working folder, `mkdir AD_Lab && cd AD_Lab`

3. Then create a sub-folder called `terraform`, this is where all your Terraform `.tf` files will be, `mkdir terraform && cd terraform`. 

4. Create a new file called `provider.tf` with the following code. This is let Terraform know tht we are using the Azure Provider.
{% highlight terraform %}
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 2.41.0"
    }
  }
}

# Configure the Microsoft Azure Provider
provider "azurerm" {
  features {}
}
{% endhighlight %}

{:start="4"}
5. Run `terraform init`

# Create our virtual network

Let's get the virtual networking out of the way first. Before we can do that, we need a few things: 
- A _resource group_ to place our resources into - I'll be using East US, you can use one closest to you.
- A _virtual network_ 
- A _subnet_ for our servers
- A _network interface_ to attach to our DC
- A _private IP_ subnet
- A _public IP_ we can remote desktop into
- A _network security group_ 

1. Here's how we can define it in Terraform. Create a new file called `domain-controller-net.tf`. We'll put all our network modules here.
{% highlight terraform %}
# domain-controller-net.tf

# Resource group in East US
resource "azurerm_resource_group" "main" {
    name = "aa-identity-lab"
    location = "eastus"
}

# Virtual network of 10.0.0.0/16
resource "azurerm_virtual_network" "main" {
  name                = "virtual-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

# The subnet for servers
resource "azurerm_subnet" "servers" {
  name                 = "servers"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.10.0/24"]
}

# The domain controller network interface
resource "azurerm_network_interface" "main" {
  name                = "domain-controller-nic"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

# The domain controller private subnet
  ip_configuration {
    name                          = "static"
    subnet_id                     = azurerm_subnet.servers.id
    private_ip_address_allocation = "Static"
    private_ip_address = cidrhost("10.0.10.0/24", 10)
    public_ip_address_id = azurerm_public_ip.main.id
  }
}

# The domain controller public IP
resource "azurerm_public_ip" "main" {
  name                    = "dc-public-ip"
  location                = azurerm_resource_group.main.location
  resource_group_name     = azurerm_resource_group.main.name
  allocation_method       = "Static"
  idle_timeout_in_minutes = 30
}
{% endhighlight %}

{:start="2"}
2. You can verify with `terraform plan` or go ahead with `terraform apply --auto-approve` at this point to provision the resources in Azure. 

![Image](/assets/images/azure_idp_lab/azuretf1.png)

{:start="3"}
3. To see the output of the public IP, which you will need to Remote Desktop later, create a new file called `outputs.tf` and put the following code block.

{% highlight terraform %}
# Displays domain controller public ip
output "domain_controller_public_ip" {
  value = azurerm_public_ip.main.ip_address
}
{% endhighlight %}

{% highlight console %}
Outputs:

domain_controller_public_ip = 40.95.102.10
{% endhighlight %}

{:start="4"}
4. Create a network security group and we will allow both [RDP](https://docs.microsoft.com/en-us/troubleshoot/windows-server/remote/understanding-remote-desktop-protocol) and [WinRM](https://docs.microsoft.com/en-us/windows/win32/winrm/portal). Add this to `domain-controller-net.tf`. Then run `terraform init` again.

{% highlight terraform %}
# Note: you'll need to run 'terraform init' again before terraform apply-ing this, because 'http' is a new provider

# Dynamically retrieve our public outgoing IP
data "http" "outgoing_ip" {
  url = "http://ipv4.icanhazip.com"
}
locals {
  outgoing_ip = chomp(data.http.outgoing_ip.body)
}

# Network security group
resource "azurerm_network_security_group" "domain_controller" {
  name                = "domain-controller-nsg"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  # RDP
  security_rule {
    name                       = "Allow-RDP"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "3389"
    source_address_prefix      = "${local.outgoing_ip}/32"
    destination_address_prefix = "*"
  }

  # WinRM
  security_rule {
    name                       = "Allow-WinRM"
    priority                   = 101
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "5985"
    source_address_prefix      = "${local.outgoing_ip}/32"
    destination_address_prefix = "*"
  }
}

# Associate our network security group with the NIC of our domain controller
resource "azurerm_network_interface_security_group_association" "domain_controller" {
  network_interface_id      = azurerm_network_interface.main.id
  network_security_group_id = azurerm_network_security_group.domain_controller.id
}
{% endhighlight %}

{:start="5"}
5. Enter `terraform apply` and you should see your network security group added. 
![Image](/assets/images/azure_idp_lab/azuretf2.png)

## **Create our domain controller**

Now that we have our virtual network out of the way. Let's provision our domain controller. You can run the following commands to check the list of available Windows Server images, `az vm image list --publisher MicrosoftWindowsServer --all -o table > msserver.txt`. You can also output the catalogue for Windows Desktops by running `az vm image list --publisher MicrosoftWindowsServer --all -o table > windesk.txt`

In our case, we will be using the `MicrosoftWindowsServer:2019-Datacenter:latest`.

1. Here's the template for provisioning the domain controller.

{% highlight terraform %}
# Note: you'll need to run 'terraform init' before terraform apply-ing this, because 'random_password' is a new provider
# Generates a random password with 8 digits for our domain controller
resource "random_password" "domain_controller_password" {
  length = 8
}

# VM for our domain controller
resource "azurerm_virtual_machine" "domain_controller" {
  name                  = "domain-controller"
  location              = azurerm_resource_group.main.location
  resource_group_name   = azurerm_resource_group.main.name
  network_interface_ids = [azurerm_network_interface.main.id]
  vm_size               = "Standard_D1_v2"
  # Base image
  storage_image_reference {
    publisher = "MicrosoftWindowsServer"
    offer     = "WindowsServer"
    sku       = "2019-Datacenter"
    version   = "latest"
  }

  # Disk
  delete_os_disk_on_termination = true
  storage_os_disk {
    name              = "domain-controller-os-disk"
    create_option     = "FromImage"
  }
  os_profile {
    computer_name  = "DC-1"
    # Note: you can't use admin or Administrator in here, Azure won't allow you to do so
    admin_username = "drew"
    admin_password = random_password.domain_controller_password.result
  }
  os_profile_windows_config {
    # Enable WinRM - we'll need to later
    winrm {
      protocol = "HTTP"
    }
  }

  # Explicit depends on block on the domain controller nic, so when we terraform destroy, there will be no errors
  depends_on = [
   azurerm_network_interface_security_group_association.domain_controller
 ]

  tags = {
    # Needed for Ansible later
    kind = "domain_controller"
  }
}
{% endhighlight %}

{:start="2"}
2. Update the `outputs.tf` file and add the following code. 
{% highlight terraform %}
# Display the dc local admin username
output "local_admin_username" {
  value = nonsensitive(azurerm_virtual_machine.domain_controller.os_profile[*].admin_username)
}

# Displays the domain controller password
output "domain_controller_password" {
  value =  nonsensitive(random_password.domain_controller_password.result)
}
{% endhighlight %}

{% highlight console %}
Outputs:

domain_controller_public_ip = 40.95.102.10
{% endhighlight %}

{:start="3"}
3. At this point, if we run `terraform plan`, we will be adding a total of 9 resources.

{% highlight console %}
Plan: 9 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + domain_controller_password  = (known after apply)
  + domain_controller_public_ip = (known after apply)
{% endhighlight %}

{:start="4"}
4. Enter `terraform apply --auto-approve` then sit back, relax, or grab a coffee. Come back after a few minutes and hopefully provisioning is complete. 

{% highlight console %}
Apply complete! Resources: 9 added, 0 changed, 0 destroyed.

Outputs:

domain_controller_password = "L6ETmLFH"
domain_controller_public_ip = "40.114.33.254"
{% endhighlight %}

{:start="5"}
5. Download [Microsoft Remote Desktop for Mac](https://apps.apple.com/ca/app/microsoft-remote-desktop/id1295203466?mt=12) and enter the domain controller public ip and the _admin username_ and _password_. 

![Image](/assets/images/azure_idp_lab/azuretf4.png)

{:start="6"}
6. When you are done with your lab, don't forget to deprovision your resources by entering `terraform destroy --auto-approve`. 

{% highlight console %}
Destroy complete! Resources: 9 destroyed.
{% endhighlight %}

Let's take this to [Part 2](/lab2), where there will be more Terraform code for the  workstations then some Ansible snippets to automate our whole lab deployment. 