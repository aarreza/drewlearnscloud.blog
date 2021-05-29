---
layout: post
title:  "A Cloud Security Baseline Part 1"
date:   2021-05-26 16:21:21 -0400
categories: [Software]
tags: azure security
permalink: /azure-security1
---

Managing a cloud infrastructure or multi-cloud environments is quite an undertaking. As an Infrastructure and/or Operations person, responsible for various different things, you've got a lot on your plate. You may take care of issues with the on-premise networks in multiple sites one day, or you have to ensure that your cloud services are always available, or sometimes deal with that new user who can't seem to connect to his cloud SQL database. Does it seem like you are simply fighting daily fires everyday and it doesn't seem to end? Is your organization proactively looking at ways to keep your environment secure?

In this blog we'll be looking at creating a cloud security baseline. I aim to establish actionable security configurations to consider for your Azure environment. Now a _fair warning_, take these as prescriptive recommendations and enable/disable them at your own discretion as this may have an impact to your existing tenant and dependent users, specially in production. 

## **What is a security baseline?**

A security baseline is an established minimum set of security controls that a customer (of Azure) is responsible for. Azure out-of-the-box does not come with automatic security settings and it is the customer's responsibility to ensure that all Azure services being consumed have at least the minimum base security settings applied. 

## **Who is the Center for Internet Security?**

The Center for Internet Security (CIS) is a non-profit organization that focuses on safeguarding public and private organizations which Microsoft has partnered with. Microsoft's cybersecurity group and CIS have worked together to help establish a security baseline within the Azure platform.

## **Implementation Models**

CIS recommends two implementation levels
- **Level 1 - Recommended minimum security settings**
    - **_should be configured on all systems_**
    - **_cause little or no interruption to services or reduced functionality_**
- **Level 2 - Recommendations for highly secured environments**
    - **_may result in loss or reduced functionality_**

## **Technology areas**

These are the identified areas we'll be looking at:
- Identity & Access Management 
- Azure Security Center
- Storage Accounts
- Azure SQL Database
- Logging and monitoring
- Networking
- Virtual Machines
- Other operational controls (Azure Key Vault, resource locks) 

## **Identity and Access Management (IAM)**

**Level 1 Recommendations**

I. _All non-Administrators should not have access due to the sensitive data and rules of least privilege._

1. Log in to your Azure portal

2. Go to Azure Active Directory > Users > User settings

3. Toggle _Yes_ for Restrict access to Azure AD administration. Setting this value to Yes restricts all non-administrators from accessing any Azure AD data in the administration portal, but does not restrict such access using PowerShell or another client such as Visual Studio.
![Image](/assets/images/az_sec_baselines/azsec1.png)

II. Ensure that no guest users exist, or if the business requires guest users, ensure that they have limited access only.

1. Log in to your Azure portal

2. Go to Azure Active Directory > Users

3. Filter the Guest Users

![Image](/assets/images/az_sec_baselines/azsec2.png)

III. _Enable Self-Service Password Reset, Set the password authentication options, Set the Notification Options_

- Notify users on password resets - Level 1
- Require two methods to reset passwords - Level 1
- Notify all admins when other admins reset passwords - Level 2

1. Log in to your Azure portal

2. Go to Azure Active Directory > Users

3. Select _Password reset_. Sign up for Azure AD Premium trial or enable it.

![Image](/assets/images/az_sec_baselines/azsec3.png)

{:start="4"}
4. Choose one of the two license options to enable Password Reset. 

![Image](/assets/images/az_sec_baselines/azsec4.png)

{:start="5"}
5. Go to the _Authentication methods_ blade and chose the number of methods required to reset a password.

![Image](/assets/images/az_sec_baselines/azsec5.png)

{:start="6"}
6. Go to the _Notifications_ blade and choose one of the options

![Image](/assets/images/az_sec_baselines/azsec6.png)

IV. Establish an interval for reconfirming user authentication methods

1. Log in to your Azure portal

2. Go to Azure Active Directory > Users

3. Go to Password Reset

4. Go to Registration

5. Choose an interval for the Number of days before users are asked to re-confirm their authentication. Make sure it is not set to 0 and the default is 180 days.

![Image](/assets/images/az_sec_baselines/azsec7.png)

**Level 2 Recommendations**

I. _Members and guests can invite_

1. Log in to your Azure portal

2. Go to Azure Active Directory > Users > User Settings

3. Go to External users > External Collaboration Settings

4. Under Guest invite settings, choose _Only users assigned to specific admin roles can invite guest users_

![Image](/assets/images/az_sec_baselines/azsec8.png)

II. _Disable self-service group management_

1. Log in to your Azure portal

2. Go to Azure Active Directory > Groups > General settings

3. Ensure that Self-service group management enabled is set to No.
![Image](/assets/images/az_sec_baselines/azsec9.png)

III. _Disable Application options - Allow users to register apps_

1. Log in to your Azure portal

2. Go to Azure Active Directory > Users > User settings

3. Ensure that App registrations is set to No.

![Image](/assets/images/az_sec_baselines/azsec10.png)

This would be the end of Part 1 of this blog post. We will continue our security baseline recommendations in [Part 2](/azure-security2). 