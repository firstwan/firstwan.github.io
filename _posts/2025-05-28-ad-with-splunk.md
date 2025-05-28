---
title: Active Directory with Splunk
description: >-
  Setup a Active Directory environment and Splunk aggregate logs data and detect unauthorized login with Vultr cloud service provider.
author: first_wan
categories: [Cybersecurity, Project]
tags: [SIEM, SOC, AD, Splunk]
pin: true
---

Active Directory is a directory service developed by Microsoft for Windows domain networks. It acts as a central database and set of services that manage user accounts, computer objects, groups, policies, and other resources on the network, ensuring secure access to those resources.

## Introduction

In this walkthrough, we will setup an Active Directory (AD) environment and send the log to Splunk Security Information and Event Management (SIEM). Furthermore, we will setup an alert to detect potential unauthorized event from any IP address we not familiar.

## Overview

![ad_with_splunk_image](/blogs/ad_with_splunk/overview_1.png)

We will create 3 servers, a Windows server as AD Domain Controller, a normal Windows server that joined the AD, and a Splunk server. Those 2 Windows servers will send their Windows Event Log to Splunk server periodically.

Moreover, we will create an alert to detect any unauthorized login to the Windows servers and notify user if found any.

## Servers

We will utilize [Vultr](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbmlYX1hxamh4WHg2bHZDWUdvN2tCOUgzdGx1UXxBQ3Jtc0ttMm1INWw3b2NlRVZBQ0g0VThfbE01Z3VTU2hMa1ctT2s4YUw5UGRBNlZhSlhUMk80UVUxX2hhYTI3REpWRWJTOWNZWnNBcGNVTVZfWWN1WVFhN0t1OTQ4clRDUGFoT2lydDhKSzd5LWhfWUxKWWtPQQ&q=https%3A%2F%2Fwww.vultr.com%2F%3Fref%3D9632889-9J&v=mkx5e_UzpvI) to create and host all of the servers. Vultr is a cloud provider that offering a wide range of services, including virtual servers (VPS), cloud compute, and managed databases.

You can register an account through [Vultr](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbmlYX1hxamh4WHg2bHZDWUdvN2tCOUgzdGx1UXxBQ3Jtc0ttMm1INWw3b2NlRVZBQ0g0VThfbE01Z3VTU2hMa1ctT2s4YUw5UGRBNlZhSlhUMk80UVUxX2hhYTI3REpWRWJTOWNZWnNBcGNVTVZfWWN1WVFhN0t1OTQ4clRDUGFoT2lydDhKSzd5LWhfWUxKWWtPQQ&q=https%3A%2F%2Fwww.vultr.com%2F%3Fref%3D9632889-9J&v=mkx5e_UzpvI) to get a $300 credit into your account and use it to setup this lab. But you need to bind your credit card in order to create virtual machine in this platform.

You can use AWS, Azure, or Google Cloud Platform if you want.

### Domain Controller Server

We don’t need a lot of resource for this lab, since it this lab is more on practice and testing purpose. 

1. Click the “Deploy” button.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/server_1.png)
    
2. Select the “**Shared CPU**” and choose the location that closer to your location. I will choose Singapore for my case.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/server_2.png)
    
3. Again this lab is for practice purpose, we don’t need the backup feature. Click “Disable” on right panel and disable it.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/server_3.png)
    
4. Let select 2 CPU and 2 GB of memory for Domain Controller server.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/server_4.png)
    
5. Select “Windows Standard 2022 x64” as the OS.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/server_5.png)
    
6. Leave other as blank for now, and name it as “SOC-ADDC01”.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/server_6.png)
    
7. Leave other as default and click “Deploy” on the right panel.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/server_7.png)
    

### Normal Server

The step to create normal server is same as previous one. Go through the same step until select the size of the server. We can use a lower resources for normal server, since we won’t have much configuration on this machine.

- Select 1 CPU and 2 GB of memory for normal server.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/server_8.png)
    
