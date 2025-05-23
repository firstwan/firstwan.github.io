---
title: Homelab honeypot
description: >-
  This project will setup a lab environment that have a honeypot server with a ELK server keep monitoring it and trigger an alert once unauthorized access to the honeypot server.
author: first_wan
categories: [Cybersecurity, Project]
tags: [SIEM, SOC, ELK]
pin: true
---


## Introduction 

This project will guide you to build an ELK stack and a honeypot server. An ELK stack is a log management platform that uses three open-source projects to collect, process, and analyze data from multiple sources. ELK provides a centralized solution for managing large volumes of data, offering real-time insights and analytics. A honeypot server is a cybersecurity tool that lures and analyzes cyberattacks to help organizations improve their security. Honeypots are designed to look like legitimate targets but isolated from critical systems to prevent attackers from causing real damage.

This project will set up a honeypot server in our lab, and forward the generated logs to the ELK stack through an elastic agent. Furthermore, it also included instructions on how to set up an alert that detects brute-force authentication attempts. Set up a dashboard to overview those users and IP addresses that tried to log in to the system but failed. 

Last but not least, this project prepared a bash script to simulate brute force and other attacks from the attacker's perspective. Use the bash script to test the ELK alert and view the log from the ELK stack. 

## Methodology 

### Network setup

We will use pfSense firewall router to do network segmentation and enhance network security by limiting the connection between networks.   

![homelab_honeypot_image](/blogs/homelab_honeypot/network_1.png)

#### Install pfSense 

1. File \> New Virtual Machine

   ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_1.png)

2. Select Custom  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_2.png)

3. Stick with defaults for this  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_3.png)

4. Select the right ISO  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_4.png)

5. Name the new machine and decide where it “lives”  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_5.png)

6. Set the specifications accordingly  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_6.png)  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_7.png)
   ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_8.png)
   ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_9.png)
   ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_10.png)
   ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_11.png)
   ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_12.png)

7. Uncheck “Power on this virtual machine after creation”, and select “Customize Hardware”  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_13.png)

8. Add new Network Adapters  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_14.png)  
9. Create new network segments as “CFC Lab”, and assign it to the Network Adapters added previously.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_15.png)

10. Close the setting window and click “Finish”  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_16.png)

11. Power on the machine

12. Go ahead with the default, until you reach the ZFS configuration screen. Press ***spacebar*** to check the box, then press enter key.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_17.png)

13. Use arrow key to select “Yes”, then press enter key to confirm.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_18.png)

14. Select “Reboot”, and wait for it.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_19.png)  
15. Done  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/pfsense_20.png)

#### Configure pfSense 

After we install pfSense, we can manually assign the IP address to each interface to have more control over the network.

1. Select Option 1 to `Assign interfaces` to decide which network adapter will be used for which network  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_1.png)

2. Enter “n” to select no vlan, “em0” as WAN interface, “em1” as LAN interface  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_2.png)  
3. Review the config and enter “y” to accept  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_3.png)

4. Now you configured the network for different interfaces, configure the IP network in the next step.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_4.png)

5. Select option 2 in the main menu to `Set Interface IP Address`  
   
   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_5.png)
6. Configure as follows.  
   1. Disable DHCP assign IPv4 for this interface.  
   2. Enter the IPv4 address for this interface.  
   3. Enter the network subnet for this interface.  
   4. Because we setting up LAN, can leave it empty.

   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_6.png)

7. We are not going to use IPv6, can enter “n” for no and leave empty for options.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_7.png)

8. No need to setup DHCP server for this network, we going to configure IP address manually to have more control over the network.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_8.png)

9.  Enter “n” for HTTP option  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_9.png)

10. Done setup interface.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_10.png)

11. We need to allow access to the web UI, go into `Shell` and use command `pfctl -d` to disable packet filtering.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_11.png)

12. Access pfSense web UI via my host OS (which is in VMware NAT as 192.168.236.132 in my case) through the WAN IP of pfSense.  
    1. user: admin  
    2. pw: pfsense

    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_12.png)

13. Click next until you get to this page  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_13.png)

14. Input the address for Google’s DNS servers, then Next  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_14.png)  
15. Select the time zone in your area  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_15.png)

