---
title: Elastic EDR
description: >-
  Confi gured EDR in ELK and installed EDR agent into a Linux virtual machine aggregate logs and analyze the virtual machine to detect abnormal behaviour.
author: first_wan
categories: [Cybersecurity, Project]
tags: [SIEM, SOC, ELK]
pin: true
---

## What is EDR?
EDR stand for Endpoint detection & response. The endpoint are referring server, laptop, desktop, mobile device. EDR collect activity data happen within the endpoint, analyze the behavior of the endpoint. The key component of ERD included real-time monitoring, threat detection, incident response, and data collection and analysis.

Threat detection techniques used by EDR solutions include: 

- **Signature analysis:** Network traffic signatures are checked against a database of known [malware](https://www.spiceworks.com/security/endpoint-security/articles/what-is-malware-types-removal/) signatures to find a match.
- **Behavioral analysis:** The accepted behavior threshold of the endpoint is benchmarked to identify instances of unusual behavior, even if all traffic signatures are valid.
- **Sandbox analysis:** Potentially malicious files are placed in a safe environment called a sandbox and then executed to observe their behavior without risking any damage to the endpoint.
- **Whitelist/blacklist matching**: Endpoint activities are checked against a predetermined list of whitelisted and blacklisted IP addresses to allow or deny network traffic.

## Setup ERD on Elastic
### Create Elastic account
1. Create a free account on Elastic platform. → https://cloud.elastic.co/registration
2. Free account have short trial period for you to test out Elastic services. 
3. Create a deployment with any cloud provider you want.
   
   ![elastic_edr_image](/blogs/elastic_edr/elastic_account_1.png)
4. Choose a region that near you server or device location and size you need for your project. Then click on “Create Deployment”.
5. Wait for the configuration to complete.
6. Once deployment is ready, click “continue”

### Create VM
We need to have a host to represent as endpoint that EDR can gather logs. You can use any VM or device for this, as long as that device able to connect internet. I will use Kali Linux VM for this tutorial.

We can download a Kali Linux VM from [HERE](https://www.kali.org/get-kali/#kali-virtual-machines).

![elastic_edr_image](/blogs/elastic_edr/vm_1.png)

Download and install any VM according your preferred virtualization platform.

### Install Elastic agent on VM to collect logs
An agent is an application running in device’s background to collect and send data to centralized system like Elastic platform for analysis and monitoring.

1. Log in to Elastic platform, navigate to integration page.

   ![elastic_edr_image](/blogs/elastic_edr/elastic_agent_1.png)
2. Search for “Elastic Defend”.

   ![elastic_edr_image](/blogs/elastic_edr/elastic_agent_2.png)
3. Click “Add Elastic Defend”
   
   ![elastic_edr_image](/blogs/elastic_edr/elastic_agent_3.png)
4. Click “Install Elastic Agent”

   ![elastic_edr_image](/blogs/elastic_edr/elastic_agent_4.png)
5. Follow the instructions to install an agent on your device.

   ![elastic_edr_image](/blogs/elastic_edr/elastic_agent_5.png)
6. Paste the command script to your device that you wish to install the Elastic agent. The script will install the agent for you.

   ![elastic_edr_image](/blogs/elastic_edr/elastic_agent_6.png)
7. Once the agent finished install, you will see a message mentioning “Elastic Agent has been successfully installed”. It will start collecting the information in the device and forward it to Elastic platform for analysis. You can verify the generated log files on Linux with this file page: `/opt/Elastic/Endpoint/state/log`{: .filepath}, but this folder required root user privilege to access.
   
   You can also verify is the Elastic agent running in you device by this command:
   ```bash
   systemctl status elastic-agent.service
   ```
   ![elastic_edr_image](/blogs/elastic_edr/elastic_agent_7.png)

### Generate log stream
In order to verify the agent is running and sending the data to Elastic platform, we run a few command on this device. For example activate some service like ssh, running a nmap scan within that deivces.

1. Enable ssh in your attack machine

   ![elastic_edr_image](/blogs/elastic_edr/log_stream_1.png)
2. Run Nmap scan to the VM that installed Elastic agent

   ![elastic_edr_image](/blogs/elastic_edr/log_stream_2.png)

### Verify logs on Elastic
After a few minute, the agent should be log those action preformed on previous step and send to Elastic platform.

1. Navigate to Observability > Log.

   ![elastic_edr_image](/blogs/elastic_edr/verify_1.png)
2. Enter query into search bar. For example: to search all log related to nmap by `process.command_line.text :"nmap”` or `process.args: "nmap”`.
   
   You can get full detail by clicking the extend icon on the most left icon button under the action column.

   ![elastic_edr_image](/blogs/elastic_edr/verify_2.png)

## Extra: Create dashboard on Elastic
We can also create a visualization dashboard on Elastic to analyze the logs and identify patterns or anomalies in the data.

1. Navigate to Analytics > Dashboards.

   ![elastic_edr_image](/blogs/elastic_edr/dashboard_1.png)
2. Click “Create Dashboard”
   
   ![elastic_edr_image](/blogs/elastic_edr/dashboard_2.png)
3. Click “Create visualization”
   
   ![elastic_edr_image](/blogs/elastic_edr/dashboard_3.png)
4. Drag the “Record” from left panel to center.
   
   ![elastic_edr_image](/blogs/elastic_edr/dashboard_4.png)
5. Select “Area” on the visualization setting panel on right-hand side. Leave “Horizontal axis” and ”Vertical axis” as default.
   
   ![elastic_edr_image](/blogs/elastic_edr/dashboard_5.png)
6. Save the dashboard.

   ![elastic_edr_image](/blogs/elastic_edr/dashboard_6.png)

   ![elastic_edr_image](/blogs/elastic_edr/dashboard_7.png)

## Conclusion
We enabled EDR service on Elastic platform, and installed an agent on our device that keep collecting and sending the device processes data to Elastic platform for analysis and monitoring. We also created a dashboard to visualize our data on Elastic platform. 

## References
- [A Simple Elastic SIEM Lab](https://medium.com/@aali23/a-simple-elastic-siem-lab-6765159ee2b2)
- [Build a Powerful Home SIEM Lab Without Hassle! (Step by Step Guide)](https://www.youtube.com/watch?v=2XLzMb9oZBI)
- [strandjs/IntroLabs](https://github.com/strandjs/IntroLabs/blob/master/IntroClassFiles/Tools/IntroClass/md/elk_in_the_cloud.md)
- [How To Setup ELK](https://www.youtube.com/watch?v=wiQ8U5mFncw&pp=ygUWc2llbSBsYWIgd2l0aCBlbGFzdGljcw==)
- [Understanding the function of EDR, IDS, and IPS in Cyber Security](https://medium.com/@ademkucuk/understanding-the-function-of-edr-ids-and-ips-in-cyber-security-74ad35fb0775)
- [Linux IDS/EDR vs. CDR](https://sysdig.com/learn-cloud-native/linux-ids-edr-vs-cdr/)
- [How EDR Works](https://www.xcitium.com/how-edr-works/)
- [Protecting Your Castle: Understanding IDS, IPS, and EDR](https://www.linkedin.com/pulse/protecting-your-castle-understanding-ids-ips-edr-sohail-хакер--ifhtf/)
- [What Is EDR and Why Is It Important?](https://www.linkedin.com/pulse/what-edr-why-important-usman-shahzad/)
- [What Is Endpoint Detection and Response?](https://www.spiceworks.com/it-security/endpoint-security/articles/what-is-edr/)