- Select the “Windows Standard 2022 x64” as the OS.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/server_9.png)
    
- Name is as “SOC-ADNormal”.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/server_10.png)
    

### Splunk Server

Do the same as previous step for Splunk server, but we need a higher resources than other 2 servers. Because this Splunk machine will be aggregate the log from other 2 servers and running the alert to detect unauthorized login.

- Let select 4 CPU and 8 GB of memory for Splunk server.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/server_11.png)
    
- This server will be using different OS since it not need to join the AD. Select the “Ubuntu 22.04 x64” as the OS.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/server_12.png)
    
- Name it as “SOC-Splunk”.

    ![ad_with_splunk_image](/blogs/ad_with_splunk/server_13.png)

## How to access servers

We will be connect to the servers and make configuration on it. There are 2 method to access the servers.

- Web Console
- RDP / SSH

### Web Console

Vultr provided a way for us to access the server through web browser. But this method is not so convenient, it take extra step for copy & paste feature.

We can launch the web console by clicking this button.

![ad_with_splunk_image](/blogs/ad_with_splunk/access_1.png)

It did provide some of the shortcut key for us to use. For example, we need to press **Clrt+Alt+Delete** in order to open the Windows lock screen. Vultr provided the shortcut key that will simulate pressing **Clrt+Alt+Delete**

![ad_with_splunk_image](/blogs/ad_with_splunk/access_2.png)

In order to paste the word you copy through web console, you need to open the “Clipboard” panel and paste your word inside and click “Paste” button.

![ad_with_splunk_image](/blogs/ad_with_splunk/access_3.png)

![ad_with_splunk_image](/blogs/ad_with_splunk/access_4.png)

### RDP / SSH

RDP and SSH is the most popular tools for remote connection. RDP is used on Windows machine and SSH is used on Linux machine. One of the differences between RDP and SSH is RDP will have GUI on connected remote server and SSH will be terminal base without GUI.

#### RDP

1. To connect server via RDP, search “RDP” on your local machine,
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/access_5.png)
    
2. Enter the public IP address of the remote server. Click “Show Options”.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/access_6.png)
    
3. Enter the Username.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/access_7.png)
    
4. It will popup a window ask you to input password. Copy the password from Vultr.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/access_8.png)
    
5. It will popup a window ask do you trust the remote server, click “Yes”.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/access_9.png)
    
6. You successfully connect to remote server via RDP.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/access_10.png)
    

#### SSH

1. To connect remote server via SSH, search “PowerShell” on your local machine.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/access_11.png)
    
2. Type `ssh <username>@<remote server public IP>` and press enter.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/access_12.png)
    
3. Since this is the first time we try to access this server, it will ask do we trust this server. Type “yes” and enter.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/access_13.png)
    
4. Copy the password and paste in by right click on your mouse.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/access_14.png)
    
5. You successfully login via SSH.

    ![ad_with_splunk_image](/blogs/ad_with_splunk/access_15.png)

## Firewall

We need to enable firewall network to safeguard the lab environment. 

1. Navigate to Network > Firewall > Add Firewall Groups.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/firewall_1.png)
    
2. Add a new Firewall Group name as “SOC-Network”.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/firewall_2.png)
    
3. We need to add 2 rules in the “Inbound IPv4 Rules”:
    - Port 22 for SSH connection and only allow My IP address.
    - Port 3389 for RDP connection and only allow My IP address.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/firewall_3.png)
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/firewall_4.png)
    

This is the inbound rule after you added.

![ad_with_splunk_image](/blogs/ad_with_splunk/firewall_5.png)

### Apply firewall rules to servers

Now we created the firewall group, but it won’t auto apply to all the server. We need to apply it manually.

1. Navigate to Compute > SOC-ADDC01
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/firewall_6.png)
    
2. Navigate to Settings > Firewall > Select the Firewall Group > Update the Firewall Group
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/firewall_7.png)
    