16. Uncheck these options  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_16.png)

17. Click next for next steps  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_17.png)

18. Reset admin password  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_18.png)

19. Click “Reload”  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_19.png)

20. When you reload the machine, you need to go back to your shell and enter `pfctl -d` again  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_20.png)

    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_21.png)  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_22.png)

21. We need to allow WAN network able to access pfSense to do configuration, so that we don’t have disable pfSense package filter again in the future.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_23.png)  
22. Select WAN network and add new rule  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_24.png)

23. Configure it to allow traffic access to pfSense at port 443 (HTTPS).  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_25.png)

24. Save and then apply changes  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_26.png)

25. Last but not least, enable DNS Resolver to forward DNS query to 8.8.8.8 or 8.8.4.4 as configured in the firewall.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_27.png)  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_pfsense_28.png)

### Set up Honeypot server

We will set up a Windows Server 2016 as our honeypot server.

#### Install Windows Server 2016

1. Open browser and navigate to [https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016), and select “Download ISO”.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_1.png)

2. Register an account  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_2.png)

3. Download Windows Server 2016 ISO

   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_3.png)

4. Install the ISO  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_4.png)  
5. Enter the username for your user, no need to key in the product key  
   
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_5.png)  
    
6. Click “Yes”  
   
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_6.png)

7. Give the new VM a name and click “Next”  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_7.png)  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_8.png)  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_9.png)
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_10.png)
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_11.png)
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_12.png)
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_13.png)
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_14.png)

8. Click “Finish” to install it,  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_15.png)

9. Open the setting, remove the “Floppy” disk  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_16.png)  
    

10. Launch the VM, follow the instruction  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_17.png)

11. Select the time zone of your area.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_18.png)

12. Click “Install Now” and wait it install  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_19.png)

13. Select Standard Evaluation (Desktop Experience)  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_20.png)

14. Accept the license  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_21.png)

15. Custom install  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_22.png)

16. Select default and wait for the installation to be done  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_23.png)

17. Setup Administrator account.  
    * Username: Administrator  
    * Password: Passw0rd\!
  
   > This password just for demonstrate on this walkthrough, you can use any password you like.
   {: .prompt-tip }

    ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_24.png)

18. Installed Windows Server 2016, login into VM, with VMware you can press `ctrl + alt + insert` to unlock instead.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_25.png)

19. Install VMware Tools  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_26.png)

20. Click the notification  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_27.png)

21. Go with the default options and click Finish  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_28.png)

22. Select no, we will restart by ourself.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_29.png)

23. Go to start menu, select restart \> Other (Planned)  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_30.png)  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_31.png)

#### Configure honeypot

1. Close the VM if it is running, and assign `CFC Lab` into network adaptor.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_honeypot_1.png)

2. Launch the VM.

3. Click “Local Server”, select the network interface which is **“Ethernet0”** in this case.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_honeypot_2.png)

4. Select the IPv4 to configure it.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_honeypot_3.png)

5. Configure a static IP for it.  
   * **IP address:** The static IP address for the machine.  
   * **Subnet mask:** Subnet mask of this network.  
   * **Default gateway:** The IP address of the pfSense router.  
   * **Preferred DNS server:** The IP address of the pfSense router, pfSense router will help to resolve the DNS query.

   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_honeypot_4.png)

6. Verify the configuration by ping the pfSense router.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_honeypot_5.png)  
7. We need to enable RDP. From Server Manager, open Local Server \> Remote Desktop  
   * RDP allow administrator configure the system remotely. RDP is disabled by default, we will enable it for this lab.

   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_honeypot_6.png)

8. Select allow remote connection, the Administrator is allowed to use RDP by default. If you have other users who need to access via RDP, you can grant RDP access to your user by adding the user to it.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_honeypot_7.png)  
9. Click “OK” to close it.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_honeypot_8.png)  
10.  Press F5 on Server Manager, then it will show “Enabled” on remote desktop.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/config_honeypot_9.png)

#### Install sysmon

1. Navigate to [Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) and download Sysmon  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/sysmon_1.png)  
2. Extract the file  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/sysmon_2.png)

