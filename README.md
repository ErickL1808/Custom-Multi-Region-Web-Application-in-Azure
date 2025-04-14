<h1>Custom Multi-Region Web Application in Azure </h1>
This project focuses on deploying a highly available web application across two Azure regions using Load Balancers & Traffic manager for cross-region failover. It guides you through deploying VMs with private IP addresses and configuring load balancers to efficiently manage all incoming traffic. Load balancing was configured for both HTTP and HTTPS protocols, each paired with a health probe to monitor backend VM responsiveness and ensure service availability. The infrastructure is segmented into custom subnets which improves manageability and security. The Web-Subnet hosts the frontend VM and handles incoming user traffic, the App-Subnet manages the application server and is accessible only internally and the DB-Subnet secures database access using private endpoints. To enforce least privilege and eliminate internet exposure, Network Security Groups are applied to each subnet to control inbound/outbound traffic. Additionally, Virtual Network Peering is configured between the East US and West US VNets to enable seamless and secure communication across regions. A VPN Gateway is also implemented allowing secure remote connectivity with the VNet for management and administrative access.
<br />

<h2>Technologies Used in This Project</h2>

**Azure Infrastructure**
- Azure Virtual Machines (*EastUS-VM-HA / WestUS-VM-HA*)

**Networking**
- Azure Virtual Networks (*Prod-VNet-EastUS / Prod-VNet-WestUS*)
- Azure Virtual Network Peering
- Virtual Network Gateway (VPN)
- Azure Load Balancer
- Azure Traffic Manager

**Security & Access Control**
- Network Security Groups (NSGs)
- Subnet isolation: (*Web, App & DB*)

**Windows**
- Windows Server 2019
- PowerShell
- RDP
- Certificate Manager (certmgr)

<h2>Deployment and Configuration Steps</h2>
<h3 align="center">In Azure (portal.azure.com)</h3>
<br />

In the **Basic Tab**, Create a VM (*EastUS-VM-HA*) in the East US region. Create a Username (*AdminEast-EL*) & strong password </br>
Select *"none"* for public inbound ports (***This is to enhance security by limiting direct exposure to the internet***)

