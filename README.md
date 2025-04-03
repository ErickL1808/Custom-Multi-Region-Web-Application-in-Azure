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
<p>
  Create a VM (EastUS-VM-HA) In this case, the VM is created in the East US region & designed for High Avaiability.
</p>

![Screenshot 2025-03-29 220219](https://github.com/user-attachments/assets/e42616e7-85f3-4d0f-8816-0bff03a35e3e)
![Screenshot 2025-03-29 220305](https://github.com/user-attachments/assets/0b49f4f8-1f9c-4673-bfc8-f394526a72c3)
![Screenshot 2025-03-29 220322](https://github.com/user-attachments/assets/5712478c-34c5-4b7b-855e-0d0362ca390c)
<br></br>

<p>
An additional disk is not needed but if you will like to store logs & application data separately, you can create a New Disk & set the disk size to 64GB for extra storage
</p>

![Screenshot 2025-03-29 223926](https://github.com/user-attachments/assets/933c7690-66fb-4766-a2b7-bc12ca00dcce)
<br></br>

<p>
In the Networking Tab, you can create your VNet associated with your VM and its region. Do not assign a Public IP directly as the Load Balancer will handle external access. Allow RDP or SSH for inbound ports 
</p>

![Screenshot 2025-03-29 230532](https://github.com/user-attachments/assets/a8b0acd4-b7ad-48b6-8835-bf04803035c1)


![Screenshot 2025-03-29 230548](https://github.com/user-attachments/assets/d343d4c3-52e8-464c-aa13-37f27e4f9ffc)

![Screenshot 2025-03-29 233256](https://github.com/user-attachments/assets/81e4c7dd-aebe-4451-9dfb-13c136e70f5f)

![Screenshot 2025-03-29 233603](https://github.com/user-attachments/assets/c5579ab1-a2a4-459f-8941-029fc9bdf544)

![Screenshot 2025-03-29 234743](https://github.com/user-attachments/assets/43ebcf11-a1a5-4307-816e-9e80c8c95459)

![Screenshot 2025-03-29 234845](https://github.com/user-attachments/assets/f2bdb786-e1b9-454e-a9db-59fcac2e168e)

![Screenshot 2025-03-29 235054](https://github.com/user-attachments/assets/9318e008-d4bc-433b-9bbb-02c2d6022d79)

