3. Repeat the same step for SOC-ADNormal and SOC-Splunk.

## Enable VPC Network

In order to let each server communicate with each other, we need to put them in same network. We can achieve it with VPC network, it will create a virtual network and connect all servers as it in same network.

1. Navigate to Settings > IPv4 > Enable VPC.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/vpc_1.png)
    
2. It will prompt to notify the server will be restart to enable it, click “Enable VPC”.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/vpc_2.png)
    
3. After few minute, the VPC will be enable. Repeat the same step for SOC-ADNormal and SOC-Splunk.

You will get something like below.

#### SOC-ADDC01

![ad_with_splunk_image](/blogs/ad_with_splunk/vpc_3.png)

#### SOC-ADNormal

![ad_with_splunk_image](/blogs/ad_with_splunk/vpc_4.png)

#### SOC-Splunk

![ad_with_splunk_image](/blogs/ad_with_splunk/vpc_5.png)

## Verify connection

After all the server restart, we can connect to our server and verify the connection.

First we connect to SOC-Splunk server and check the network interfaces by `ip a` command.

![ad_with_splunk_image](/blogs/ad_with_splunk/connect_1.png)

We can see that we have 3 network interfaces.

- First one is the loopback interface, we can ignore it.
- Second is the interface associate with public IP address.
- Third is the interface associate with private IP address, which is VPC network.

Let ping other server private IP address, it should be pingable since it in the same network.

![ad_with_splunk_image](/blogs/ad_with_splunk/connect_2.png)

From above image we know that SOC-ADDC01 is unreachable from SOC-Splunk.

Let connect to SOC-ADDC01 to check it. Check the network interface with `ipconfig` command.

![ad_with_splunk_image](/blogs/ad_with_splunk/connect_3.png)

We can see 2 network interfaces.

- First one is associate with public IP address.
- Second one is associate with private IP address but not the one show in VPC network.

We need to change the IP address to the one show on VPC network. Right click the Network icon from bottom right.

![ad_with_splunk_image](/blogs/ad_with_splunk/connect_4.png)

Open Network & Internet settings > Change adapter options.

![ad_with_splunk_image](/blogs/ad_with_splunk/connect_5.png)

The one we want to change is “Ethernet Instance 0 2”.

![ad_with_splunk_image](/blogs/ad_with_splunk/connect_6.png)

Double click it and navigate to Properties > IPv4 > Properties.

![ad_with_splunk_image](/blogs/ad_with_splunk/connect_7.png)

Enter the VPC IP address into the box as the picture showed below then click "Ok".

![ad_with_splunk_image](/blogs/ad_with_splunk/connect_8.png)

Verify the changes.

![ad_with_splunk_image](/blogs/ad_with_splunk/connect_9.png)

Now ping it again from SOC-Splunk to SOC-ADDC01.

![ad_with_splunk_image](/blogs/ad_with_splunk/connect_10.png)

Now SOC-Splunk able to communicate with SOC-ADDC01.

Do the same for SOC-ADNormal.

## Install AD service

AD service are not installed by default, we need to install it manually.

1. Navigate to Server Manager > Add roles and feature
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/ad_service_1.png)
    
2. Leave all as default, click Next until “Server Roles”. Check the “Active Directory Domain Services”.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/ad_service_2.png)
    
3. Click “Add Features”.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/ad_service_3.png)
    
4. Leave all as default, click Next until “Confirmation” and install it.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/ad_service_4.png)
    
5. Wait for the installation. Once it done, you can click “Close” to close the window.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/ad_service_5.png)
    
6. You can see a flag icon on top right of the navbar. Click the flag and click “Promote this server to a domain controller”.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/ad_service_6.png)
    
7. Add a domain name for your domain.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/ad_service_7.png)
    
8. Create a strong password for your domain. The password need to be:
    - More than 12 character.
    - At lease 1 uppercase.
    - At lease 1 lowercase.
    - At lease 1 special character (symbol).
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/ad_service_8.png)
    