![Screenshot 2025-03-29 220219](https://github.com/user-attachments/assets/e42616e7-85f3-4d0f-8816-0bff03a35e3e)
![Screenshot 2025-03-29 220305](https://github.com/user-attachments/assets/0b49f4f8-1f9c-4673-bfc8-f394526a72c3)
![Screenshot 2025-03-29 220322](https://github.com/user-attachments/assets/5712478c-34c5-4b7b-855e-0d0362ca390c)
<br></br>

In the **Disk Tab**, create a new disk & set the disk size to **64GB** for extra storage (Premium SSD) </br>
(*Added a disk to store logs & application data separately but it's not needed*)

![Screenshot 2025-03-29 223926](https://github.com/user-attachments/assets/933c7690-66fb-4766-a2b7-bc12ca00dcce)
<br></br>

In the **Networking Tab**, create a VNet (*Prod-VNet-EastUS*) associated with your VM and its' designated region. Within the VNet, setup a **Web-Subnet** (*10.30.1.0/24*) to isolate & manage web traffic. Do not assign a public IP directly, the **Load Balancer** will handle external access

![Screenshot 2025-03-29 230548](https://github.com/user-attachments/assets/7d760980-bc52-414e-b9bd-76f95445e9dc)
![Screenshot 2025-03-29 230548](https://github.com/user-attachments/assets/5da813a6-77cf-4216-bf5f-504b2eae7e25)
<br></br>

In the **Management Tab**, select **Azure-Orchestrated** for the patch option if you would like for it to be centralized & managed by Azure </br>
(*This ensures consistency, compliance and high availability across all the VMs. Allows automatic patching and less manual efforts*)
![Screenshot 2025-03-29 233256](https://github.com/user-attachments/assets/81e4c7dd-aebe-4451-9dfb-13c136e70f5f)
<br></br>

In the **Monitoring Tab**, enable application health monitoring which will monitor the health of applications running inside the VM 

![Screenshot 2025-03-29 233603](https://github.com/user-attachments/assets/c5579ab1-a2a4-459f-8941-029fc9bdf544)
<br></br>

In the **Tags Tab**, create your own form of key-value pairs. Then select **create** </br>
(*Creating tags help organize, manage & track resources very effectively*)

![Screenshot 2025-03-29 230548](https://github.com/user-attachments/assets/b231a9fa-ec33-4368-9850-d339f4c48723)
![Screenshot 2025-03-29 235054](https://github.com/user-attachments/assets/9318e008-d4bc-433b-9bbb-02c2d6022d79)
<br></br>

**Repeat these steps & create another VM in the WestUS region** </br>
(Deploy a secondary VNet in the WestUS for failover)
  - **Virtual Machine** (*WestUS-VM-HA*)
  - **Virtual Network** (*Prod-VNet-WestUS*)
      - Web-Subnet: *10.40.1.0/24*
  
![Screenshot 2025-03-30 013121](https://github.com/user-attachments/assets/ee312efd-2799-4e7b-bd34-91729455d436)

In the **East US VNet** (*Prod-VNet-EastUS*), add a **App-Subnet** (*10.30.2.0/24*) & **DB-Subnet** (*10.30.3.0/24*) </br>
<code style="color : blue">(*Ignore the errors*)</code>

![image](https://github.com/user-attachments/assets/73aa8b35-6fc2-430a-b96e-c4cc440cc012)
![image](https://github.com/user-attachments/assets/9aeb199d-e177-4329-9eb0-69067e49210d)

**Repeat these steps for the West US region** (*Prod-VNet-WestUS*)
 - **App-Subnet**: 10.40.2.0/24
 - **DB-Subnet**: 10.40.3.0/24

![image](https://github.com/user-attachments/assets/51a5807d-e39b-4811-9d65-85e22867a0ff)

Create a **Load Balancer** (*EastUS-LB*) for the designated region & select **'Public'** </br>
(*Allows incoming traffic from the internet to be distributed to the backend VM*) 

![Screenshot 2025-03-30 014300](https://github.com/user-attachments/assets/2bb6aaf5-b335-4466-a039-620de6cfe32f)

Add a **frontend IP**. Click '*Create new*' and name the IP address (*PublicWebAppIP-EastUS*)
![Screenshot 2025-03-30 100926](https://github.com/user-attachments/assets/47130496-6fa2-46a0-90cc-aba1c1c29f8f)

Add a **backend pool** (*EastUS-BackendPool*). Select the designated VNet. Click **'add'** under IP configurations </br>
Select the VM that will be associated to the Load Balancer. When done, click 'add' then **save**
![Screenshot 2025-03-30 102242](https://github.com/user-attachments/assets/a770d4f1-cc6f-4bfb-a23d-b191d6cb2659)

Create a name for the **load balancing rule** (*HTTP-LB-Rule*). Select the associated frontend IP address & backend pool. Since this rule is for HTTP traffic, use port 80 as the input. Then '*Click new*' to create a **Health probe**

![Screenshot 2025-03-30 103128](https://github.com/user-attachments/assets/c2becebd-0783-4165-acab-b1a813376ebd)

Name the **Health probe** (*HealthProbe-80*) In this case, using port 80 with the HTTP protocol will monitor the health & status of HTTP traffic on the VM in your backend pool

![Screenshot 2025-03-29 230548](https://github.com/user-attachments/assets/12c69c24-35fc-45d6-a6e3-1ac57e7b76d1)
<br></br>

Repeat and create another **load balancing rule** (*HTTPS-LB-Rule*) with a **Health probe** for HTTPS (*HealthProbe-443*)

![Screenshot 2025-03-30 104812](https://github.com/user-attachments/assets/84a993c9-8656-4998-9ad1-5446d397ea75)

Include the same key-value pairs for the tags as before and **create** the load balancer

![Screenshot 2025-03-30 110302](https://github.com/user-attachments/assets/4a87efac-ae8a-4d28-a5ff-625d8d0ace76)

**Repeat these steps & create another Load Balancer in the WestUS region**
 - Load Balancer name (*WestUS-LB*)
 - Frontend IP Configuration (*PublicWebAppIP-EastUS*)
 - Backend Pool (*EastUS-BackendPool*)
 - Load balancing rule with health probes (*HTTP/HTTPS*)
</br> 

Search **Traffic Manager** in the portal. Create a Traffic Manager Profile (*GlobalTrafficMgr*)

![Screenshot 2025-03-30 220547](https://github.com/user-attachments/assets/17ef6b70-8a5d-454d-bb17-64cb4d016d4f)
![Screenshot 2025-03-30 222128](https://github.com/user-attachments/assets/ad513c3a-2570-4dba-88e9-4dda39281073)

Head to **Endpoint** in your Traffic Manager profile. Create an endpoint (*EastUS-LB-Endpoint*) & associate it with the assigned Public IP address </br> 
***Tip**: When creating an endpoint, if you are using a public IP address as the target resource, you must assign a *DNS name label* to that public IP address in Azure

![Screenshot 2025-03-30 225034](https://github.com/user-attachments/assets/3658066d-4d31-46f2-817e-dcb3a32e07f6)
![Screenshot 2025-03-30 224804](https://github.com/user-attachments/assets/fe5e7d68-31e5-4d17-8f42-0c8a6cbccbb0)

**Repeat and create an Endpoint for the WestUS region (*WestUS-LB-Endpoint*)**

![Screenshot 2025-04-04 165537](https://github.com/user-attachments/assets/ade85403-b618-4a23-acca-3517da95651d)
</br>

Implement a Virtual Network Peering (Peer EastUS to WestUS) </br>
**Azure Portal > Search Virtual Networks > Select the VNet (Prod-VNet-EastUS) > Settings > Peerings** </br>
<code style="color : blue">(*Ignore the error*)</code>

![Screenshot 2025-04-07 133818](https://github.com/user-attachments/assets/bdb39a6c-1c93-4d19-a86e-3569366e9cfc)
![Screenshot 2025-04-07 134914](https://github.com/user-attachments/assets/889c2433-d8b0-4efa-8f3e-342fc967e7bf)
![Screenshot 2025-04-01 122926](https://github.com/user-attachments/assets/08e28f5f-383b-47f9-a8f9-21e2c08a8ebf)
<br></br>

Create a **NSG** and **apply an inbound rule** to allow **HTTP** & **HTTPS** traffic coming from the public IP address of the **Load balancer** (*EastUS-LB*) </br>
**Azure Portal > Search Network Security Groups (*WebSubnet-EastUS-nsg*) > Click 'Create'** </br>

![image](https://github.com/user-attachments/assets/4e5b1435-318d-446d-8903-cb87fb01ff3f)
![Screenshot 2025-04-01 134911](https://github.com/user-attachments/assets/e73c09da-6fe3-428f-9b06-d4b04f7ddc2a)
![Screenshot 2025-04-01 135704](https://github.com/user-attachments/assets/1eb880c0-26fb-402e-9f83-983db4c506f8)

Create another **inbound rule** to allow secure **administrative access** (*Local Machine*) to VMs in the Web-Subnet</br>
(Don't forget to do the same and repeat for the ***West US region***) 

![Screenshot 2025-04-01 162638](https://github.com/user-attachments/assets/5755d9a0-f38f-447b-afe5-f38426440d07)

Create **NSGs** for the **App-Subnet** and **DB-Subnet** in both regions. Then associate each **NSG** with their corresponding subnet under *setting*

![Screenshot 2025-04-01 164312](https://github.com/user-attachments/assets/c51a1b38-b360-48eb-a46f-ae54ded50995)
![Screenshot 2025-04-01 164340](https://github.com/user-attachments/assets/5116f99f-2b67-47b9-94f9-ca317e77d3c5)
![Screenshot 2025-04-01 164522](https://github.com/user-attachments/assets/0f352dd8-38d1-41d9-a5c6-ce0ac04ccd63)

Create a **inbound rule** (*Allow-WebToApp*) to allow web traffic from the **Web-Subnet** (*10.30.1.0*) to reach the **App-Subnet** </br>
(*Any VM in the Web-Subnet can initiate traffic to VMs in the App-Subnets. Without this rule, the NSG would block the traffic by default*) 

![Screenshot 2025-04-01 172311](https://github.com/user-attachments/assets/7b07865a-585b-4991-8668-1e73e054e884)

Create a **inbound rule** in the **AppSubnet-EastUS-nsg** which allows traffic from the **App-Subnet** to reach the **DB-Subnet** </br>
(*Allows application servers in the App-Subnet to initiate connection to the database servers in the DB-Subnet*)

![Screenshot 2025-04-01 172912](https://github.com/user-attachments/assets/0039a168-34d6-4641-bd5e-cd341b401d59)

Create a **inbound rule** in the **DBSubnet-WestUS-nsg** to allow traffic from the *10.30.2.0/24* IP range to **destination ports 1433 & 3306** </br>
(*This enables secure database access from the application subnet*)

![Screenshot 2025-04-02 115752](https://github.com/user-attachments/assets/54f19ac0-e3ad-4d5e-aaa6-a325a42f631e)

Create a **inbound rule** in the **DBSubnet-WestUS-nsg** to deny traffic from the **Web-Subnet** (*10.40.1.0/24*) </br>
(*This prevents unauthorized access from the Web-Subnet to the DB-Subnet*)

Create a DDoS Portection Place

![Screenshot 2025-04-02 115917](https://github.com/user-attachments/assets/90326907-b615-49cc-9b42-31961b0b0549)

**<h1>Since both of the VM's were![Uploading Screenshot 2025-04-02 115917.png…]()
 created with a private IP address**</h1>
 - **EastUS-VM-HA**: 10.30.1.4
 - **WestUS-VM-HA**: 10.40.2.4 </br>
 
<h2>There are 2 ways to connect:</h2>
1. **A jumpbox VM** within the VNet (Gives you a public-facing entry point into the network, apply NSG to restrict unknown IP addresses) </br>
2. **Using a VPN** (VPN Gateway configures a P2S VPN connection from a local machine to the Virtual Network) </br>
***(You can't access VMs with private IPs directly from the internet)***

<h1>Jumpbox VM</h1>

Create a VM (*TestVM*) in the **EastUS VNet** and connect to it via RDP from a local machine 

![Screenshot 2025-03-31 100952](https://github.com/user-attachments/assets/95ed09eb-2981-4769-8a51-1dc6bbfcb6b8)

After deploying the VM (*TestVM*) into the **Web-Subnet** (*10.30.1.0/24*), it will be assigned a private IP address (*10.30.1.7*) from that range </br>
***(When running ipconfig /all, you are seeing the VM’s internal IP address, not its public IP)***

![Screenshot 2025-04-07 154420](https://github.com/user-attachments/assets/79fe1846-8db4-4092-badd-a48d1c566deb)

Remote into ***EastUS-VM-HA*** through the TestVM via RDP (*Private IP*: ***10.30.1.4***)

![Screenshot 2025-04-07 162624](https://github.com/user-attachments/assets/563f258f-9848-45e1-87fd-dd7582986afa)
![Screenshot 2025-04-07 162832](https://github.com/user-attachments/assets/64774458-8d10-42ae-9ad1-58a067a4c9d3)

Ping **EastUS-VM-HA** (*10.30.1.4*) from the local machine</br>
(Pings from my local machine or any device outside the VM's virtual network will not reach to the VM)

![Screenshot 2025-04-07 163109](https://github.com/user-attachments/assets/c9c3aeaa-9607-4851-8587-c34a4cf6e8c5)

Login to **EastUS-VM-HA** and make sure *ICMPv4* is enabled on the *local Windows Firewall*</br>
Then ping **EastUS-VM-HA** (*10.30.1.4*) through the TestVM since only machines within the same VNet can connect to it </br>

![Screenshot 2025-04-07 163322](https://github.com/user-attachments/assets/12ddbda1-ae1b-47d2-a4c8-3c686ce21c93)

<h1>Azure Virtual Network Gateway (VPN)</h1>

Create a **VPN** (*EastUS-VPN*) </br>
**Azure Portal > Search Virtual Network Gateway > Select the Create** </br>
<code style="color : blue">(*Ignore the error*)</code>

![Screenshot 2025-04-08 102553](https://github.com/user-attachments/assets/61b74575-3f67-4f01-b546-a2b8c5785641)
![Screenshot 2025-04-08 102605](https://github.com/user-attachments/assets/d313688d-fbb1-402c-a5a8-a2ca4cf9b8d2)

Set up a Point to Site VPN access after creating a VPN in Azure </br>
**In Virtual Network Gateways > Select the VPN (*EastUS-VPN*) > Settings > Point-to-site Configuration** </br>
- Used ***192.168.100.0/24*** as the address pool so it wont overlap with the **local network** (*172.20.10.x*) & the **VNet subnet** (*10.30.x.x.*) </br>
- **IKEv2** & **OpenVPN** is a great Windows option delivering flexibility across devices. </br>
- **Azure Certificate Authentication** does not require additional infrastructure and uses certificates to verify clients (**Secure & Manageable**)

![Screenshot 2025-03-31 164132](https://github.com/user-attachments/assets/6735ef2c-91c6-4e45-8616-c1b90ddd9886)

Generate a **self-signed root certificate** (*AzureVPNRootCert*) on the local machine </br>
Open PowerShell and run ***$cert = New-SelfSignedCertificate*** command to generate a self-signed root certificate 

![Screenshot 2025-03-31 164438](https://github.com/user-attachments/assets/c73b2d9f-8a08-4301-b011-85056399dde2)

Open **Run** (*Win + R*), type ***certmgr.msc*** and **Enter** or type in *certificate* in the Windows search bar and select ***'Manage user certificates'***</br>
Navigate to **Personal** > **Certificates**. Find ***AzureVPNRootCert.cer*** and right-click, choose **'All Task' > Export**

![Screenshot 2025-03-31 140038](https://github.com/user-attachments/assets/d2371880-ae2e-4108-9557-e479b59358c1)

**Do not export the private key** with the certificate & choose **Base-64 encoded X.508 (.CER)** as the file format.  </br>
Save the file as **AzureVPNRootCert.cer**

![Screenshot 2025-03-31 140111](https://github.com/user-attachments/assets/effd2d20-5be4-47e7-a340-3978158726ec)
![Screenshot 2025-03-31 140136](https://github.com/user-attachments/assets/30c5afa0-c053-40c6-9f16-1ed687448ab6)
![Screenshot 2025-03-31 140205](https://github.com/user-attachments/assets/21c121dd-bf84-4750-90c7-136f93476136)

Open up **AzureVPNRootCert.cer** in *Notepad* </br>
Copy the entire text *excluding* **---Begin Certificate---** & **---End Certificate---**

![Screenshot 2025-03-31 140738](https://github.com/user-attachments/assets/58a51f2b-237d-4420-81e7-6c0593c6800b)

Paste it in the **Public certificate data** box

![image](https://github.com/user-attachments/assets/650cd128-196b-4468-a338-e345cb669249)

Generate a **client certificate** (*AzureVPNClientCert*) for the local machine </br>
Open PowerShell and run ***$cert = New-SelfSignedCertificate*** command to generate a client certificate

![Screenshot 2025-03-31 164510](https://github.com/user-attachments/assets/a5d1661f-014d-4c5d-8992-e2e05acbb721)
![image](https://github.com/user-attachments/assets/8e2fca98-a757-4efb-b82a-1a11e1cf0522)

**Export the private key** with the certificate & choose **Personal Information Exchange - PKCS #12 (.PFX)** as the file format.  </br>

![Screenshot 2025-03-31 164546](https://github.com/user-attachments/assets/f182c1bf-d210-4b5d-891c-230cd65a8af4)
![Screenshot 2025-03-31 164627](https://github.com/user-attachments/assets/4247d876-d28d-4bb4-a361-9bdd2ea9e383)

Create a **strong password for the private key** and change the encryption to ***AES256-SHA256*** (This will ensure the VPN setup is secure)</br>
(**Tip**: AES256-SHA256 is more secure and uses stronger encryption algorithm. SHA1 is considered weak and outdated, while AES is widely used and recommened for modern security practices) </br>

![Screenshot 2025-03-31 165135](https://github.com/user-attachments/assets/0770d72b-e2a4-422f-8401-3fb6f2fea99e)

Download the **VPN Client** & extract it

![Screenshot 2025-04-08 154407](https://github.com/user-attachments/assets/99360b55-964a-4211-b8e3-fb6158166628)
![Screenshot 2025-03-31 165957](https://github.com/user-attachments/assets/515c5e32-1974-402f-8a3e-3e04d69e2bef)

You extracted the VPN client and now see 5 folders. Proceed to the **WindowsAMD64 folder** if you're using a 64-bit Windows system & install

![Screenshot 2025-03-31 170020](https://github.com/user-attachments/assets/4439e588-c1a6-4cfa-9aee-d128abcb19fe)
![Screenshot 2025-03-31 170046](https://github.com/user-attachments/assets/f3f26232-affa-4608-a08b-c18a02e8ec02)
![Screenshot 2025-03-31 170241](https://github.com/user-attachments/assets/e9a2295f-92aa-4837-9628-ae47f2bd8fd3)

Connect to **Prod-VNet-EastUS**

![Screenshot 2025-03-31 170705](https://github.com/user-attachments/assets/99dc43fc-b0b0-4fdc-8728-454cad7353c6)
![Screenshot 2025-03-31 170746](https://github.com/user-attachments/assets/a8a825d1-0a6b-46f7-96ae-ccea55bbbfda)
![Screenshot 2025-03-31 170809](https://github.com/user-attachments/assets/51e87493-c6fa-4a7c-b2a0-dbbf129abbb1)
![Screenshot 2025-03-31 171122](https://github.com/user-attachments/assets/0996d578-3f1d-4c76-a308-546d14e4764e)
![Screenshot 2025-03-30 224804](https://github.com/user-attachments/assets/88ca360a-6c2e-42dd-801a-ce1937a30b5a)
![Screenshot 2025-03-31 171747](https://github.com/user-attachments/assets/90fb18a2-91f6-442c-a28a-9fbcd053bc19)
![Screenshot 2025-03-31 171441](https://github.com/user-attachments/assets/061c1acb-6678-4863-acb1-576c65cf1cee)

I hope this tutorial helped you learn a little bit about Network Security Protocols, subnets, Virutal Networks, load balancers, Virtual network gateway & Virtual Machines
Now that we're done, DON'T FORGET TO CLEAN UP YOUR AZURE ENVIRONMENT so that you don't incur unnecessary charges.
Close your Remote Desktop connection, delete the Resource Group(s) created at the beginning of this tutorial, and verify Resource Group deletion.



Virtual network peering was implemented to enable seamless and secure connectivity between the East US and West US regions. In addition, to enhance security, NSGs were applied with security rules, custom subnets & a Virtual Network Gateway (VPN) was configured to securely connect an on-premises resource to Azure. 

This project focuses on deploying a highly available web application across two Azure regions using Load Balancers & Traffic manager for cross-region failover. Load balancing was configured for both HTTP and HTTPS protocols, each paired with a health probe to monitor backend VM responsiveness and ensure service availability. The environment is segmented using custom subnets which improves manageability and security. Web-Subnet, which hosts the frontend VM, receives user traffic. App-Subnet, which handles the application server and is only accessible internally. DB-Subnet, which securely stores & manages the database server using private endpoints for security.







