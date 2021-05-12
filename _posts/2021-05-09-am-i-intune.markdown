---
layout: post
title:  "Am I Intune?"
date:   2021-05-09 16:21:21 -0400
categories: [Software]
tags: mdm
permalink: /mdm
---

System admins like myself are constantly being pulled in different directions in IT no matter where you work. Wouldn't it be nice for your end users to have a great experience with their devices, while freeing up your time for other more _important_ or _meaningful_ tasks? Well, the answer is a definite _"YES"_ in my world, so today we will look at a Mobile Device Management solution to make this possible.

## **Solution**

We'll be looking at Microsoft's cloud solution for device and app management, [Intune](https://docs.microsoft.com/en-us/mem/intune/fundamentals/what-is-intune) or now called [Microsoft Endpoint Manager](https://www.microsoft.com/en-ww/microsoft-365/microsoft-endpoint-manager). IT admins who used to manage Windows devices using traditional tools like [SCCM](https://en.wikipedia.org/wiki/Microsoft_System_Center_Configuration_Manager), [WDS](https://docs.microsoft.com/en-us/windows/win32/wds/windows-deployment-services-portal), and [MDT](https://en.wikipedia.org/wiki/Microsoft_Deployment_Toolkit) would feel right at home. Intune/Microsoft Endpoint Manager is a cloud-first solution to organizations looking to enhance their compliance, security, and empower their users to be able to work remotely from anywhere while maintaining full/partial control of devices or personal Bring-Your-Own-Devices (BYOD).

## **Goal**

I want to be as practical as possible and if you follow along and this is your first time setting up for your organization, you'll have a functional baseline for Intune with Autopilot.

Our goals: 
- Understand the licensing model required to get Intune
- Bootstrap and set up a new tenant
- Set up groups in Azure AD
- Set up Company Branding
- Enable Autopilot
- Test deployment of Autopilot with Hyper-V

Note: We are going to a fully cloud managed platform so we will not be doing local domain-joined or Group Policy devices as well as co-management with SCCM, for the sake of decreasing the **complexity** within our environment and sticking to our main objective which is to have a nimble and mobile experience for our end users. 

## **Licensing for Intune**

In our case, I have already signed up for an Intune [trial](https://signup.microsoft.com/create-account/signup?OfferId=40BE278A-DFD1-470a-9EF7-9F2596EA7FF9&dl=INTUNE_A&ali=1&products=40BE278A-DFD1-470a-9EF7-9F2596EA7FF9#0%20). If you haven't done so, follow the link to set up your account and choose a domain name.

As an IT administrator, one of the first things you'll be faced with when setting up your own Intune production or test environments is understanding Microsoft's licensing. Here's a list that outlines the licensing requirements for [Intune](https://docs.microsoft.com/en-us/mem/intune/fundamentals/licenses).

In our test environment I've signed up for the following 30 day trial(s):

![Image](/assets/images/mdm_intune/intune1.png)

## **Setting up a new tenant**

1. Once you have set up your account, log in to [admin.microsoft.com](http://admin.microsoft.com/) to view your Admin Page for Endpoint Manager. 
![Image](/assets/images/mdm_intune/intune2.png)

{:start="2"}
2. The first thing you should do if you're setting this up in your production environment, is to add and verify your domain ownership. Instructions vary depending on your hosting provider. 
![Image](/assets/images/mdm_intune/intune3.png)

Since this is just a test tenant, I am leaving mine to _drewlearnscloudtest.onmicrosoft.com_.

{:start="3"}
3. Next, create a couple of accounts for testing. You will see one main already created which is your primary Global Admin account used to sign up.  
![Image](/assets/images/mdm_intune/intune4.png)

{:start="4"}
4. Create the account without a license as we will assign the licenses through Azure AD Groups in the next step.
![Image](/assets/images/mdm_intune/intune5.png)

{:start="5"}
5. We now have our an account _loki@drewlearnscloudtest.onmicrosoft.com_ showing as an _Unlicensed_ user, which is exactly what we wanted.
![Image](/assets/images/mdm_intune/intune6.png)

## **Setting up our Azure AD groups**
1. Head on over to [Azure Active Directory](https://portal.azure.com/) and go to Groups
![Image](/assets/images/mdm_intune/intune7.png)

I created 2 separate Azure AD groups that bundles the licenses together. You can also simply create just one group that will apply all of the licenses altogether. Add the following users you've created in Step 3 to the groups. 

**Intune_O365_License**
![Image](/assets/images/mdm_intune/intune8.png)

**E5_P2_Trial**
![Image](/assets/images/mdm_intune/intune9.png)

## **Setting up Company Branding**
The Autopilot feature requires the set up of Company Branding. So it is important to not skip this step.

1. Head on back to the Azure AD blade and select _Company Branding_.
![Image](/assets/images/mdm_intune/intune10.png)

{:start="2"}
2. To set up a new branding click _Configure_ and fill in the following information and _Save_. 

_By the way, these are not my own branding and they are just images that I grabbed quickly from Google. Feel free to create and add your own!_

![Image](/assets/images/mdm_intune/intune11.png)
![Image](/assets/images/mdm_intune/intune12.png)

## **Enable MDM and MAM**

These options give you the ability to automatically enroll your devices. _Note that this is where you'll probably get asked to sign up for the Premium licenses for Azure AD (P1 or P2) if setting up for the first time._ P1 licensing is typically associated with O365 E3 license and P2 is associated with O365 E5 licenses. The pricing model breakdown are found here, for [Azure AD](https://azure.microsoft.com/en-us/pricing/details/active-directory/) and [O365](https://www.microsoft.com/en-ca/microsoft-365/compare-microsoft-365-enterprise-plans).

For this tutorial, I went ahead with Azure AD P2 and M365 E5 licenses.

_On a side note, a lot of this push from Microsoft to use MDM will change the way organizations handle user data. If your environment still relies on network shares or if you still support or should I say "hold the hand" of the majority of your end users when they lose their data, you have to start moving away from this model. IT should be putting the focus on using the technology that is out there already like OneDrive, SharePoint Online, known folder redirection to abide by the security compliance policies which is standard now in IT._

1. Once you have enabled the correct licensing, you should now be able to set up MDM and MAM. Go back to the Azure AD blade and select _Mobility (MDM and MAM)_.
![Image](/assets/images/mdm_intune/intune13.png)

{:start="2"}
2. Click on Microsoft Intune and turn on the following settings. Click _Save_.

![Image](/assets/images/mdm_intune/intune14.png)

{:start="3"}
3. Click on _Microsoft Intune Enrollment_ and turn on the following settings. Click _Save_.

![Image](/assets/images/mdm_intune/intune15.png)

>**Note: Any Windows 10 1609 versions or older cannot be enrolled in the new version of Intune.**

{:start="4"}
4. Go to the new portal of [Microsoft Endpoint Manager](https://endpoint.microsoft.com/) which is [https://endpoint.microsoft.com/](https://endpoint.microsoft.com/). We will now setup Autopilot.

5. Go to Devices > Windows > Windows Enrollment > Deployment Profiles

{:start="5"}
![Image](/assets/images/mdm_intune/intune16.png)

{:start="6"}
6. Create a New Profile > Select Windows PC and choose the following settings.

![Image](/assets/images/mdm_intune/intune17.png)

## **Test the deployment**
Here's the fun part. Since we now have Autopilot, we can test our deployment! We will be using:
- Hyper-V with a VM running Windows 10
- Powershell

1. If you don't already have a Windows 10 iso, you may download it from [Microsoft's website](https://www.microsoft.com/en-us/software-download/windows10ISO). 

2. We will convert an ISO file to _.vhdx_ using a couple of Powershell commands. Let's install the cmdlet. 
{% highlight powershell %}
Install-Module -Name Convert-WindowsImage
{% endhighlight %}

{:start="3"}
3. Make a folder called C:\Temp and put the Windows 10 ISO file in it.

{:start="4"}
4. Open up your Powershell command prompt as _Admin_ and run the following commands.
{% highlight powershell %}
# Allow Powershell scripts to run
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned

# Convert ISO to VHDX
Convert-WindowsImage -SourcePath "C:\Temp\Win10_20H2_v2_English_x64.iso" 
–PassThru -VHDFormat "VHDX" -Edition "Windows 10 Pro" -SizeBytes 50GB 
-DiskLayout "UEFI" -VHDPath "C:\Temp\Win10_20H2_v2_English_x64.vhdx"
{% endhighlight %}

{:start="5"}
5. We're going to need to pull our Autopilot configuration from Intune. To do so, run the following Powershell commands. 

{% highlight powershell %}
Install-Module -name WindowsAutoPilotIntune
Connect-MSGraph
{% endhighlight %}

{:start="6"}
6. Follow the prompts and enter your UPN to sign in. You will be prompted with a permissions requested screen (This ties in with the Microsoft Graph). Check the Consent on behalf of my organization box and select _Accept_.

{:start="7"}
7. Run the following Powershell commands to export the AutopilotConfigurationFile.json from Intune to save it to your local C:\ drive.
{% highlight powershell %}
$apppolicies = Get-AutoPilotProfile
$apppolicies | ConvertTo-AutoPilotConfigurationJSON | Out-File
"C:\Temp\AutopilotConfigurationFile.json" -Encoding ascii
{% endhighlight %}

{:start="8"}
8. At this point we should be able to load up the Autopilot JSON file from Intune and add it to our _.vhdx_ file. Mount the `Win10_20H2_v2_English_x64.vhdx` file and go to drive E:\. Navigate to E:\Windows\Provisioning\Autopilot and drop the _AutopilotConfigurationFile.json_ file. 

![Image](/assets/images/mdm_intune/intune18.png)

{:start="9"}
9. Unmount the _.vhdx_ file from drive E:/. Load up your Hyper-V Manager and create a new ISO and point its Hard Drive to use `Win10_20H2_v2_English_x64.vhdx` file. 

![Image](/assets/images/mdm_intune/intune19.png)

{:start="10"}
10. Now sit back and relax as this process might take a few minutes but you should now be able to follow the setup and see the Out-of-Box Experience with Intune Autopilot.

![Image](/assets/images/mdm_intune/intune20.png)

Voila! You now have a working bare bones Autopilot configuration. 

You can find more information on the different types of Autopilot deployments [here](https://docs.microsoft.com/en-us/mem/autopilot/add-devices), such as manually registering your devices in Intune. 