9. Leave all as default, click Next until “Prerequisites Check”. Click “Install”.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/ad_service_9.png)
    
10. Wait for it install. Once it completed installation, you will be logout and RDP session will be end. Because the server need to restart for latest installed AD services.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/ad_service_10.png)
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/ad_service_11.png)
    
11. After few minutes, you will be able connect to remote server via RDP again. You can verify the AD services successfully installed by the new AD tools in **Tools** added by AD services**.**

    ![ad_with_splunk_image](/blogs/ad_with_splunk/ad_service_12.png)

## Add new domain user

Let create a domain user on the AD so that we can login into another AD machine with this user.

1. Navigate Tools > Active Directory Users and Computers.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/domain_user_1.png)
    
2. Navigate to SOC.local > Users (Right click) > New > User
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/domain_user_2.png)
    
3. Fill in the user information, I name it as Jerry.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/domain_user_3.png)
    
4. Create a password for this user. Since this user it just for practice purpose, we will disable the need of user change the password at next logon and enable password never expires.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/domain_user_4.png)
    
5. Review the user creation and click “Finish”.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/domain_user_5.png)
    
6. New user created.

    ![ad_with_splunk_image](/blogs/ad_with_splunk/domain_user_6.png)

## Join the domain

Windows server won’t join a AD automatically, we need to configure and authorize it with Administrator account.

1. RDP to SOC-ADNormal
2. Search for “This PC” and right click it to select “Properties”.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_1.png)
    
3. Select “Rename this PC (advanced)”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_2.png)
    
4. Click “Change”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_3.png)
    
5. Select the domain and enter domain name.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_4.png)
    
6. Take note here, we need to enter the Administrator credential on SOC-ADDC01 because it need the Administrator from Domain Controller to authorize it. It make sense, since an outsider trying to join your network, so it will need permission from you.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_5.png)
    
7. You might encounter this error, this is because the server unable to locate the Domain Controller.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_6.png)
    

#### Unable to locate Domain

1. To solve this problem, open the Network & Internet settings > Change adapter options.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_7.png)
    
2. Right click the “Ethernet Instance 0 2” and select Properties.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_8.png)
    
3. Navigate to IPv4 > Properties. Enter the private IP address of the SOC-ADDC01.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_9.png)
    
4. Now try again to join the domain.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_10.png)
    
5. It work and you need to restart the server to apply the change.

    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_11.png)

    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_12.png)

### Use Domain User credential

After restart the server, we can login as the domain user we created on SOC-ADDC01.

1. Enter the domain user credential.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_13.png)
    
2. Successfully login.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_14.png)
    

### Allow RDP login for Normal Machine

Now, let try login into Normal Machine as Domain User with RDP. 

1. In order to login as domain user via RDP, we need to specific the domain in front of username.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_15.png)
    
2. Click “yes”.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_16.png)
    
3. It prompt error state user is not authorized for RDP. 
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_17.png)
    
4. Let allow the user to RDP. RDP to SOC-ADNormal with Administrator credential and search for “remote” and select “Allow remote connections to this computer”.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_18.png)
    
5. Click “Show settings”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_19.png)
    
6. Login with Domain Controller Administrator credential.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_20.png)
    
7. Click “Select Users”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_21.png)
    
8. Click “Add”, enter the domain user name and click “Check Names” to verify is the user exist then click “OK”.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_22.png)
    
9. Try to RDP with domain user again.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_23.png)
    
10. Successfully RDP with domain user credential.

    ![ad_with_splunk_image](/blogs/ad_with_splunk/join_domain_24.png)

## Set up Splunk server

Splunk is one of the powerful SIEM tool that collecting and managing massive volumes of machine-generated data and analyze it.

We will open a port in Splunk to receive logs from other servers and create an alert to monitor the event to detect unauthorized successful logon.

