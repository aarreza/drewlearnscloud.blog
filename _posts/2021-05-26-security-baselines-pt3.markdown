---
layout: post
title:  "A Cloud Security Baseline Part 3"
date:   2021-05-26 16:21:21 -0400
categories: [Software]
tags: azure security
permalink: /azure-security3
---

In the last two posts, we covered the following: 

- ~~Identity & Access Management~~
- ~~Azure Security Center~~
- ~~Storage Accounts~~
- Azure SQL Database
- Logging and monitoring
- Networking
- Virtual Machines
- Other operational controls (Azure Key Vault, resource locks) 

If you are starting here go check out [Part 1](azure-security1) and [Part 2](azure-security2) first then come right back. If not carry on!

## **Azure SQL Database Security Baseline**

**Level 1 Recommendations**

I. _Enable auditing_

1. Log in to your Azure portal

2. Go to SQL databases

3. Create or select an existing database

4. Under Security, select Auditing

5. Ensure that Auditing is set to On

![Image](/assets/images/az_sec_baselines/azsec16.png)

II. _Enable Azure Defender on SQL database_

1. Log in to your Azure portal

2. Go to SQL databases

3. Create or select an existing database

4. Under Security, Select Security Center

![Image](/assets/images/az_sec_baselines/azsec17.png)

{:start="5"}
5. Configure the security recommendations and assessment from the Security Center for SQL servers

![Image](/assets/images/az_sec_baselines/azsec18.png)

III. _Configure audit retention for more than 90 days_

1. Log in to your Azure portal

2. Go to SQL databases

3. Create or select an existing database

4. Under Security, select Auditing

5. Ensure Retention (days) is greater than 90 days.

![Image](/assets/images/az_sec_baselines/azsec19.png)

## **Logging and Monitoring Security Baseline**

**Level 1 Recommendations**

I. _Create the following network alerts_

1. Log in to your Azure portal

2. Go to Monitor, and then select Alerts.

- **_Creating a policy assignment_**

![Image](/assets/images/az_sec_baselines/azsec20.png)

- **_Creating, updating, or deleting a Network Security Group_**

![Image](/assets/images/az_sec_baselines/azsec21.png)

- **_Creating or updating an SQL Server firewall rule_**

![Image](/assets/images/az_sec_baselines/azsec22.png)

## **Networking Security Baseline**

**Level 1 Recommendations**

I. _Restrict RDP and SSH access from the Internet_

1. Log in to your Azure portal

2. Go to Virtual Machines and choose a VM

3. Verify that the Inbound Port Rules does not have a rule for RDP, for example: `port=3389, protocol = TCP, Source = Any or Internet`

![Image](/assets/images/az_sec_baselines/azsec23.png)

II. _Enable Network Watcher_

1. Login to your Azure portal

2. Go to Network Watcher

4. Select the Regions to expand it and select your Region

5. Select Enable Network Watcher

![Image](/assets/images/az_sec_baselines/azsec24.png)

## **Virtual Machine Security Baseline**

**Level 1 Recommendations**

I. _A VM agent must be installed and enabled for Log Analytics in Azure Security Center_

1. Log in to your Azure portal

2. Go to Security Center and select Pricing & Settings

3. Choose your subscription

4. Select Auto Provisioning and Turn Log Analytics agent for Azure VMs

![Image](/assets/images/az_sec_baselines/azsec25.png)

II. Ensure that OS disk are encrypted

1. Log in to your Azure portal

2. Go to the Virtual Machines

3. For each VM, go to Advisor Recommendations

4. Follow the remediation steps to Encrypt your VMs which can be found from [Microsoft documentation](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/disk-encryption-overview)

![Image](/assets/images/az_sec_baselines/azsec26.png)

III. _Ensure that the OS patches for the VMs are applied_

1. Log in to your Azure portal

2. Go to Security Center, choose Recommendations

3. Ensure to complete the recommendations under Apply system updates

![Image](/assets/images/az_sec_baselines/azsec27.png)

## **Other Services Security Baseline**

**Level 1 Recommendations**

We're down to the home stretch, the following recommendations include considerations in Azure Key Vault and using Azure resource locks

I. _Set an expiration date on all keys in Azure Key Vault_

1. Log in to your Azure portal

2. Go to Key Vaults

3. Create or select a Key Vault

4. For each key vault, select Keys

5. Ensure that each key in the vault has an Expiration Date set

![Image](/assets/images/az_sec_baselines/azsec28.png)

II. _Set an expiration date on all secrets in Azure Key Vault_

1. Log in to your Azure portal

2. Go to Key Vaults

3. Create or select a Key Vault

4. For each key vault, select Secrets

![Image](/assets/images/az_sec_baselines/azsec29.png)

III. _Set resource locks for mission-critical Azure resources_

1. Log in to your Azure portal

2. Go to Resource Groups, select the resource group or subscription you want to lock

3. Go to Locks and select Add

4. Provide a lock name and choose the type of lock, either `CanNotDelete` or `ReadOnly`. Provide notes on the type of lock

That's it you've made it to the end of this series. I hope by now you'll consider some of the security recommendations here and I hope to see you next time!
