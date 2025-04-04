<h1>Custom Multi-Region Web Application in Azure </h1>
This project focuses on deploying a highly available web application across two Azure regions using Azure Load Balancers & Traffic manager for cross-region failover. A virtual network peering was implemented for seamless connection between regions. In addition, to enhance security, NSGs were applied with security rules, DDoS protection, custom subnets for network segmentation & traffic management & a VPN Gateway was also configured to securely connect an on-premises resourse to Azure. This tutorial provides a guide on implementing a 
<br />

<h2>Technologies Used in This Project</h2>

**Azure Infrastructure**
- **Azure Virtual Machines** (*EastUS-VM-HA / WestUS-VM-HA*)
  - *Windows Server 2019*
  - *East US & West US regions*

**Networking**
- **Azure Virtual Networks** (*Prod-VNet-EastUS / Prod-VNet-WestUS*)
  - Custom Subnets: (*Web, App & DB*)
- Azure Virtual Network Peering
- Virtual Network Gateway (VPN)
- Azure Load Balancer
- Azure Traffic Manager

**Security & Access Control**
- Network Security Groups (NSGs)
- DDos Protection
- Remote Desktop

**Windows**
- PowerShell
- Windows Defender Firewall
- RDP
- Certificate Manager (certmgr)

<h2>Deployment and Configuration Steps</h2>
<h3 align="center">In the Azure (portal.azure.com)</h3>
<br />

In the **Basic Tab**, Create a VM (*EastUS-VM-HA*) In this case, the VM is created in the East US region & designed to enhance fault tolerance & high avaiability.
Create a Username & strong password. Select *"none"* for public inbound ports (To enhance security by limiting direct exposure to the internet)
![Screenshot 2025-03-29 220219](https://github.com/user-attachments/assets/e42616e7-85f3-4d0f-8816-0bff03a35e3e)
![Screenshot 2025-03-29 220305](https://github.com/user-attachments/assets/0b49f4f8-1f9c-4673-bfc8-f394526a72c3)
![Screenshot 2025-03-29 220322](https://github.com/user-attachments/assets/5712478c-34c5-4b7b-855e-0d0362ca390c)
<br></br>

In the **Disk Tab**, create a new disk & set the disk size to 64GB for extra storage (Premium SSD). I added a disk to store logs & application data separately but it's not needed
![Screenshot 2025-03-29 223926](https://github.com/user-attachments/assets/933c7690-66fb-4766-a2b7-bc12ca00dcce)
<br></br>

In the **Networking Tab**, create a VNet (*Prod-VNet-EastUS*) associated with your VM and its designated region. Within the VNet, setup a **Web-Subnet (10.30.1.0/24)** to isolate & manage web traffic. Do not assign a Public IP directly, the Load Balancer will handle external access.
![Screenshot 2025-03-29 230548](https://github.com/user-attachments/assets/7d760980-bc52-414e-b9bd-76f95445e9dc)
![Screenshot 2025-03-29 230548](https://github.com/user-attachments/assets/5da813a6-77cf-4216-bf5f-504b2eae7e25)
<br></br>

In the **Management Tab**, for patch orchestration options, select Azure-Orchestrated if you would like for it to be centralized and managed by Azure, ensuring consistency, compliance and high availability across all the VMs. Basically automatic patching and less manual efforts. 
![Screenshot 2025-03-29 233256](https://github.com/user-attachments/assets/81e4c7dd-aebe-4451-9dfb-13c136e70f5f)
<br></br>

In the **Monitoring Tab**, enable application health monitoring which will monitor the health of applications running inside the VM 
![Screenshot 2025-03-29 233603](https://github.com/user-attachments/assets/c5579ab1-a2a4-459f-8941-029fc9bdf544)
<br></br>

In the **Tags Tab**, create your own form of key-value pairs which helps organize, manage & track resources very effectively. Then **create** 
![Screenshot 2025-03-29 230548](https://github.com/user-attachments/assets/b231a9fa-ec33-4368-9850-d339f4c48723)
![Screenshot 2025-03-29 235054](https://github.com/user-attachments/assets/9318e008-d4bc-433b-9bbb-02c2d6022d79)
<br></br>

**Repeat these steps & create another VM & VNet in the WestUS region** </br>
(Deploy a secondary VNet in WestUS for failover)
  - Virtual Machine (WestUS-VM-HA)
  - VNet (Prod-VNet-WestUS)
      - Subnet- 10.40.1.0/24
![Screenshot 2025-03-30 013121](https://github.com/user-attachments/assets/ee312efd-2799-4e7b-bd34-91729455d436)

Create a Load Balancer (*EastUS-LB*) for the designated region & select 'Public', which allows incoming traffic from the internet to be distributed to the backend VM 
![Screenshot 2025-03-30 014300](https://github.com/user-attachments/assets/2bb6aaf5-b335-4466-a039-620de6cfe32f)

Add a frontend IP. Click '*Create new*' and name the IP address (*PublicWebAppIP-EastUS*)
![Screenshot 2025-03-30 100926](https://github.com/user-attachments/assets/47130496-6fa2-46a0-90cc-aba1c1c29f8f)

Add a backend pool and name it (*EastUS-BackendPool*). Select the designated VNet. Click 'add' under IP configuration and select the VM associated to the Load Balancer. When done click 'add' then save
![Screenshot 2025-03-30 102242](https://github.com/user-attachments/assets/a770d4f1-cc6f-4bfb-a23d-b191d6cb2659)

Create a name (*HTTP-LB-Rule*). Select the associated frontend IP address & backend pool. Since this rule is for HTTP traffic, use port 80 a
![Screenshot 2025-03-30 103128](https://github.com/user-attachments/assets/ca99da17-eaaf-48af-a277-f958f9e9fa17)



This VM will be configured with a Private IP address only. 

"When setting up a Virtual Machine in Azure, I need to ensure high availability, scalability, and fault tolerance. Instead of using the default VNet, I would design a custom networking architecture that supports a globally distributed application."

Step-by-Step Guide: Configuring High Availability, Load Balancing, and Security in Azure


When enabling "none" for Public Inbound ports, all traffic from the internet will be blocked by default. You can change the inbound port rule 
	****(Public IP: Choose None (since Load Balancer will manage traffic)***


Choosing the Frontend IP for Your Load Balancer in Azure
When creating a Load Balancer for your High-Availability Web Application, the Frontend IP Configuration tab requires you to specify an IP address that clients will use to access your application.

Each load-balancing rule (one for HTTP, one for HTTPS) needs an associated health probe to determine whether a backend VM is responsive for that specific protocol.


Health Probe checks if your backend VMs are responding correctly. When path is set to "/", the probe will send HTTP request to / (the root of your web server) verify it is running.

Routing Preference:
Determines how your traffic routes between Azure and the Internet
Microsoft Network - Selecting Microsoft global network delivers traffic via Microsoft global network closest to user. 
Internet - Uses transit ISP network. 
Routing preference option of a Public IP can’t be changed once created
nd select Microsoft network as the routing preference (Delivers traffic via Microsofts global network to users) 








 




