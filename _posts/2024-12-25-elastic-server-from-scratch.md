---
title: Setup Elastic server in Ubuntu from scratch
description: >-
  Setup a new Elastic server from download and install the machine's OS to install the Elastic from package. 
author: first_wan
categories: [Cybersecurity]
tags: [SIEM, SOC, ELK]
pin: true
---

## Introduction
The ELK stack is a collection of three open-source tools that work together to process and analyze data from multiple sources. This guide will walkthrough how to host a ELK on a Linux server.

## Procedures
### 1. Download Ubuntu server
Download latest version of Ubuntu server that used to host ELK services. By the time of writing, the latest version was Ubuntu 24.04.1.

[Download Here ->](https://ubuntu.com/download/server)

### 2. Install ubuntu server
After download the ISO file, install it into server. For this showcase, I will install it in VMware Workstation. 

Video Walkthrough: [Shah Rukh Install Ubuntu Server on a Local VMware Virtual Machine](https://www.youtube.com/watch?v=xdP2xJrxCTo)

- Select “New Virtual Machine”
  
  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_1.png)
- Use the ISO file downloaded on previous procedure
  
  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_2.png)
- Give a name to your virtual machine

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_3.png)
- Other option can use default options. Click “Next >” until last window, then click “Finish”.
- Start the VM, and select “Try or Install Ubuntu Server”

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_4.png)
- Select the language

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_5.png)
- Select “Update to the new installer”

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_6.png)
- Leave other as default

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_7.png)

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_8.png)

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_9.png)

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_10.png)

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_11.png)
- Deselect the LVM 

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_12.png)
- Review the storage configuration and click “Done”

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_13.png)
- Select “Continue”

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_14.png)
- Enter the server name and your username. For this lab setup, I will keep it simple to use elk as my username, server name, and password.

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_15.png)
- Skip for upgrade it to Ubuntu Pro

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_16.png)
- I want to enable remote connect into this server via SSH, so I select “Install OpenSSH server”.

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_17.png)
- Leave as default and click “Done”

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_18.png)
- Wait for the server install necessary packages and click “Reboot Now”

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_19.png)
- Press enter after you see this screen, then the server will start to reboot.

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_20.png)
- After that, the server is ready

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/install_ubuntu_21.png)
  
### 3. Configure Ubuntu server
After installed the Ubuntu server, we need to configure the time zone of the server. Otherwise, elastic might logged in different timezone. Because the timestamp logged into ELK will be follow the time set on the server. The time zone across all the server should be same.

- Find the name of your time zone
  ```bash
  timedatectl list-timezones | grep -i Singapore
  ```
  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/configure_ubuntu_1.png)
- Set the time zone
  ```bash
  sudo timedatectl set-timezone Asia/Singapore
  ```
  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/configure_ubuntu_2.png)
- Verify it
  
  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/configure_ubuntu_3.png)

### 4. Configure static IP address (Optional)
> This need to be configure before install ELK package if you are using DHCP for you server. During installation of ELK package, it will generate certificate with the server’s IP address. If you switch the IP address in the future, you will need to reinstall the ELK or change the ELK certificate.
{: .prompt-warning }

For this showcase, I don’t want DHCP assign IP address to this server, so I assign a static IP address to it.

- Disable cloud initiative network configuration
  - Create a file in /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
  - Copy the following into the file
    ```yaml
    network: {config: disabled}
    ```
- Disable network interface if it is up, verify with `ip a`
  
  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/configure_ip_1.png)
  ```bash
  sudo networkctl down ens33
  ```
  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/configure_ip_2.png)

- Modify `netplan`
  ```yaml
  network:
    version: 2
    ethernets:
        ens33:
            dhcp4: false
            addresses:
              - 172.16.50.2/24
            routes:
              - to: default
                via: 172.16.50.1
            nameservers:
              addresses:
                - 172.16.50.1

  ```
- Verify the latest netplan with `sudo netplan try`
- Enable network interface with `sudo networkctl up ens33`

### 5. Install Elasticsearch
In this step, we can finally install the Elasticsearch.