3. Download Sysmon config file from [sysmonconfig-export.xml](https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/refs/heads/master/sysmonconfig-export.xml)  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/sysmon_3.png)

4. Copy the file to Sysmon folder  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/sysmon_4.png)

5. Open Windows PowerShell as administrator.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/sysmon_5.png)

6. Install Sysmon with the config file by following command:
   
   ```powershell
   .Sysmon.exe \-n \-accepteula \-i .sysmonconfig-export.txt
   ```
   
   ![homelab_honeypot_image](/blogs/homelab_honeypot/sysmon_6.png)

#### Install tools to configure ELK VM 

There are a lot of commands and config to be set in order to configure ELK VM, but the Ubuntu server does not support the copy & paste feature if you directly interact with the server. To make our life easier, we can install tools like MobaXterm to establish SSH connection and configure it.

I will install MobaXterm on the honeypot VM because it is on same network, so that I don’t have to configure a firewall rule to forward my traffic to the ELK VM.

1. On honeypot VM, open the browser and search for **“MobaXterm”**.

   * If you encounter this popup window, just click “Add”.

   ![homelab_honeypot_image](/blogs/homelab_honeypot/mobaxterm_1.png)

2. Download the free version.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/mobaxterm_2.png)
   ![homelab_honeypot_image](/blogs/homelab_honeypot/mobaxterm_3.png)

3. Extract the file and install it.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/mobaxterm_4.png)

4. Stick to default for all options.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/mobaxterm_5.png)

### Honeypot hardening

We don’t want attackers to be aware that they are attacking a Honeypot server, to lure them in, we need to configure the server as a normal server.

#### Close unnecessary port 

By default, Windows server closed all ports except port 5985\. Port 5985 is used for the "Windows Remote Management" (WinRM) service, for admin to connect it and manage the server.

![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_hardening_1.png)

Let assume we have opened some unnecessary ports, we can disable them through the Firewall setting.

1. Go to Tools \> Windows Firewall with Advanced Security  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_hardening_2.png)  
2. Find the firewall rules press right click and select “Disable Rule”  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_hardening_3.png)

#### Setup strong password policy 

1. Press Windows key \+ R to open the Run program, and type “gpedit.msc”.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_hardening_4.png)

2. Navigate to Computer Configuration\> Windows Settings \> Security Settings \> Account Policies \> Password Policy  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_hardening_5.png)

3. Configure as follow  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_hardening_6.png)

4. Change your Administrator’s password if password length less than 12 character.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_hardening_7.png)

#### Configure to auto update 

1. Press Windows key \+ R to open Run program, and type “gpedit.msc”.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_hardening_8.png)

2. Navigate to Computer Configuration \> Administrative Templates \> Windows Components \> Windows Update  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_hardening_9.png)  
3. Double click “Configure Automatic Updates”  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_hardening_10.png)

4. Select “Enable” to enable to auto update, since we want this machine is honeypot and we won’t often access to this machine, we want it update automatically, so we select “4 \- Auto download and schedule the install” and make it run on every Sunday.

   * **3 \- Auto download and notify for install:** This option will download latest update and let admin choose when to install  
   * **4 \- Auto download and schedule the install:** This option will force the machine to install the update once it downloaded.

   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_hardening_11.png)

5. Click “Ok” to finish the setting.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/honeypot_hardening_12.png)

### Setup ELK

The ELK stack is a collection of three open-source tools that work together to process and analyze data from multiple sources. This walkthrough will guild you to setup a Ubuntu server and install ELK stack on that server.

#### Install Ubuntu server
First we need to download the Ubuntu ISO from the Internet and configure it.

* Reference video: [https://www.youtube.com/watch?v=xdP2xJrxCTo](https://www.youtube.com/watch?v=xdP2xJrxCTo)  

1. Download latest version of Ubuntu server. By the time of writing, the latest version was Ubuntu 24.04.1.  
   * Download URL: [https://ubuntu.com/download/server](https://ubuntu.com/download/server)

   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_1.png)

2. Select “New Virtual Machine”  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_2.png)  
3. Use the ISO file downloaded on previous procedure  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_3.png)

