---
layout: post
title:  "Azure AD with Azure Cloud Shell"
date:   2021-05-17 16:21:21 -0400
categories: [Software]
tags: azure
permalink: /cloud-shell
---

This is a quick tutorial on how to create new accounts in Azure Active Directory (AAD) using the Azure Cloud Shell. 

There's 3 ways to manage users in Azure. 
- Azure Portal 
- Azure Powershell
- Azure CLI

As much as I like the Azure graphical user interface, I much prefer writing scripts. The idea is to create code that you can reuse and is repeatable, quite similar to creating a template. Using this method to programatically create accounts significantly reduces human error. 

I won't be showing you how to create an account with using the Azure Portal because that's boring right? Instead we'll use Powershell and CLI (Bash) which is built in to [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview) to create an account.

## **Create an account using Powershell**

1. Open up the Azure Cloud Shell.
![Image](/assets/images/cloud_shell/cloudshell1.png)

{:start="2"}
2. Select _Create storage_ to create a storage account if you haven't done so.
![Image](/assets/images/cloud_shell/cloudshell2.png)

{:start="3"}
3. The first time running Cloud Shell presents you with either _Powershell_ or _Bash_. Choose _Powershell_.
![Image](/assets/images/cloud_shell/cloudshell3.png)

{:start="4"}
4. You may toggle between both _Powershell_ and _Bash_.
![Image](/assets/images/cloud_shell/cloudshell4.png)

{:start="5"}
5. Let's run the following Powershell cmdlet

{% highlight powershell %}
# Sets a secure string password 
$SecureStringPassword = ConvertTo-SecureString -String "Password123!" 
-AsPlainText -Force
New-AzADUser -DisplayName "Billy Bob Thornton" 
-UserPrincipalName "billy@drewlearnscloudoutlook.onmicrosoft.com" 
-Password $SecureStringPassword -MailNickname billy
{% endhighlight %}
![Image](/assets/images/cloud_shell/cloudshell5.png)

{:start="6"}
6. You now should have a new user _"Billy Bob Thornton"_

![Image](/assets/images/cloud_shell/cloudshell6.png)

## **Create an account using CLI**

1. Open the _Cloud Shell_ and choose _Bash_.
![Image](/assets/images/cloud_shell/cloudshell4.png)

{:start="2"}
2. Run the following command

{% highlight bash %}
az ad user create --display-name "Bruce Willis" --password "Password123!" 
--user-principal-name bruce.willis@drewlearnscloudoutlook.onmicrosoft.com
{% endhighlight %}

{:start="3"}
3. You now should have a new user _"Bruce Willis"_
![Image](/assets/images/cloud_shell/cloudshell7.png)

## **More Switches and commands**

If you'd like explore more commands check out the following links here:

- [Azure Powershell Documentation](https://docs.microsoft.com/en-us/powershell/module/az.resources/?view=azps-5.9.0&viewFallbackFrom=azps-2.0.0#active_directory)
- [Azure CLI Documentation](https://docs.microsoft.com/en-us/cli/azure/ad/user?view=azure-cli-latest)