1. Connect to Ubuntu server via SSH or web console.
2. Update the Ubuntu package manager with `apt-get update && apt-get upgrade -y`
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_server_1.png)
    
3. Navigate to [Splunk](https://www.splunk.com/) and register an account. (You can log in your account if you already have one.)
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_server_2.png)
    
4. Click “Trials & Downloads”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_server_3.png)
    
5. Find “Splunk Enterprise” and click “Get My Free Trial”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_server_4.png)
    
6. Switch to Linux tab and find .deb version and copy the wget link.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_server_5.png)
    
7. Paste the wget link into Ubuntu server. It will download Splunk deb package.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_server_6.png)
    
8. Install with`dpkg -i splunk-9.4.2-e9664af3d956-linux-amd64.deb`
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_server_7.png)
    
9. Start the splunk with `/opt/splunk/bin/splunk start` 
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_server_8.png)
    
10. Read through the term and condition and accept the license at the end.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_server_9.png)
    
11. Enter administrator username and a strong password.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_server_10.png)
    
12. Wait for the installation complete. Once completed, you will see message like below.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_server_11.png)
    

### Access Splunk web interface

Now we can try to access the Splunk from our browser on `https://<SOC-Splunk public IP>:8000`. But browser state it unreachable. This is because port 8000 block by the Firewall Group on Vultr and firewall rule on Ubuntu server.

![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_server_12.png)

#### Configure Firewall Group

1. Navigate to Network > Firewall > Edit Firewall
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_server_13.png)
    
2. Allow port 8000 to my IP address only
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_server_14.png)
    

This this the inbound rules after you make the change.

![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_server_15.png)

#### Configure Ubuntu server’s firewall

1. Type `ufw allow 8000` command into terminal and enter.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_server_16.png)
    

Now we will able to access Splunk from browser.

![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_server_17.png)

### Configure Splunk

Before we can actually use Splunk to analyze logs data, we need to make some of the configuration.

#### Time zone

Time zone will affect how we filter the data, we might miss out some of the data if the date out of range. We need configure the time zone into GMT to prevent this.

1. Navigate to Administrator > Preferences
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_1.png)
    
2. Set the time zone into GMT then click Apply
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_2.png)
    

#### Install windows add-on

Windows add-on is pre-configure filter that apply into Windows Event Log, it included predefined field from the Windows Event Log so that you don’t have to do it yourself.

1. Navigate to Apps > Find More Apps 
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_3.png)
    
2. Search for Windows > Install “Splunk Add-on for Microsoft Windows”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_4.png)
    
3. Login Splunk account and agree the term & condition and install.

    > Login with the Splunk account, not the one used to login this web interface.
    {: .prompt-tip }
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_5.png)
    
4. Once it complete the installation, you can close this window.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_6.png)
    

#### New index for our logs data

Splunk categorize data with index, we can create our own index for our servers’ logs. So that we can just filter this index to get all the data related to our servers.

1. Navigate to Settings > Indexes
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_7.png)
    
2. Click “New Index”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_8.png)
    
3. Give this index a name and create the index.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_9.png)
    

#### Open receiving port

We need open a port for other servers send log data to our Splunk server periodically.

1. Navigate to Settings > Forwarding and receiving
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_10.png)
    
2. Under Receive data section, click “Configure receiving”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_11.png)
    
3. Click “New Receiving Port”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_12.png)
    
4. Assign a port, by default is 9997 and save it.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_13.png)
    
5. Successfully enable receiving port
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_14.png)

## Install Splunk Forwarder

Splunk Forwarder is an agent installed on servers. This agent will aggregate server’s logs data and send it to Splunk via the Splunk’s receiving port.

1. Open browser, navigate to [Splunk Download page](https://www.splunk.com/en_us/download.html), find “Universal Forwarder” and click “Get My Free Download”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_forwarder_1.png)
    
2. Switch to Windows tab, select the **Windows Server 2022** version and click “Download Now”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_forwarder_2.png)
    
3. Agree the term & condition
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_forwarder_3.png)
    