Reference link: [Elastic official guide](https://www.elastic.co/guide/en/elasticsearch/reference/8.16/deb.html#deb-repo)

- Import Elasticsearch GPG key
  ```bash
  wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
  ```
- Create Elasticsearch repo file
  ```bash
  sudo apt-get install apt-transport-https
  echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
  ```
- Install Elasticsearch from apt
  ```bash
  sudo apt-get update && sudo apt-get install elasticsearch
  ```
  > During installation of Elasticsearch, it will generate password for superuser. We need to record down the password for future use.
  {: .prompt-info }

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/elastic_1.png)
- Enable the Elasticsearch
  ```bash
  sudo systemctl daemon-reload && \
  sudo systemctl enable elasticsearch.service && \
  sudo systemctl start elasticsearch.service
  ```
- Verify the Elasticsearch with the superuser password generated.
  ```bash
  curl -X GET -k https://elastic:{superuser password}@localhost:9200
  ```

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/elastic_2.png)

### 6. Install Kibana
Kibana is a user interface for admin to interact with.

Reference link: [Elastic official guide](https://www.elastic.co/guide/en/kibana/8.15/deb.html#deb-repo)

- Install Kibana from apt
  ```bash
  sudo apt-get install kibana
  ```
- Create enrollment token to connect Elasticsearch and Kibana
  ```bash
  sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
  ```

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/kibana_1.png)
- Copy the token and run Kibana setup and paste the token into it
  ```bash
  sudo /usr/share/kibana/bin/kibana-setup
  ```
- Enable Kibana web interface by modify `server.host` in `/etc/kibana/kibana.yml`{: .filepath}
- Start Kibana
  ```bash
  sudo systemctl daemon-reload && \
  sudo systemctl enable kibana.service && \
  sudo systemctl start kibana.service
  ```

### 7. Create API key to enable Kibana alert feature
Kibana alert feature is not enable until we generate and install the API key.

- Generate Kibana encryption key
  ```bash
  sudo /usr/share/kibana/bin/kibana-encryption-keys generate
  ```
- It will generate 3 secret key, copy all of it into notepad
  
  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/kibana_alert_1.png)
- Add these key into Kibana
  ```bash
  sudo /usr/share/kibana/bin/kibana-keystore add {key name}
  ```
  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/kibana_alert_2.png)
- Restart Kibana service
  ```bash
  sudo systemctl restart kibana.service
  ```
- Navigate to Security > Alert to verify it. If no warning, then you’re good to go.

  ![elk_from_scratch_image](/blogs/elk_server_from_scratch/kibana_alert_3.png)

### 8. Install Logstash (Optional)
Logstash is a service that collect and aggregate data from multiple sources in real-time. Once the data is ingested, Logstash allows you to parse and transform it using a variety of filters. You can use these filters to clean, enrich, and modify the data before it is sent to the final destination.

- Install Logstash from apt
  ```bash
  sudo apt-get install kibana
  ```
- Enable Logstash
  ```bash
  sudo systemctl daemon-reload && \
  sudo systemctl enable logstash.service && \
  sudo systemctl start logstash.service
  ```

## Conclusion
We had installed a blank Ubuntu server and configure the timezone and IP of the server for Elastic use. Moreover, we download and install Elastic into the server and and enable Kibana to visualize Elastic log data in web. We also enable the alert feature on the Kibana and install Logstash to collect and aggregate log data from multiple sources.

## References
- [Download Elasticsearch](https://www.elastic.co/downloads/elasticsearch)
- [Setting Up the ELK Stack in 2023: Step-by-Step Tutorial](https://www.youtube.com/watch?v=wy24XVEYk_Y)
- [Setting Up Elastic 8 with Kibana, Fleet, Endpoint Security, and Windows Log Collection](https://www.youtube.com/watch?v=Ts-ofIVRMo4&t=1570s)
- [Installing Logstash](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html)
- ["Step-by-Step Guide: Installing Logstash for Beginners"](https://www.youtube.com/watch?v=PjFvTXxCGbE)
