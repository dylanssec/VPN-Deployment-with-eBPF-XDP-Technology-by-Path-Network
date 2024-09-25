# VPN-Deployment-with-eBPF-XDP-Technology-by-Path-Network

This project demonstrates how to deploy a DDoS-protected VPN server utilizing eBPF/XDP Technology by Path Network on a Linux Ubuntu 20.04 server. The setup includes installing the Pritunl panel, configuring robust DDoS protection using FranTech Stallion, and establishing essential firewall rules to secure network communications.

## Table of Contents
- [Introduction to eBPF/XDP Technology](#introduction-to-ebpfxdp-technology)
- [Prerequisites](#prerequisites)
- [Firewall Configuration](#Firewall-Configuration)
- [Pritunl Installation Steps](#Pritunl-Installation-Steps)
- [Pritunl Database Setup](#Pritunl-Database-Setup)
- [Create VPN Server on Pritunl Panel](#Create-VPN-Server-On-Pritunl-Panel)
- [Create Organization Pritunl Panel](#Create-Organization-Pritunl-Panel)
- [Connect the Organization to a VPN Server](Connect-the-Organization-to-a-VPN-Server)
- [Start VPN Server](#Start-VPN-Server)
- [Connect Using OpenVPN Client](#Connect-Using-OpenVPN-Client)
- [Conclusion](#conclusion)
- [Additional Documentation](#additional-documentation)

## Introduction to eBPF/XDP Technology

eBPF/XDP (Extended Berkeley Packet Filter/eXpress Data Path are advanced networking technologies that allow for high-performance packet processing at the kernel level. XDP, in particular, enables the server to handle and filter network packets before they reach the traditional networking stack, providing an efficient mechanism to mitigate DDoS attacks and reduce server load.

Path Network Integration
[Path Network's](https://path.net) eBPF/XDP technology integrates directly into the Linux kernel, enhancing the server's ability to process and filter incoming network traffic with minimal latency. Path Network’s technology is deployed in this project to provide robust DDoS protection for the VPN server, ensuring that malicious traffic is effectively filtered out.

## Prerequisites

- Access to FranTech Stallion: You'll need this to manage firewall settings and implement DDoS protection.
- A VPN Server: This will be used to whitelist protocols such as SSH, HTTP, HTTPS, and Pritunl, ensuring only authorized traffic is allowed.
- SSH Access to the Server: Secure remote access is necessary to configure and manage the server.

*Hardware Requirements*
- **Linux Server:** A Linux server (Ubuntu preferred) with:
  - **CPU:** 1 GHz or higher
  - **RAM:** 2 GB or more
  - **Disk Space:** 10 GB free space
- **Network Configuration:** Static IP address for consistent network communication.

*Software Requirements*
- **Operating System:** Ubuntu 20.04 LTS or later.

## Firewall Configuration

*Whitelist the following ports utilizing the FranTech Stallion Panel.*
*SSH (Port 22): Used for secure remote access to the server.*

![SSH (Port 22): Used for secure remote access to the server.](https://imgur.com/Nt6kwoF.png)

*HTTP (Port 80): Used for serving web content.*

![HTTP (Port 80): Used for serving web content.](https://imgur.com/WLGvg65.png)

*HTTPS (Port 443): Used for secure web traffic.*

![HTTPS (Port 443): Used for secure web traffic.](https://imgur.com/hUfN4Zl.png)

*OpenVPN UDP (Port 22000): Used for OpenVPN UDP traffic.*

![OpenVPN UDP (Port 22000): Used for OpenVPN UDP traffic.](https://imgur.com/hxda5Bw.png)

*Pritunl Web Panel (Port 31450): This port is used to access the Pritunl web panel after changing its default port. By allowing traffic on TCP port 31450, you ensure that the web interface for managing your VPN server remains accessible while using a non-default, custom port for added security.*

![Pritunl Web Panel (Port 31450): This port is used to access the Pritunl web panel after changing its default port. By allowing traffic on TCP port 31450, you ensure that the web interface for managing your VPN server remains accessible while using a non-default, custom port for added security.](https://imgur.com/dBdfBH3.png)

*Deny All Traffic: Configure the firewall to block all traffic by default, allowing only traffic on the whitelisted ports and from the specified IP addresses.*

![Deny All Traffic: Configure the firewall to block all traffic by default, allowing only traffic on the whitelisted ports and from the specified IP addresses.](https://imgur.com/deNxktg.png)

*Application Filter: Apply an OpenVPN UDP filter to Port 22000 to ensure only legitimate VPN traffic is allowed.*

![Application Filter: Apply an OpenVPN UDP filter to Port 22000 to ensure only legitimate VPN traffic is allowed.](https://imgur.com/njovjwT.png)

*Firewall Rules.*

![Firewall Rules](https://imgur.com/ErXd4k1.png)

*Application Filter.*

![Application Filter](https://imgur.com/qcJ2M1i.png)

## Pritunl Installation Steps

## 1. Update and Upgrade the System
*This step is crucial to mitigate security vulnerabilities and to ensure compatibility with the latest software packages. The -y flag automatically answers "yes" to prompts, streamlining the upgrade process.*
```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt update && sudo apt -y full-upgrade
````
![Update and Upgrade the System](https://imgur.com/wcrTh6o.png)

## 2. Install Required Dependencies

*Next, install necessary utilities such as GPG and curl, which are required for adding and managing software repositories.*

````bash
sudo apt install gpg curl -y
````
![Install Required Dependencies](https://imgur.com/IL8C1AC.png)

## 3: System Reboot (if required)

*Some upgrades may necessitate a system reboot. Check if a reboot is required and perform it if necessary.*

````
[ -f /var/run/reboot-required ] && sudo reboot -f
````
![System Reboot](https://imgur.com/YCe9hKk.png)

## 4. Add Pritunl and MongoDB Repositories

*Pritunl requires MongoDB as a database backend. You'll need to add both the Pritunl and MongoDB repositories to your system.*

````
echo "deb http://repo.pritunl.com/stable/apt $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/pritunl.list
````
````
sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list << EOF
````
````
deb https://repo.mongodb.org/apt/ubuntu $(lsb_release -cs)/mongodb-org/6.0 multiverse
````
````
EOF
````
![Add Pritunl and MongoDB Repositories](https://imgur.com/CeIgcxs.png)

*These commands add Pritunl and MongoDB repositories to your APT sources list, allowing you to install the latest versions of Pritunl and MongoDB directly from their respective sources. $(lsb_release -cs) dynamically inserts your Ubuntu version code name, ensuring compatibility.*

## 5. Import GPG Keys for Package Authentication

*To ensure the integrity and authenticity of the packages you're about to install, import the necessary GPG keys for both Pritunl and MongoDB.*

````
curl -fsSL https://www.mongodb.org/static/pgp/server-6.0.asc | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/mongodb-6.gpg
````
````
curl -fsSL https://raw.githubusercontent.com/pritunl/pgp/master/pritunl_repo_pub.asc | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/pritunl.gpg
````
![Import GPG Keys for Package Authentication](https://imgur.com/cOQRMqk.png)

*GPG keys are used to verify that the packages are signed by a trusted source and have not been tampered with. The --dearmor option converts the key to a binary format that APT can use.*

## 6. Install Pritunl and MongoDB

*With the repositories added and keys imported, you can now proceed to install Pritunl and MongoDB.*

````
sudo apt update
````
![Install Pritunl and MongoDB](https://imgur.com/9wiNZfP.png)
````
sudo apt --assume-yes install pritunl mongodb-org
````
![Install Pritunl and MongoDB](https://imgur.com/1r3NXoV.png)

*The apt update command refreshes the package lists to include the newly added repositories. The --assume-yes option automatically confirms the installation, ensuring a smooth process.*

## 7. Start and Enable Pritunl and MongoDB Services

*After installation, start the Pritunl and MongoDB services and enable them to start automatically on boot.*

````
sudo systemctl start pritunl mongod
````
````
sudo systemctl enable pritunl mongod
````
![Start and Enable Pritunl and MongoDB Services](https://imgur.com/OI6WjNt.png)

## Pritunl Database Setup

## 1. Access Pritunl Web Panel

*Open your web browser and navigate to the following URL, replacing 198.251.83.155 with your server's IP address:*
```
https://198.251.83.155/
```
![Access Pritunl Web Panel](https://imgur.com/20e8rwN.png)

## 2. Generate the Pritunl Setup Key
*Run the following command on your server*
````
sudo pritunl setup-key
````
![Generate the Pritunl Setup Key](https://imgur.com/OeuoexN.png)

## 3. Enter the Setup Key in the Web Panel

*Go back to the Pritunl web panel in your browser. When prompted, paste the setup key you copied from the server.*

![Enter the Setup Key in the Web Panel](https://imgur.com/2mEn723.png)

## 4. Generate Default Password
````
sudo pritunl default-password
````
![Generate Default Password](https://imgur.com/RBXZJZM.png)

## 5. Log into the Pritunl Web Panel

*Go to the Pritunl web panel in your browser at https://your-server-ip/.*

*Use the default credentials provided:*

*Username: pritunl*

*Password: defaultpassword (or whatever password was provided).*

![Log into the Pritunl Web Panel](https://imgur.com/urukpJB.png)

## 6. Change Default Username and Password

*Open your web browser and navigate to the Pritunl web panel at https://<your-server-ip>*
*Log in using the default credentials provided by the sudo pritunl default-password command.*

**Change the Default Password & Username**

*After logging in, navigate to the Settings section by clicking on your username or the settings icon.*

*Enter a new username.*

*Enter a new, secure password. Ensure it follows NIST guidelines.*

*Length: At least 12 characters.*

*Complexity: Include a mix of upper and lower case letters, numbers, and special characters.*

*Avoid: Common words or easily guessable patterns.*

*Example of a strong password: G!9aC7%kB*3vQ^Rz*

**Change the Default Web Console Port to 31450**

*Look for the Web Console settings or a similar option related to server configuration.*

**Update the Port Setting**

*Find the field labeled Web Console Port.*
*Change the port number from the default (443) to 31450.*

![Change the Default Password & Username](https://imgur.com/IaubtgD.png)

## Create VPN Server On Pritunl Panel

*On the Pritunl dashboard, click the "Add Server" button.*

## 1. Set Server Name
*In the server configuration window, set a name for your VPN server (e.g., "MyVPNServer").*

## 2. Protocol and Port 
*Under "Protocol", select "UDP". Set the "Port" to 22000.*

## 3. DNS Configuration
*Set the "DNS Server" to 1.1.1.1 (Cloudflare DNS).*

## 4. Encryption Cipher
*Under "Encryption", select AES-256-CBC (AES-256 bit encryption).*

## 5. Allow Multiple Devices
*Check the box for "Allow Multiple Devices" to enable multiple devices to connect using the same VPN profile.*

## 6. Block Outside DNS
*Check the box for "Block Outside DNS" to ensure that DNS requests are only routed through the VPN.*

![Create VPN Server on Pritunl Panel](https://imgur.com/MgPh4AZ.png)

## Create Organization Pritunl Panel

## 1. Go to Users Tab
*On the Pritunl dashboard, click on the "Users" tab in the left-hand menu.*

## 2. Add Organization
*In the "Users" section, click the "Add Organization" button.*

## 3. Name the Organization:

*In the dialog box that appears, enter a name for your new organization (e.g., "MyOrganization").*
*Click "Add" to create the organization.*

![Create Organization Pritunl Panel](https://imgur.com/YcA3WId.png)

## Connect the Organization to a VPN Server

## 1. Go to Servers Tab
*Navigate to the "Servers" tab in the Pritunl panel.*

## 2. Link Organization to Server
*Find the VPN server you want to link to the organization and click on it.*

*In the server settings, find the "Attach Organization" section.*

*Select the organization you created and click "Attach".*

*Now, the organization is set up and linked to the VPN server. Users within the organization can connect to the VPN server using their respective profiles.*

![Connect the Organization to a VPN Server](https://imgur.com/LmbJI0Y.png)

## Start VPN Server

# 1. Go to the Servers Tab
*On the Pritunl dashboard, click on the "Servers" tab in the left-hand menu.*

# 2. Locate Your VPN Server
*Find the VPN server that you want to start from the list of servers.*

# 3. Start the Server

*Click the "Start" button next to the server.*
*Once clicked, the server will start, and its status will change to "Online."*

![Start VPN Server](https://imgur.com/jgTmTTd.png)

## Connect Using OpenVPN Client

# 1. Download the VPN Configuration File

*Retrieve the .ovpn configuration file from the Pritunl web panel.*

![Retrieve the .ovpn](https://imgur.com/SjXHds2.png)

# 2. Download and Install OpenVPN Connect V3
*Visit the [OpenVPN](https://openvpn.net/client/client-connect-vpn-for-windows/) official website to download the latest version of OpenVPN Connect V3.*
*Install the application on your Windows machine by following the on-screen instructions.*

# 3. Import the Configuration File

*In the OpenVPN Connect interface, click on the Import Profile option.
Choose the File tab and then click on Browse.
Navigate to the location where you saved the .ovpn file and select it.
Click Open to import the file.*

![Import the Configuration File](https://imgur.com/5zORUDG.png)

# 4. Connect to the VPN

*Once imported, you will see the profile listed in OpenVPN Connect.
Click on Connect to establish a VPN connection using the imported profile.*

![Connnect to the VPN](https://imgur.com/dcvWnIg.png)

# 5. Verify VPN Connection
Once connected to the VPN, go to [checkhost](https://check-host.net/) to confirm that your IP address matches the one assigned by the VPN.
Ping Cloudflare DNS: Open Command Prompt on your Windows machine and run the following command:
```ping 1.1.1.1```

![Verify VPN Connection](https://imgur.com/YDQwuNf.png)
![Verify VPN Connection](https://imgur.com/VZJSP7e.png)

## Conclusion

*This project provides a comprehensive guide to deploying a DDoS-protected VPN server using eBPF/XDP technology by Path Network on a Linux Ubuntu 20.04 server. It details the process of integrating advanced networking technologies like eBPF and XDP to fortify the server against DDoS attacks, ensuring secure and reliable VPN access. The step-by-step instructions cover essential tasks, including the installation and configuration of the Pritunl VPN panel, the implementation of robust firewall rules using FranTech Stallion, and the management of network traffic with enhanced security measures. This setup demonstrates best practices for VPN deployment in high-security environments, offering strong protection and reliability.* 

