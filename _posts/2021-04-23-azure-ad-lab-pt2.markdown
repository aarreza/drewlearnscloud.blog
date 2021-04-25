---
layout: post
title:  "It's just a lab in the cloud - Part 2"
date:   2021-04-23 16:21:21 -0400
categories: [Software]
tags: [azure, terraform, ansible]
permalink: /lab2
---
If you've been following this series. We left off from [Part 1](/lab1) with our deployment of a Windows Server 2019 VM as our domain controller. 

We're going to pivot a bit from Terraform and set up our virtual environment and play with Ansible. We'll be using `virtualenv` to set up our dependencies first. This is a very important step and if you're new to software development, it can get a little _hairy_ here. So pay close attention and I'll try to break it down the best I can to get us up and running. 

## **Install Ansible**

1. Open up your Terminal and enter `brew install ansible`. 

2. Verify your installation with `ansible --version`
{% highlight console %}
ansible 2.10.7
  config file = None
  configured module search path = ['/Users/drew-dev/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/Cellar/ansible/3.2.0/libexec/lib/python3.9/site-packages/ansible
  executable location = /usr/local/bin/ansible
  python version = 3.9.4 (default, Apr  5 2021, 01:50:46) [Clang 12.0.0 (clang-1200.0.32.29)]
{% endhighlight %}

## **Install Python and pip**
1. In our project folder, make a new folder called _ansible_. 
{% highlight console %}
mkdir ansible
{% endhighlight %}

{:start="2"}
2. Next, you will need to install Python, `pip`, and `virtualenv`. Follow the guide [here](https://docs.python-guide.org/starting/install3/osx/).

3. Verify your _Python_, `pip`, and `virtualenv` installations.
{% highlight console %}
which python3
/usr/local/bin/python3

python3 --version
Python 3.9.4

which pip3
/usr/local/bin/pip3

pip3 --version
pip 21.0.1 from /Users/drew-dev/Library/Python/3.9/lib/python/site-packages/pip (python 3.9)

which virtualenv
/usr/local/bin/virtualenv

virtualenv --version
virtualenv 20.4.4 from /usr/local/lib/python3.9/site-packages/virtualenv/__init__.py
{% endhighlight %}

Note: _Before moving on to the next steps, make sure you have all of these prerequisites installed._

## **Set up our virtual environment**

1. Go to the _ansible_ folder and create a `requirements.txt` file. 

{% highlight console %}
pywinrm==0.4.1
requests==2.23.0
msrest==0.6.13
msrestazure==0.6.3
azure-cli==2.5.1
ansible==2.9.9
{% endhighlight %}

{:start="2"}
2. `cd ..` to go back to the project root folder and enter the following commands. You can reference my Github [repo](https://github.com/aarreza/idp-demolab) for the folder structure.

{% highlight console %}
python3 -m venv ansible/venv 
source ansible/venv/bin/activate
pip3 install -r ansible/requirements.txt
# Only run deactivate when you are finished with the lab
deactivate
{% endhighlight %}

This should create a `venv` folder inside your _ansible_ folder.

## **Configuring the domain controller**

Earlier on we allowed WinRM in our domain controller security groups. We will use the [Azure dynamic inventory](https://docs.ansible.com/ansible/latest/plugins/inventory/azure_rm.html) plugin, which will allow us to automatically target our virtual machines without hardcoding any IP address in the Ansible configuration.

1. Create a new folder in _ansible_ `group_vars` and create a file named `all`. You can do create a filename without an extension by doing, `echo "ansible_connection: winrm\nansible_winrm_transport: ntlm\nansible_winrm_scheme: http\nansible_winrm_port: 5985" > all`.

{% highlight console %}
# ansible/group_vars/all
ansible_connection: winrm
ansible_winrm_transport: ntlm
ansible_winrm_scheme: http
ansible_winrm_port: 5985
{% endhighlight %}

{:start="2"}
2. Set up a dynamic inventory YAML file with Ansible

{% highlight yaml %}
# Place in ansible/inventory_azure_rm.yml
plugin: azure_rm
auth_source: cli
include_vm_resource_groups:
- aa-identity-lab
conditional_groups:
  # Place every VM with the tag "kind" == "domain_controller" in the "domain_controllers" Ansible host group
  domain_controllers: "tags.kind == 'domain_controller'"
keyed_groups:
- prefix: tag
  key: tags
{% endhighlight %}

{:start="3"}
3. Create a new Ansible configuration file named `ansible.cfg` 

{% highlight yaml %}
# Place in ansible/ansible.cfg
[defaults]
inventory=./inventory_azure_rm.yml
nocows=1
{% endhighlight %}

## **Create our Ansible playbook**

1. Create a new file called `domain-controllers.yml`.
{% highlight yaml %}
# Place in ansible/domain-controllers.yml
---
- name: Configure domain controllers
  hosts: domain_controllers
  gather_facts: no
  vars:
    domain_name: drewlearnscloud.blog
    domain_admin: drew@drewlearnscloud.blog
    domain_admin_password: Pizza123!
    safe_mode_password: Pizza123!
  tasks:
  - name: Ensure domain is created
    win_domain:
      dns_domain_name: drewlearnscloud.blog
      safe_mode_password: Pizza123!
    register: domain_creation
  - name: Reboot if domain was just created
    win_reboot: {}
    when: domain_creation.reboot_required
  - name: Ensure domain controllers are promoted
    win_domain_controller:
      dns_domain_name: drewlearnscloud.blog
      domain_admin_user: drew@drewlearnscloud.blog
      domain_admin_password: Pizza123!
      safe_mode_password: Pizza123!
      state: domain_controller
      log_path: C:\Windows\Temp\promotion.txt
    register: dc_promotion
  - name: Reboot if server was just promoted to a domain controller
    win_reboot: {}
    when: dc_promotion.reboot_required
{% endhighlight %}

{:start="2"}
2. If you still have the lab running in terraform, go to your _ansible_ directory and run the Ansible playbook we created with the command below, this should take about 10 mininutes. It installs Active Directory, promotes the server to a domain controller, and automatically reboots it.  

{% highlight console %}
# Command to add dc to domain
ansible-playbook domain-controllers.yml --inventory inventory_azure_rm.yml \
  -e AZURE_RESOURCE_GROUPS=aa-identity-lab \
  --user drew --ask-pass
{% endhighlight %}

{:start="3"}
3. Once the Ansible is complete, the output should be:
{% highlight console %}
PLAY RECAP *********************************************************************
domain-controller_bd16     : ok=4    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
{% endhighlight %}

## **Add our workstation(s)**

We'll now add our workstations using Terraform. Creating a Windows 10 VM is very similar to how we created our domain. I'll be using the latest Windows 10 20H2 for this image. If you'd like to customize the image, you can use the Azure CLI command `az vm image list --publisher MicrosoftWindowsServer --all -o table > windesk.txt`. This will output the list in a filename called `windesk.txt`.

1. If you'd like to modify the number of workstations, we can use variables in Terraform. Go back to your terraform root folder and create a file name called `variables.tf` and put the following code. 

{% highlight terraform %}
# Let's us increase or decrease the number of workstations
variable "num_workstations" {
  description = "Number of workstations to create"
  type = number
  default = 1
}
{% endhighlight %}

{:start="2"}
2. Similar to how we set up our domain controllers, we'll need to declare a few things:
- A _subnet_ for our workstations
- x1 _network interface_ per workstation
- A _private IP_ subnet
- x1 _public IP_ per workstation
- A _network security group_

Create a new file called `workstations-net.tf` inside our Terraform directory.

{% highlight terraform %}
# The subnet for workstation
resource "azurerm_subnet" "workstations" {
  name                 = "workstations"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.11.0/24"]
}

# Create 1 network interface per workstation
resource "azurerm_network_interface" "workstations_nic" {
  count = var.num_workstations
  name                = "workstation-${count.index + 1}-nic"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  ip_configuration {
    name                          = "static"
    subnet_id                     = azurerm_subnet.workstations.id
    private_ip_address_allocation = "Static"
    # Private IP subnet
    private_ip_address            = cidrhost("10.0.11.0/24", 100 + count.index)
    public_ip_address_id          = azurerm_public_ip.workstation[count.index].id
  }
}

# Create 1 public IP per workstation
resource "azurerm_public_ip" "workstation" {
  count = var.num_workstations
  name                    = "workstation-${count.index + 1}-public-ip"
  location                = azurerm_resource_group.main.location
  resource_group_name     = azurerm_resource_group.main.name
  allocation_method       = "Static"
}

# Network security group for workstation(s)
resource "azurerm_network_security_group" "workstations" {
  name                = "workstations-nsg"
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

# Associate our network security group with the NIC of our workstations
resource "azurerm_network_interface_security_group_association" "workstations" {
  count = var.num_workstations
  network_interface_id      = azurerm_network_interface.workstations_nic[count.index].id
  network_security_group_id = azurerm_network_security_group.workstations.id
}
{% endhighlight %}

Note the following lines:

- `count = var.num_workstations` - this repeats the current resource `var.num_workstations` times
- `private_ip_address = cidrhost("10.0.11.0/24", 100 + count.index)` - uses the [cidrhost](https://www.terraform.io/docs/language/functions/cidrhost.html) function, so we can assign multiple ip addresses to each workstation starting at 10.0.11.100, 10.0.11.101, 10.0.11.102, ...

{:start="3"}
3. Create the template for the Windows 10 workstations. Make a new file called `workstations.tf`.

{% highlight terraform %}
# Windows 10 workstations
# Generate a random password and reuse it for each local admin account on workstations
resource "random_password" "workstations_local_admin_password" {
  length  = 16
  special = false
}

# Windows 10 workstations
resource "azurerm_virtual_machine" "workstation" {
  count = var.num_workstations

  name                  = "workstation-${count.index + 1}"
  location              = azurerm_resource_group.main.location
  resource_group_name   = azurerm_resource_group.main.name
  network_interface_ids = [azurerm_network_interface.workstations_nic[count.index].id]
  vm_size               = "Standard_D1_v2"

  storage_image_reference {
    publisher = "MicrosoftWindowsDesktop"
    offer     = "Windows-10"
    #sku       = "19h1-pron"
    sku       = "20h2-pron"
    version   = "latest"
  }

  delete_os_disk_on_termination = true
  storage_os_disk {
    name              = "workstation-${count.index + 1}-os-disk"
    create_option     = "FromImage"
  }

  os_profile {
    computer_name  = "WORKSTATION-${count.index + 1}"
    admin_username = "localadmin"
    admin_password = random_password.workstations_local_admin_password.result
  }
  os_profile_windows_config {
      winrm {
        protocol = "HTTP"
      }
  }

  # Explicit depend on workstations nic, so when we terraform destroy, there will be no errors
  depends_on = [
    azurerm_network_interface_security_group_association.workstations
  ]

  tags = {
    kind = "workstation"
  }
}
{% endhighlight %}

{:start="4"}
4. Before we run this, let's add the following code to the `outputs.tf` file so that we can display the public ip's of the workstations. 

{% highlight terraform %}
# Displays workstation public ip(s)
output "workstations_public_ips" {
  value = azurerm_public_ip.workstation.*.ip_address
}

# Displays workstations password(s)
output "workstations_local_admin_password" {
  value = nonsensitive(random_password.workstations_local_admin_password.result)
}
{% endhighlight %}

{:start="5"}
5. Run `terraform apply --auto-approve` and you should have your workstations added. The amount of time it takes to provision everything is about 10 minutes. 

Note: You can easily scale the number of workstations you would like to add by running: 

{% highlight terraform %}
# Scale up!
$ terraform apply -var 'num_workstations=5'
# Scale down and remove 2 workstations
$ terraform apply -var 'num_workstations=3'
{% endhighlight %}

You can play around with these Terraform commands and see how easily you can now provision and deprovision resources in Azure. Don't forget to `terraform destroy --auto-approve` to tear down your lab infrastructure and always double check your Resource Groups in Azure to make sure everything is gone when you're finished with the lab.

See you on the next post ([Part 3](/lab3)) and we'll glue everything together using Ansible!