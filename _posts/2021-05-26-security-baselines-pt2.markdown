---
layout: post
title:  "A Cloud Security Baseline Part 2"
date:   2021-05-26 16:21:21 -0400
categories: [Software]
tags: azure security
permalink: /azure-security2
---

Continuing on to establish a security baseline from [Part 1](/azure-security1) of this series. Let's dive right in to Part 2.

## **Azure Security Center**

**Level 1 Recommendations**

I. _Enable automatic provision of a monitoring agent_

Enabling automatic provisioning installs the Microsoft Monitoring Agent on all supported Azure VMs and any new ones created. 

1. Log in to your Azure portal

2. Select Security Policy on the Security Center main menu

3. Under Getting Started, choose Install Agents

![Image](/assets/images/az_sec_baselines/azsec11.png)

II. _Enable System Updates_

1. Log in to your Azure portal

2. Select Security Policy on the Security Center main menu 

3. Choose a subscription group for the list

4. Click on ASC default and find _System updates should be installed on your machines_

5. ASC agent has not been deployed to a VM or physical machine so the message AuditIfNotExists appears. AuditIfNotExists enables auditing on resources that match the if condition

![Image](/assets/images/az_sec_baselines/azsec12.png)

{:start="6"}
6. Go through the rest of the default policies that are enabled by default. You can find security policies from [Microsoft documentation](https://aka.ms/ascpolicies).

## **Create an Azure storage accounts baseline**

**Level 1 Recommendations**

I. _Require security-enhanced transfers_

1. Log in to your Azure portal

2. Go to Storage Accounts under Azure services

3. Select an existing storage account

4. Under Setting, select Configuration

5. Set Secure Transfer required to Enabled

![Image](/assets/images/az_sec_baselines/azsec13.png)

II. _Enable binary large object (blob) encryption_

1. Log in to your Azure portal

2. Go to Storage Accounts under Azure services

3. Select an existing storage account

4. Under Security & Networking, select Encryption

5. By default, Azure Storage encryption is enabled for all new and existing storage accounts and cannot be disabled

III. _Periodically regenerate storage access keys_

1. Open up the Azure Cloud shell 

2. Run the following command using Azure Cloud Shell. Check out my [post](/cloud_shell) on Azure Cloud Shell if you have not used it before. 

- **Replace `<resource-group>` with the name of the Resource Group belongs to**
- **Replace `<storage-account>` with the name of your Storage Account**


{% highlight bash %}
az storage account keys renew \
--resource-group <resource-group> \
--account-name <storage-account>
--key primary
{% endhighlight %}

**_Make sure to change the references in your code to use the new primary access key_**

IV. _Require Shared Access Signature (SAS) tokens to expire within an hour_

1. Log in to your Azure portal

2. Go to Storage Accounts under Azure services

3. Select an existing storage account

4. Under Security & Networking, select Shared Access signature

5. Choose one or more Allowed resource types

6. Set the Start and expiry date/time

7. Set Allowed protocols to HTTPS only

![Image](/assets/images/az_sec_baselines/azsec14.png)

V. _Enable Azure Files encryption_

1. Log in to your Azure portal

2. Go to Storage Accounts under Azure services

3. Select an existing storage account

4. Under Security & Networking, select Encryption

5. By default, Storage service encryption is on and protects your data at rest using Microsoft-managed keys. You also have the option to use your own keys through Key Vault or Key [URI](https://docs.microsoft.com/en-us/azure/storage/common/customer-managed-keys-configure-key-vault?tabs=portal#configure-encryption-with-customer-managed-keys). 

VI. _Require only private access to blob containers_

1. Log in to your Azure portal

2. Go to Storage Accounts under Azure services

3. For each storage account, go to Containers under Data Storage

4. Ensure that Public access level is set to Private

![Image](/assets/images/az_sec_baselines/azsec15.png)

Stay tuned for more security baselines settings which is going to be in [Part 3](/azure-security3) of this series.