4. Give a name to your virtual machine  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_4.png)

5. Other option can use default options. Click “Next \>” until last window, then click “Finish”.

6. Start the VM, and select “Try or Install Ubuntu Server”  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_5.png)

7. Select the language  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_6.png)

8. Select “Update to the new installer”  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_7.png)

9.  Leave other as default  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_8.png)
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_9.png)
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_10.png)
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_11.png)
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_12.png)

10. Uncheck the LVM group  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_13.png)

11. Review the storage configuration and click “Done”  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_14.png)

12. Select “Continue”  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_15.png)

13. Enter the server name and your username. For this lab setup, I will keep it simple to use **`elk`** as my username, server name, and password.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_16.png)  
     

15. Skip for upgrade it to Ubuntu Pro  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_17.png)

16. I want to enable remote connect into this server via SSH, so I select “Install OpenSSH server”.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_18.png)  
17. Leave as default and click “Done”  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_19.png)  
18. Wait for the server install necessary packages and click “Reboot Now”  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_20.png)

19. Press enter after you see this screen, then the server will start to reboot.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_21.png)

20. After that, the server is ready  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_22.png)  
21. Close the VM if it is running, and assign `CFC Lab` into network adaptor.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_server_23.png)

22. Launch the VM.

#### Configure Ubuntu server

After installed the Ubuntu server, we need to configure the time zone of the server. The timestamp logged into ELK will follow the time set on the server. The time zone across all the servers should be the same.

There are a lot of commands and config to be set in order to configure ELK VM, but the Ubuntu server does not support the copy & paste feature if you directly interact with the server. We will connect via MobaXterm for better configuration.

1. Find the name of your time zone  
   ```bash
   timedatectl list-timezones | grep -i Singapore
   ```

   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_server_1.png)

1. Set the time zone  
   ```bash
   sudo timedatectl set-timezone Asia/Singapore
   ```

   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_server_2.png)

1. Verify it  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_server_3.png)

#### Configure static IP address

Beware before installing the ELK package, during installation of the ELK package, it will generate a certificate with the server’s IP address. If you switch the IP address in the future, you will need to reinstall the ELK or change the ELK certificate.

For this setup, I don’t want DHCP to assign the IP address to this server, so I assign a static IP address to it.

1. Create a file in `/etc/cloud/cloud.cfg.d/99-disable-network-config.cfg`{: .filepath}.
   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_server_4.png)

2. Copy the following into the file
   ```yaml
   network: {config: disabled}
   ```

3. Disable network interface if it is up, verify with `ip a`  
   * **ip a:** Command that output all the network interfaces’ information.

   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_server_5.png)

   ```bash
   sudo networkctl down ens33 
   ```

   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_server_6.png)

4. Modify netplan
   ```yaml
   network:
    version: 2
    ethernets:
        ens33:
            dhcp4: false
            addresses:
              - 10.10.10.2/24
            routes:
              - to: default
                via: 10.10.10.1
            nameservers:
              addresses:
                - 10.10.10.1
   ```

5. Verify the latest netplan with `sudo netplan try`

6. Enable network interface
   ```bash
   sudo networkctl up ens33
   ```

7. Verify it.

   * **ip route:** This command will output assigned IPv4 address and the default gateway of the network.

   * **ping \-c 1 8.8.8.8:** Ping command to check we have network connection.

   ![homelab_honeypot_image](/blogs/homelab_honeypot/config_server_7.png)

#### Install Elasticsearch

In this step, we can finally install the Elasticsearch. We can follow the instructions from Elastic official website to install it into our Ubuntu server.

Elastic official link: [https://www.elastic.co](https://www.elastic.co/guide/en/elasticsearch/reference/8.16/deb.html#deb-repo)

There are a lot of commands and config to be set in order to configure ELK VM, but the Ubuntu server does not support the copy & paste feature if you directly interact with the server. We will connect via MobaXterm for better configuration.

1. Make sure the SSH service is started on the ELK server.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_1.png)  
2. Open MobaXterm, and start a SSH session to ELK VM  
   * Username: elk  
   * Password: elk

   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_2.png)

3. Connected to ELK VM, now we can copy and paste the command with *mouse right click*.

