---
layout: post
title:  "It's just a lab in the cloud - Part 3"
date:   2021-04-23 16:21:21 -0400
categories: [Software]
tags: [azure, terraform, ansible]
permalink: /lab3
---

If you have made it this far, kudos to you! We're on the final stretch. Let's review what we've done so far. 

- [Part 1](/lab1) - Set up our environment with Terraform, provisioned our domain controller
- [Part 2](/lab2) - Set up our environment with Ansible, made our first Ansible playbook, and provisioned our workstations

In this section, we will use Ansible again to bind our workstations to the domain controller, all done automatically!

## **Join workstations to the domain with Ansible**

Ansible has a built in handy module, [win_domain_membership](https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_domain_membership_module.html), which allows management of the domain membership status of a machine. All we need to do is therefore to map our `kind: workstations` tag in the dynamic Ansible inventory and write the appropriate playbook to ensure workstations are domain-joined. 

1. Let's go to our _ansible_ folder and add `workstations: "tags.kind == 'workstation'"` to our `inventory_azure_rm.yml` file. 

{% highlight yaml %}
# Place in ansible/inventory_azure_rm.yml
plugin: azure_rm
auth_source: cli
include_vm_resource_groups:
- aa-identity-lab
conditional_groups:
  # Place every VM with the tag "kind" == "domain_controller" in the "domain_controllers" Ansible host group
  domain_controllers: "tags.kind == 'domain_controller'"
  # Place every VM with the tag "kind" == "workstations" in the "workstations" Ansible host group
  workstations: "tags.kind == 'workstation'"
keyed_groups:
- prefix: tag
  key: tags
{% endhighlight %}

Remember we tagged our vm's in `domain-controller.tf` and `workstations.tf`and we're simply asking Ansible to look for these in the inventory so we can configure them. 

{:start="2"}
2. Create a new playbook `workstations.yml`, which will bind our workstations to the domain controller. 

{% highlight yaml %}
# ansible/workstations.yml
---
- name: Configure workstations
  hosts: workstations
  vars:
    # Note: these should ideally placed into a shared variable file (such as group_vars/all)
    # to avoid duplication with the DC playbook
    domain_name: drewlearnscloud.blog
    domain_admin: drew
  vars_prompt:
  - name: domain_admin_password
    prompt: "Domain admin password"
  tasks:
  - name: Set DC as DNS server
    win_dns_client:
      adapter_names: '*'
      ipv4_addresses: "{{ hostvars[groups['domain_controllers'][0]].private_ipv4_addresses }}"

  - name: Ensure workstation is domain-joined
    win_domain_membership:
      dns_domain_name: "{{ domain_name }}"
      hostname: "{{ ansible_env.COMPUTERNAME }}"
      domain_admin_user: "{{ domain_admin }}@{{ domain_name }}"
      domain_admin_password: "{{ domain_admin_password }}"
      state: domain
    register: domain_state

  - name: Reboot machine if it has just joined the domain
    win_reboot: {}
    when: domain_state.reboot_required
{% endhighlight %}

{:start="3"}
3. Enter the following command to run the Ansible playbook. You will be asked to enter your _SSH password_ which is the random password generated for the workstations and domain password will be the random password generated for the domain controller. 

{% highlight console %}
# Command to add dc to domain
ansible-playbook workstations.yml --inventory inventory_azure_rm.yml \
    -e AZURE_RESOURCE_GROUPS=aa-identity-lab \
    --user localadmin --ask-pass
{% endhighlight %}

Result:
{% highlight console %}
SSH password:
Domain admin password:

PLAY [Configure workstations] **************************************************

TASK [Gathering Facts] *********************************************************
ok: [workstation-2_de6a]
ok: [workstation-1_1927]

TASK [Set DC as DNS server] ****************************************************
ok: [workstation-1_1927]
ok: [workstation-2_de6a]

TASK [Ensure workstation is domain-joined] *************************************
changed: [workstation-1_1927]
changed: [workstation-2_de6a]

TASK [Reboot machine if it has just joined the domain] *************************
changed: [workstation-1_1927]
changed: [workstation-2_de6a]

PLAY RECAP *********************************************************************
workstation-1_1927         : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
workstation-2_de6a         : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
{% endhighlight %}

{:start="4"}
4. Look at that! Our workstations are now domain joined. 

![Image](/assets/images/azure_idp_lab/azuretf5.png)

![Image](/assets/images/azure_idp_lab/azuretf6.png)

## **Putting it all together**

We now have our Terraform blocks and we've used a couple of Ansible playbooks but we're still manually entering a total of 3 commands. Let's automate this further!

- `terraform apply` - Create our resources
- `ansible-playbook domain-controllers.yml --inventory inventory_azure_rm.yml \
  -e AZURE_RESOURCE_GROUPS=aa-identity-lab \
  --user drew --ask-pass` - Promote our domain
- `ansible-playbook workstations.yml --inventory inventory_azure_rm.yml \
    -e AZURE_RESOURCE_GROUPS=aa-identity-lab \
    --user localadmin --ask-pass` - Add our workstations

We'll combine all of these together and the goal is to run a single command to make it all run. Now this is where the magic happens!

We will use the [local-exec](https://www.terraform.io/docs/language/resources/provisioners/local-exec.html) provisioner in Terraform to help us run local commands when a resource is created and we'll use it to automatically run our playbooks in Ansible. 

1. Modify the `domain-controller.tf` and add the following code.

{% highlight terraform %}
resource "azurerm_virtual_machine" "domain_controller" {
  name = "domain-controller"
  
  # ...
  provisioner "local-exec" {
    working_dir = "${path.root}/../ansible"
    command = "ansible-playbook domain-controllers.yml --user drew -e ansible_password=${random_password.domain_controller_password.result} -e AZURE_RESOURCE_GROUPS=${azurerm_resource_group.main.name} -v"
  }
}
{% endhighlight %}

{:start="2"}
2. Modify the `workstations.tf` and add the following code.

{% highlight terraform %}
resource "azurerm_virtual_machine" "workstation" {
  count = var.num_workstations

# ...

# you'll need to run terraform init again to reinitialize this plugin
resource "null_resource" "provision_workstation_once_dc_has_been_created" {
  provisioner "local-exec" {
    working_dir = "${path.root}/../ansible"
    command = "ansible-playbook workstations.yml --user localadmin -e domain_admin_password=${random_password.domain_controller_password.result} -e ansible_password=${random_password.workstations_local_admin_password.result} -e AZURE_RESOURCE_GROUPS=${azurerm_resource_group.main.name} -v"
  }

  # Note: the dependance on 'azurerm_virtual_machine.workstation' applies to *all* resources created from this block
  # The provisioner will only be run once all workstations have been created (not once per workstation)
  depends_on = [
    azurerm_virtual_machine.domain_controller,
    azurerm_virtual_machine.workstation
  ]
{% endhighlight %}

Note: The Ansible playbook will only be run when the resource (here, the domain controller VM) is created. You’ll still need to run it manually every time you want to add a workstation to the domain.

{:start="3"}
3. You can test out this Terraform command from start to finish with `terraform apply --auto-approve`. Remember this will take about 15-20 minutes to provision everything so be patient. 

## **Conclusion**
You've made it and thanks for following through the series! I hope you learned a few things along the way. I've wrapped all of the code up in [Github](https://github.com/aarreza/idp-demolab) to share and for your reference. 