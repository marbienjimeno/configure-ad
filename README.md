<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Video Demonstration</h2>

- ### [YouTube: How to Deploy on-premises Active Directory within Azure Compute](https://www.youtube.com)

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Step 1: Set up our resources in Azure.
- Step 2: Ensure connectivity between our client machine and the Domain Controller.
- Step 3: Install Active Directory.
- Step 4: Create an Admin account and a normal User account in Active Directory.
- Step 5: Join our Client-1 machine to our domain (mydomain.com).
- Step 6: Create additional users and attempt to log into our CLient-1 machine as on of the users.

<h2>Deployment and Configuration Steps</h2>

**Step 1: Set up our resources in Azure.**
<p>
  We'll start by creating a new virtual machine. For Resource group, create a new Resource group and name it "AD-Lab". We will name this virtual machine "DC-1". This VM will act as our Domain Controller. For Region, we'll select (US) West US 3. For Image, select Windows Server 2022 Datacenter: Azure Edition - x64 Gen2.
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/ae13f8ea-d30f-4799-8ac3-2143501418ea)

<p>
  For Size, select Standard_E2s_v3 - 2 vcpus, 16 GiB memory. For Username, we'll use "labuser". Enter a password that satisfies the requirements. Make sure to record this VM's username and password for later reference. Click Review + create. After the validation process has passed, click Create. Wait for our DC-1 VM to be successfully deployed before creating another new VM. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/4bf81228-4b39-4ab9-830a-7374f442354a)

<p>
  Once our DC-1 VM has been successfully deployed, create another Virtual machine. For Resource group, select AD-Lab. We want to put this VM in the same resource group as our DC-1 VM. We'll name this virtual machine "Client-1". This VM will act as our client machine where non-administrativ users can log in and use computer resources. For region, select (US) West US 3. For Image, select Windows 10 Pro, version 22H2 - x64 Gen2. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/d315639f-6d80-415b-b3ef-6757398388a7)

<p>
  For Size, select Standard_E2s_v3 - 2 vcpus, 16 GiB memory. For Username, we'll use "labuser". enter a password that satisfies the requirements. Make sure to record the username and password for this VM for later reference. Check the box stating "I confirm I have an eligible Windows 10/11 license with multi-tenant hosting rights."
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/4bfbfd34-c18a-439a-9b2e-26de1b2ca604)

<p>
  Go to Networking settings. For Virtual network, make sure DC-1-vnet is selected. We want to put our DC-1 and Client-1 machines in the same virtual network. We'll use the default assigned subnet and the automatically created Client-1-ip for Public IP. Click Review + create. When the validation process has passed, click Create. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/e2a537b2-bbdb-4774-b76f-255e7f3c133d)

<p>
  We want to set our Domain Controller machine's NIC private IP address to be static. Go to our DC-1 virtual machine. Under Settings on the sidebar menu, select Networking. Click on the Network Interface. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/1ca9a410-50c4-4bbd-843b-34593e86aaf0)

<p>
  Under Settings on the left sidebar menu, select IP configurations. Click on ipconfig1 and change the Allocation setting to Static. Click Save.
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/8da57168-edb5-4594-821f-334c1a315ff2)

**Step 2: Ensure connectivity between our client machine and the Domain Controller.**

<p>
  After successfully creating our DC-1 and Client-1 VMs, we will configure and confirm connectivity between the two machines. Using Remote Desktop, connect into our Client-1 VM. Log in using the username and password we created when setting up our Client-1 VM. Open Command Prompt. Sent a perpetual ping to our DC-1 machine's private IP address by entering the command "ping 10.0.0.4 -t". We see that the pings are not being recieved as DC-1 is blocking incoming ICMP traffic. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/a0c10295-221f-42ca-8835-197b18788658)

<p>
  Use Remote Desktop to connect into our DC-1 VM. Log in using the username and password we created when setting up our DC-1 VM. Once connected, serach "wf.msc" into the search bar and open the Microsoft Common Console Document. Once in the Windows Defender Firewall with Advanced Security window, click on Inbound Rules on the left sidebar menu. Enable the two Core Networking Diagnostics - ICMP Echo Request (ICMPv4-In).
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/2746c3f6-6fb0-4963-8e10-34225fe6c60d)

<p>
  Go back inside our Client-1 VM. We see that our pings have stopped timing out and are successfully being recieved by DC-1. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/d5aa9e3d-ce5c-485f-8225-d66947aafcf7)

**Step 3: Install Active Directory.**
