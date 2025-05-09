---
title: Azure SIEM
description: >-
  Set up an Azure SIEM that aggregates logs and detects intruders from which country based on the IP addresses.
author: first_wan
categories: [Cybersecurity]
tags: [SIEM, SOC]
pin: false
---

This showcase is for my lab practice. I setup a Azure SIEM to gather the event data that try to break into my VM machine, and show a map to see where the attack come from.

## Overview
![overview](/blogs/azure_siem/siem_overview.png)

## Walkthrough

### Create log analytics workspace
First, we need to create a Log Analytics Workspace that will be used connect to Azure Sentinel.

![overview](/blogs/azure_siem/workspace_1.png)

Click “Create” to create new Log Analytics workspaces

![overview](/blogs/azure_siem/workspace_2.png)

Create anew resource group if you don’t have one, and give a name to the Log Analytics workspace

![overview](/blogs/azure_siem/workspace_3.png)

### Create VM
Create a Window VM that will be access by attacker. 

![overview](/blogs/azure_siem/windows_vm_1.png)

This VM just for my lab practice, so no redundancy are required, and set the security to standard as this is a honeypot for any attacker to access.

![overview](/blogs/azure_siem/windows_vm_2.png)

Size will be at least 2 vcpu and 8gb memory for my preference, we need to setup something in the VM later. You need to remember the username and password.

![overview](/blogs/azure_siem/windows_vm_3.png)

Be sure to agree the license before we go to Networking configuration.

![overview](/blogs/azure_siem/windows_vm_4.png)

Select Advanced network security group for Microsoft to create a network security group resource. We will use this to configure the network firewall with this resource later.

### Setup custom log extract on VM
1. Log in to the VM
2. Open Windows PowerShell ISE and create a new shell

   ![overview](/blogs/azure_siem/log_extract_1.png)