4. Download the Splunk Forwarder.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_forwarder_4.png)
    
5. RDP Connect to SOC-ADNormal. (I log on as the domain user I created, you can log on as Administrator if you want)
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_forwarder_5.png)
    
6. Copy and paste the Splunk Forwarder msi file into SOC-ADNormal. (RDP alow direct copy & paste feature)
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_forwarder_6.png)
    
7. Wait for it finish.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_forwarder_7.png)
    
8. After finish copy, open the Splunk Forwarder. Agree the License Agreement, select on-premises and click “Next”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_forwarder_8.png)
    
9. Enter a username and click “Next”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_forwarder_9.png)
    
10. Click “Next” until “Receiving Indexer”. Enter the private IP address of SOC-Splunk and receiving port number.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_forwarder_10.png)
    
11. Click “Install”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_forwarder_11.png)
    
12. We need to login Administrator account, since this machine had join a Domain, we need to use the Administrator credential in Domain Controller. Unfortunately, this UAC popup is not allow we use copy and paste shortcut key, which mean we need to type in manually.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_forwarder_12.png)
    
13. Once it finish install, click on the “Finish”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/splunk_forwarder_13.png)

#### Splunk Forwarder configuration

We need to group our server’s log into the index we created - soc-ad.

1. Navigate to `C: > Program Files > SplunkUniversalForwarder`, it going to ask you enter the Administrator credential again. 
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_1.png)
    
2. Continue navigate to `etc > system > default`, and copy a file name `inputs.conf`
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_2.png)
    
3. Paste it to `etc > system > local`
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_3.png)
    
4. Right click the `inputs.conf` and click “Open with”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_4.png)
    
5. Open the file with Notepad. 
    
    > If you didn’t see Notepad, click “More apps” and select the Notepad to open it.
    {: .prompt-tip }
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_5.png)
    
6. Scroll until the end of the file and add in below config.
    
    ```
    [WinEventLog://Security]
    index = soc-ad
    disabled = false
    ```
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_6.png)
    
7. Save the file and close it.

#### Configure the Splunk Forwarder service

I want Splunk Forwarder run with Windows system account.

1. Search for “Services” and right click it and select “Run as Administrator”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_7.png)
    
2. Enter Administrator credential again.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_8.png)
    
3. Find the **SplunkForwarder**
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_9.png)
    
4. Right click it and select “Properties”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_10.png)
    
5. Navigate to “Log On” and select “Local System account”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_11.png)
    
6. Back to the Services window and restart the **SplunkForwarder**
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_12.png)
    
7. You might encounter this Warning, but no need to worry. Just close the windows by click “OK”.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_13.png)
    
8. And start the **SplunkForwarder** manually.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_14.png)
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_15.png)
    
9. Now do the same for SOC-ADDC01

#### Verify the Splunk Forwarder

1. Navigate to Splunk > Apps > Search & Reporting
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_16.png)
    
2. And query our log with `index="soc-ad"`
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_17.png)
    
3. But we get 0 event. This is because the Splunk server’s firewall haven’t allow the port to receive the log from other server.
4. Go back to Splunk server and allow traffic from port 9997
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_18.png)
    
5. Now we try again.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/config_splunk_forwarder_19.png)

## Configure Splunk Alert

In the nutshell, Splunk alert will trigger notification when it found an event that match the criterial you provided.

1. Before we create an alert, we need to let the alert what kind of event need to trigger the alert. Navigate to Splunk > Apps > Search & Reporting.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_1.png)
    
2. We need to form the query text for alert to query the event.
3. We only want to have an alert when a user successfully login, the Event ID 4624 indicate a successful login.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_2.png)
    
4. So we can add in `EventCode=4624` to filter out login event.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_3.png)
    
5. There are different logon type indicated how the user login. When user login through RDP, it associate with type 7 or 10.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_4.png)
    
6. So we also want to filter out the logon type.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_5.png)
    