4. Import Elasticsearch GPG key
   ```bash
   wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
   ```

5. Create Elasticsearch repo file
   ```bash
   sudo apt-get install apt-transport-https
   echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list 
   ```

1. Install Elasticsearch from apt  
   * During the installation of Elasticsearch, it will generate the password for the superuser. We need to record down the password for future use.
   ```bash
   sudo apt-get update && sudo apt-get install elasticsearch 
   ```

   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_3.png)

2. Enable the Elasticsearch
   ```bash
   sudo systemctl daemon-reload && /
   sudo systemctl enable elasticsearch.service sudo systemctl start elasticsearch.service 
   ```

3. Verify the Elasticsearch with the superuser password generated.

   ```bash
   curl -X GET -k https://elastic:{superuser password}@localhost:9200 
   ```

   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_4.png)

#### Install Kibana

Kibana is a user interface for the admin to interact with.

Reference link: [https://www.elastic.co](https://www.elastic.co/guide/en/kibana/8.15/deb.html#deb-repo)

1. Install Kibana from apt
   ```bash
   sudo apt-get install kibana
   ```

2. Create enrollment token to connect Elasticsearch and Kibana
   ```bash
   sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
   ```

   ![homelab_honeypot_image](/blogs/homelab_honeypot/kibana_1.png)

3. Copy the token to Kibana setup

   ```bash
   sudo /usr/share/kibana/bin/kibana-setup 
   ```

4. Enable Kibana web interface  
   * Modify `/etc/kibana/kibana.yml`  

   ![homelab_honeypot_image](/blogs/homelab_honeypot/kibana_2.png)` 
5. Enable Kibana

   ```bash
   sudo systemctl daemon-reload 
   sudo systemctl enable kibana.service
   sudo systemctl start kibana.service 
   ```

6. You can access Kibana now at http://10.10.10.2:5601.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/kibana_3.png)  
7. Login as elastic, password of the superuser.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/kibana_4.png)

#### Create API key to enable Kibana alert feature 

Kibana alert feature is not enable by default, we need generate and install the API key in order to enable Kibana alert.

1. Generate Kibana encryption key
   ```bash
   sudo /usr/share/kibana/bin/kibana-encryption-keys generate
   ```

2. It will generate 3 secret key, copy all of it  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/kibana_5.png)

3. Add these keys into Kibana
   ```bash
   sudo /usr/share/kibana/bin/kibana-keystore add {key name}
   ```

   ![homelab_honeypot_image](/blogs/homelab_honeypot/kibana_6.png)

4. Restart Kibana service
   ```bash
   sudo systemctl restart kibana.servicez
   ```

5. Navigate to Security \> Alert to verify it. If no warning, then you’re good to go.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/kibana_7.png)

### Install Elastic Agent in Honeypot server

In order to analyze the log or set up an intrusion alert in ELK, we need to forward the log from the Honeypot server to the ELK stack. It can be achieved by installing Elastic Agent in the Honeypot server.

1. From the Honeypot server, navigate to http://10.10.10.2:5601/ and log in to ELK with the superuser password  
   * Username: elastic

   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_1.png)

2. Navigate to `Management > Integrations`, search for `Windows`  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_2.png)

3. Click Add Windows.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_3.png)

4. Leave all options as default  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_4.png)

5. Add elastic agent to host  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_5.png)

6. Click “Add agent”  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_6.png)

7. Select “Run standalone” and download the policy  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_7.png)

8. Create API key  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_8.png)

9. Open the elastic-agent.yml with notebook, search for “api-key” and paste the generated API key  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_9.png)

10. Scroll down to the bottom, since our honeypot server is Windows, we will follow Windows instruction  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_10.png)

11. Open PowerShell with Administrator from “Download” folder for easier management.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_11.png)

12. Copy the command to PowerShell one by one until the command before installation  
    > If the link return **404** error, try to check the URL. The URL should look like *https://artifacts.elastic.co/**downloads**/beats/elastic-agent/elastic-agent-xxx.zip*
    {: .prompt-warning }

    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_12.png)

13. Copy the previously downloaded yml file into Elastic agent folder, replace the original file  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_13.png)
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_14.png)  
14. Go back to PowerShell and install Elastic agent  
    * Select “Y” to run it as a service  
    * Select “N” to enroll it into fleet

    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_15.png)

15. Let's verify the agent is sending data to ELK, open the event viewer and clear security log  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_16.png)  
     

16. Navigate to ELK Discover  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_17.png)

17. Query “event.code: 1102”, if it shows the result, it means the Elastic Agent is working  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/elk_agent_18.png)

### Set up Elastic Alert

Elastic Alert will notify the admin if the log matches the pattern configured at Elastic Alert so that the organization can take action as soon as possible before further damage is done.

The following setup will demonstrate brute force attack, bear in mind that this is a simple configuration for brute force attack.

#### Brute force attempts alert

1. Login with the wrong password to generate an event log for failed authentication  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/alert_1.png)

2. Form the query to detect the brute force attack. After research on the Internet, we know that Windows categorized unauthorized access with event id 4625\.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/alert_2.png)

3. Write a KQL query out of that log, we also specify the agent name that is used for the Honeypot server.  
   * **event.code: 4625**: Filter out log with 4625 Event ID  
   * **agent.name: “WIN-M8ON1R5EUG6”**: The agent name that is installed in the Honeypot server. The agent name will be assigned during elastic agent installation, by default is the machine name.  Your agent name should be different from this.

   ![homelab_honeypot_image](/blogs/homelab_honeypot/alert_3.png)

4. Copy the KQL query statement and navigate to Security \> Alerts  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/alert_4.png)  
5. Click “Manage rules”  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/alert_5.png)  
6. Click “Create new rule”  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/alert_6.png)  
7. Select “Threshold” and scroll down.  
   * Threshold: This rule will only be triggered if same pattern had been detected number of time on short period. 

   ![homelab_honeypot_image](/blogs/homelab_honeypot/alert_7.png)

8. Paste in the KQL query, fill in the details and click “Continue”
   ![homelab_honeypot_image](/blogs/homelab_honeypot/alert_8.png)  
9.  Fill in the name of the alert and description, you can also set the severity level based on the alert.  
   ![homelab_honeypot_image](/blogs/homelab_honeypot/alert_9.png)

10. In the advanced setting, you can fill in additional information such as MITRE ID to support this alert investigation.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/alert_10.png)  
11. We can also include the investigation guide on what to do after the alert is triggered.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/alert_11.png)  
12. The rest can be kept as default and click “Continue”.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/alert_12.png)

13. Set to 5 minutes for both the rule’s time frame and look-back time.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/alert_13.png)

14. Leave empty for action, because we didn’t connected to any messaging service.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/alert_14.png)

15. Click “Create & enable rule”  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/alert_15.png)

16. Verify the rule by brute force on the Honeypot server. We can use Msfconsole in Kali with ***auxiliary/scanner/winrm/winrm\_login*** to brute force the credential, or Hydra to brute force RDP service.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/alert_16.png)

17. Check ELK, it will take around 5 to 10 min for the alert to trigger.  
    ![homelab_honeypot_image](/blogs/homelab_honeypot/alert_17.png)

### Kibana Dashboard

We can create a Kibana dashboard to get an overview of recent events before getting into details. Let's take brute force attack for example.

* Navigate to Analytics \> Dashboards  
  ![homelab_honeypot_image](/blogs/homelab_honeypot/dashboard_1.png)

* Click “Create dashboard”  
  ![homelab_honeypot_image](/blogs/homelab_honeypot/dashboard_2.png)

* Click “Create visualization”  
  ![homelab_honeypot_image](/blogs/homelab_honeypot/dashboard_3.png)

* On the right panel, select “Table”  
  ![homelab_honeypot_image](/blogs/homelab_honeypot/dashboard_4.png)

* We can see 3 sections here.  
  ![homelab_honeypot_image](/blogs/homelab_honeypot/dashboard_5.png)  
  * 1: This section is the available data that can be dragged and dropped into the table. From the view of the table.  
  * 2: You can write a query on this section to further filter the data that will show on the table.  
  * 3: This is the configuration of this visualization. You can rearrange the column position, change to pie chart, etc.  
* Drag the *timestamp*, *source.ip*, and *user.name* into middle  
  ![homelab_honeypot_image](/blogs/homelab_honeypot/dashboard_6.png)  
* We can customize each column. Let's take timestamp for example, select timestamp from the right panel.  
  ![homelab_honeypot_image](/blogs/homelab_honeypot/dashboard_7.png)  
* Change this to “Hour”  
  ![homelab_honeypot_image](/blogs/homelab_honeypot/dashboard_8.png)

* Now, the data will show with per hour interval  
  ![homelab_honeypot_image](/blogs/homelab_honeypot/dashboard_9.png)

* Next, select the source.ip and change the number of value to 99\. We want to see as many IP addresses as possible trying to brute force us instead of the top 3\.  
  ![homelab_honeypot_image](/blogs/homelab_honeypot/dashboard_10.png)

* Click “Save and return” to save the created dashboard  
  ![homelab_honeypot_image](/blogs/homelab_honeypot/dashboard_11.png)

* Name the dashboard  
  ![homelab_honeypot_image](/blogs/homelab_honeypot/dashboard_12.png)  
* Done  
  ![homelab_honeypot_image](/blogs/homelab_honeypot/dashboard_13.png)

### Simulate the attack

We will test the ELK alert by simulating an attack. I had already prepared an attack script written in bash that can run on a Linux machine, you can download the script from my [github repo](https://github.com/firstwan/vulner). 

Github Url: [https://github.com/firstwan/vulner](https://github.com/firstwan/vulner)

* Add execute permission to the bash script.  
  ![homelab_honeypot_image](/blogs/homelab_honeypot/attack_1.png)  
* Run the script with the command

| ./vulner.sh \-f \--skip-udp 10.10.10.3 |
| :------------------------------------- |

* It will start the network scan and try to brute force credential  
  ![homelab_honeypot_image](/blogs/homelab_honeypot/attack_2.png)

* If the machine has other services, it will output a list of available scanning techniques.  
  ![homelab_honeypot_image](/blogs/homelab_honeypot/attack_3.png)  
* After running the script, we can see it generatesan few ELK alert  
  ![homelab_honeypot_image](/blogs/homelab_honeypot/attack_4.png)
  ![homelab_honeypot_image](/blogs/homelab_honeypot/attack_5.png)

## Conclusion

We set up a new Ubuntu server and installed the ELK stack into it. Allow us to ingest log data from different sources. Furthermore, we set up a new Windows Server 2016 as our Honeypot server, acting as a legitimate target for attackers to access it by hardening the server like closing unnecessary ports, scheduling auto-updates, configuring a strong password policy, etc.

Furthermore, we set up an Elastic Alert to notify the admin once brute force is detected, and a dashboard to overview the failed login logs.

Moreover, we use a bash script to attack the Honeypot server to verify the Elastic Alert is working properly and view the generated logs.

## Reference 

* [*Download Elasticsearch*](https://www.elastic.co/downloads/elasticsearch)
* [*Installing Logstash*](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html)  
* [IppSec. (2022, October 10). *Setting Up Elastic 8 with Kibana, Fleet, Endpoint Security, and Windows Log Collection*](https://www.youtube.com/watch?v=Ts-ofIVRMo4)
* [Raj. (2024, July 16). *WinRM Penetration Testing \- Hacking Articles*. Hacking Articles.](https://www.hackingarticles.in/winrm-penetration-testing/)
* [*Security Log in Event Viewer does not store IPs*.](https://serverfault.com/questions/399878/security-log-in-event-viewer-does-not-store-ips)  
* [*Startup or run key Registry modification*](https://www.elastic.co/guide/en/security/current/startup-or-run-key-registry-modification.html)
* [The Devops Diary. (2023a, September 26). *“Step-by-Step Guide: Installing Logstash for Beginners”*](https://www.youtube.com/watch?v=PjFvTXxCGbE)  
* [The Devops Diary. (2023b, September 28). *Setting Up the ELK Stack in 2024: Step-by-Step Tutorial*](https://www.youtube.com/watch?v=wy24XVEYk\_Y)