3. You can download or copy the custom log exporter from —> [Custom_Security_Log_Exporter.ps1](https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1)
4. his script are written by Josh Madakor, the script just loop all the failed login from Windows Event and retrieve the geolocation by the IP address using IP Geolocation API —> [IP Geolocation API](https://ipgeolocation.io/)
5. You will need to sign up an account in IP Geolocation API to get a API key, the account is free but have 1000 daily limit query.
6. Replace the API key and run the script.
   
   ![overview](/blogs/azure_siem/log_extract_2.png)
7. Verify the script is running by login with wrong username.
   
   ![overview](/blogs/azure_siem/log_extract_3.png)

   ![overview](/blogs/azure_siem/log_extract_4.png)
8. The script will generate a list of sample log once run, these sample log will be used to configure on data extract to Azure table later.
   
   ![overview](/blogs/azure_siem/log_extract_5.png)

### Connect Log Analytics Workspace to Microsoft Sentinel
Navigate to Content hub

![overview](/blogs/azure_siem/connect_1.png)

Install 2 solution on Content hub
- Windows Security Event
- Custom logs AMA
  
![overview](/blogs/azure_siem/connect_2.png)

#### Connect Windows VM Event log to Sentinel via Windows Security Events
1. Select Windows Security Events, then select “Manage” on the right-hand panel
   
   ![overview](/blogs/azure_siem/connect_3.png)
2. Find “Windows Security Events via AMA” and click “Open connector page”
   
   ![overview](/blogs/azure_siem/connect_4.png)
3. Create a new Data Collection Rule, this rule are used ingest log data from VM. 
   
   ![overview](/blogs/azure_siem/connect_5.png)
4. In the Resources tab, select the VM you created just now.
   
   ![overview](/blogs/azure_siem/connect_6.png)
5. The default configuration will collect all security events. After this, you can review and create the rule.
   
   ![overview](/blogs/azure_siem/connect_7.png)

#### Connect custom log generate by Windows VM to Microsoft Sentinel via Custom logs AMA
1. Select Custom logs AMA, then select “Manage” on the right-hand panel
   
   ![overview](/blogs/azure_siem/connect_8.png)
2. Select “Custom logs via AMA” and click “Open connection page”
   
   ![overview](/blogs/azure_siem/connect_9.png)
3. Create new Data Collection Rule
   
   ![overview](/blogs/azure_siem/connect_10.png)
4. Select which VM to apply the rule.
   
   ![overview](/blogs/azure_siem/connect_11.png)
5. Select “Custom new table” and give the table a name, specific the file path of the custom log.
   
   ![overview](/blogs/azure_siem/connect_12.png)

   ![overview](/blogs/azure_siem/connect_13.png)

### Verify the data connector
Before we continue, we need to make sure all the log will be send to Microsoft Sentinel by verify all the data connector is connected. In our case, we need the “Windows Security Events via AMA” & “Security Events via Legacy Agent” to be connected. 

“Windows Security Events via AMA” is for the default Windows Event log.

“Security Events via Legacy Agent” is for the custom log our script or application generated.

Go to Microsoft Sentinel > Configuration > Data connectors.

![overview](/blogs/azure_siem/verify_1.png)

As picture showed above, the “Security Events via Legacy Agent” is not connected. We need to fix the connection. On top of Azure portal, search “Data collection rules”.

![overview](/blogs/azure_siem/verify_2.png)

![overview](/blogs/azure_siem/verify_3.png)

In this page, you can see the rules you had created previously. Now, we want to focus on the rule that associate with my custom log, which is “FAILED_RDP_CUSTOM_LOG”.

![overview](/blogs/azure_siem/verify_4.png)

We go to Configuration > Resources, set the “Data Connection endpoint” that associate with Microsoft Sentinel. After that, we verify the Data connectors on Microsoft Sentinel.

![overview](/blogs/azure_siem/verify_5.png)

### Customize custom log table
By default, the custom log that gather from VM will ingest as long string data. In order to make the data useful and easy to read, we need to let Microsoft Sentinel know how to transform the data.

1. Go to Microsoft Sentinel > General > Logs. Select the table that created during "[Connect custom log generate by Windows VM to Microsoft Sentinel via Custom logs AMA](#connect-custom-log-generate-by-windows-vm-to-microsoft-sentinel-via-custom-logs-ama)"
   
   ![overview](/blogs/azure_siem/customize_1.png)
   
   ![overview](/blogs/azure_siem/customize_2.png)
2. Create a query that will output the data as you expected. Here will provide the query I used for this log.
   ```KQL
    FAILED_RDP_CL
    | parse RawData with * "latitude:" Latitude_CF ",longitude:" Longitude_CF ",destinationhost:" Destinationhost_CF ",username:" Username_CF ",sourcehost:" Sourcehost_CF ",state:" State_CF ", country:" Country_CF ",label:" Label_CF ",timestamp:" Timestamp_CF:datetime
    | project Latitude_CF, Longitude_CF, Destinationhost_CF, Username_CF, Sourcehost_CF, State_CF, Country_CF, Label_CF, Timestamp_CF
    | limit 10
   ```
   ![overview](/blogs/azure_siem/customize_3.png)
3. Go to Log Analytics workspaces, and select your workspace. Then select Setting > Table.
   
   ![overview](/blogs/azure_siem/customize_4.png)
4. Select “Edit scheme”
   
   ![overview](/blogs/azure_siem/customize_5.png)
5. Add custom field in “Custom Column”
   - All new column as type String, except Timestamp_CF as datetime
  
   ![overview](/blogs/azure_siem/customize_6.png)
6. Copy the parse query that you just created
   ```KQL
   | parse RawData with * "latitude:" Latitude_CF ",longitude:" Longitude_CF ",destinationhost:" Destinationhost_CF ",username:" Username_CF ",sourcehost:" Sourcehost_CF ",state:" State_CF ", country:" Country_CF ",label:" Label_CF ",timestamp:" Timestamp_CF:datetime
   ```
7. Go to “Data collection rules”, select the custom log rule. Go to Configuration > Data sources
   
   ![overview](/blogs/azure_siem/customize_7.png)
8. Paste the query into “Transform” field and save it.
   
   ![overview](/blogs/azure_siem/customize_8.png)
9.  The new transformed data only apply to new added data. Wait for few minute for new data to pass in or you can create new data by RDP to the VM with wrong username.
    
    ![overview](/blogs/azure_siem/customize_9.png)

### Create new workbook
Last but not least, we will create a map for us to visualize where the attack come from.

1. Go to Microsoft Sentinel, select the Log analytics workspace. Select Threat management > Workbooks.
   
   ![overview](/blogs/azure_siem/workbook_1.png)
2. Create a new Workbook. Remove all the default metric.
   
   ![overview](/blogs/azure_siem/workbook_2.png)
3. Add new query.
   
   ![overview](/blogs/azure_siem/workbook_3.png)
4. Copy the query below to create a summarize the event count of the geolocation data gather by custom log.
   ```KQL
   FAILED_RDP_CL
   | summarize event_count=count() by Sourcehost_CF, Latitude_CF, Longitude_CF, Country_CF, Label_CF, Destinationhost_CF
   | where Destinationhost_CF != "samplehost"
   | where Sourcehost_CF != ""
   ```
   ![overview](/blogs/azure_siem/workbook_4.png)
5. Select Visualization > Map
   
   ![overview](/blogs/azure_siem/workbook_5.png)
6. Configure the map as following
   
   ![overview](/blogs/azure_siem/workbook_6.png)

   ![overview](/blogs/azure_siem/workbook_7.png)
7. Click Done Editing and save the workbook. Now you have a map that show where the attack came from.
   
   ![overview](/blogs/azure_siem/workbook_8.png)

   ![overview](/blogs/azure_siem/workbook_9.png)

## Extra: Expose more port for attacker
Currently, only port 3389 is open for attacker to login. We can enable more port for attacker access that machine.

1. Go to Vitual Machine, and select the VM you wish to expose more port. Select Networking > Network settings.
   
   ![overview](/blogs/azure_siem/extra_1.png)
2. Add new inbound port rule.

   ![overview](/blogs/azure_siem/extra_2.png)
3. Allow all destination port, and make this rule as top priority by set it to 100.
   
   ![overview](/blogs/azure_siem/extra_3.png)
   ![overview](/blogs/azure_siem/extra_4.png)
4. Delete the default rule created by Network security group.

   ![overview](/blogs/azure_siem/extra_5.png)

## Conclusion
We have create a new virtual machine that open to internet. Anyone can access or break into that VM. We also configure data collection rules to collect the data from that VM. Moreover, we enable the Azure SIEM (Microsoft Sentinel) to aggregate all the log data from that machine and visualize those data as a map to know where the attack come from.

## Clean up
After finish the lab practice, we need to clean up the machine and service create on this lab practice to avoid these service used up all the credit we have.

1. Search “Resource groups” on top of Azure portal
   
   ![overview](/blogs/azure_siem/clean_up_1.png)
2. Delete the resource group created on this lab practice. For my case is “Honeypot-RG” and “NetworkWatcherRG”.
   > Note: Network Watcher created when you created a VM and no Network Watcher in that region
   {: .prompt-info }
   
   ![overview](/blogs/azure_siem/clean_up_2.png)

## References
- [SC-200T00A-Microsoft-Security-Operations-Analyst](https://microsoftlearning.github.io/SC-200T00A-Microsoft-Security-Operations-Analyst/)
- [SIEM Tutorial for Beginners](https://www.youtube.com/watch?v=RoZeVbbZ0o0&t=466s)
- [Collect logs from text files with the Azure Monitor Agent and ingest to Microsoft Sentinel - AMA](https://learn.microsoft.com/en-us/azure/sentinel/connect-custom-logs-ama?tabs=portal)
- [Data collection endpoints in Azure Monitor - Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/data-collection-endpoint-overview?tabs=portal)
- [Migration of custom fields to KQL-based transformations in Azure Monitor - Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/custom-fields-migrate)
- [What is Network Watcher and How Did It Get In My Subscription!](https://www.youtube.com/watch?v=aiQX-sOadvQ)