7. Next, let said we only allow access to the server through office IP — my IP in this case. We need to filter out my IP for the alert. From the filter on left panel, we can found a filter call “Source_Network_Address”. This filter is indicate the source of the IP connected to those servers.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_6.png)
    
8. There is a “-” in the filter. Let take deeper analysis on that. Click the “-” to filter event that have `Source_Network_Address=”-”` 
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_7.png)
    
9. It show that this kind of event is from logon type = 7. We login directly from the server windows screen.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_8.png)
    
10. For this walkthrough, we going to filter out `Source_Network_Address=”-”` .
    
    > This action just for demonstration, you might want to keep this for you alert in Production environment. 
    {: .prompt-warning }
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_9.png)
    

11. Now we need to filter out those login not from our office IP (my IP). Let said my office IP was start from 115. So we need to filter out those IP not start from 115.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_10.png)
    
12. It normal if you see 0 event because we are connecting to server with our own IP so far.
13. In order to generate event from other IP can connect to a VPN on our own machine and RDP to the server or wait for other connect to your server. But before this, we need to disable a firewall rule since it only allow my IP for RDP. 
14. Navigate to Firewall configuration page and delete the RDP rule.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_11.png)
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_12.png)
    
15. Add in RDP rule that accept any IP.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_13.png)
    
16. Now anyone can connect to your server through RDP and successfully authorized if your user’s password is weak.
17. For this walkthrough, I had connected to a VPN and login to the server. Now Splunk should be have the unauthorize successful login event from IP other than 115.*.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_14.png)
    
18. To make it prettier, we only need to show those important information. For example like time, computer name, source IP address, and username.
    
    > If you not able to filter `user` , it might cause by you didn’t install the “Splunk Add-on for Microsoft Windows”.
    {: .prompt-warning }
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_15.png)
    
19. Now we had form the query we want to use in alert, we can save it as our alert.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_16.png)
    
20. Enter a meaningful name for the alert.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_17.png)
    
21. For this walkthrough, we want the alert run scanning every 1 min so that we won’t to have too long just to see the result.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_18.png)
    
22. And for trigger action, we want Splunk to notify us when the alert trigger. Select “Add to Triggered Alerts”.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_19.png)
    
23. Severity can leave as default and save the alert.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_20.png)
    
24. We will receive a warning but can ignore it. We can close the window.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_21.png)
    

### Verify the alert

1. Navigate to Activity > Triggered Alerts.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_22.png)
    
2. Wait about 1 minute for the alert to scan the aggregated log and we will have a alert in this page.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/alert_23.png)

## Clean up

To avoid unexpected charges, we need to destroy the server on Vultr.

1. Navigate to Compute. Click the three dot on the right and select “Server Destroy”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/clean_1.png)
    
2. Check the box and click “Destroy Server”
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/clean_2.png)
    
3. Wait Vultr destroy the server.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/clean_3.png)
    
4. It destroyed. Repeat the same for all other servers.
    
    ![ad_with_splunk_image](/blogs/ad_with_splunk/clean_4.png)

## Conclusion

We had successfully created 3 server on Vultr. 2 servers running on Windows Server 2022 and 1 server running on Ubuntu. We installed Active Directory in one of the Windows Server and make it as Domain Controller, have another Windows server join into the same domain. We also created a domain user that can log on into any of the server that join the domain. Moreover, we also installed a Splunk SIEM on Ubuntu server and keep collection logs from other 2 servers. We also created a Splunk alert to keep monitoring the servers’ logs data to detect any unauthorized successful log on event.

## References
- [Active Directory](https://en.wikipedia.org/wiki/Active_Directory)
- [SoC Analyst — Active Directory Home Lab](https://medium.com/@aniketkolte10/soc-analyst-active-directory-home-lab-37580b8c6ca3)
- [Cybersecurity Project: Active Directory 2.0](https://www.youtube.com/watch?v=dUdj-OmrFi0)
