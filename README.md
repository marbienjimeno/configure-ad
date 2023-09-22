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
- Step 4: Create an Admin account in Active Directory.
- Step 5: Join our Client-1 machine to our domain (mydomain.com).
- Step 6: Setup Remote Desktop for non-administrative users on Client-1.
- Step 7: Create additional users and attempt to log into our Client-1 machine as on of the users.

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
  Use Remote Desktop to connect into our DC-1 VM. Log in using the username and password we created when setting up our DC-1 VM. Once connected, search "wf.msc" into the search bar and open the Microsoft Common Console Document. Once in the Windows Defender Firewall with Advanced Security window, click on Inbound Rules on the left sidebar menu. Enable the two Core Networking Diagnostics - ICMP Echo Request (ICMPv4-In).
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/2746c3f6-6fb0-4963-8e10-34225fe6c60d)

<p>
  Go back inside our Client-1 VM. We see that our pings have stopped timing out and are successfully being recieved by DC-1. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/d5aa9e3d-ce5c-485f-8225-d66947aafcf7)

**Step 3: Install Active Directory.**
<p>
  After confirming cinnectivity between our Client-1 and DC-1 machines, we'll start installing Active Directory on our domain controller. Go to our DC-1 machine and open Server Manager. Click on Add roles and features. Click Next until you get to Server Roles. Select Active Directory Domain Services (AD DS) and add the features required for AD DS. Click Next until you are able to install AD DS.
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/3fe59211-43e4-420a-9703-027c9ccbb5db)

<p>
  Once AD DS has finished installing, click on the flag icon on the top-right menu. Click on Promote this server to a domain controller. In Deployment Configuration, select Add a new forest. For Root domain name, enter "mydomain.com". Click Next.
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/e82673f6-866b-4316-ac35-6313b51cf934)

<p>
  In the Domain Controller Options settings, enter a password for Directory Services Restore Mode (DSRM). We will not be using this password for the rest of the lab. Click Next until we are able to proceed with the installation. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/ee1610a4-dbec-48e7-8b26-e7573c482d0d)

<p>Once the installation has finished, you will be prompted to restart the machine.</p>

**Step 4: Create an Admin account in Active Directory.**
<p>
  Now that we've installed Active Directory, we will create an administrator account. Use Remote Desktop to connect into our DC-1 machine. This time, we'll log in using with the username "mydomain.com\labuser". Our labuser account is now under our newly created domain. Use the same password. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/442af1d2-c4b8-4726-ba1c-d12861e767e8)

<p>
  On the top right menu, click on Tools. Select Active Directory Users and Computers. Create two new Organizational Units (OUs) inside mydomain.com. One will be named "_EMPLOYEES" and will hold non-administrative users.The other will be named "_ADMINS" and will hold users with administrative privileges. Right-click on mydomain.com and select Refresh. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/d421b218-2840-447c-92c2-e24143f989f2)

<p>
  Create a new User inside the _ADMINS OU. This new admin user will be named "Jane Doe". For User logon name, we'll use "a-jane.doe". The "a-" indicated that Jane Doe is an admin account. Enter a password for Jane's user account. Make sure to remember Jane's username and password for later reference. Uncheck the box stating "User must change password at next logon". 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/fe93ca05-0418-4355-a026-5fb694f684af)

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/7d1e2448-8d3b-4e77-9e3d-30a0aa01e02a)

<p>
  Richt-click on Jane Doe and select Properties. Go to the Member of tab and Click Add. Enter "domain admins" and click Check Names. Click OK. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/5bf0fe40-443a-4644-985a-9a31a926f3f9)

<p>
  Log out and close the Remote Desktop Connection to our DC-1 VM. Log back in using Jane Doe's admin account. For User name, enter "mydomain.com\a-jane.doe". Enter Jane Doe's password. We will continue this lab using Jane's admin account.
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/901f9a7c-3f93-429f-87f0-6e3f068fc316)

**Step 5: Join our Client-1 machine to our domain (mydomain.com).**
<p>
  Next, we'll join our Client-1 machine to our mydomain.com domain. We'll need to set our Client-1's DNS settings to our DC-1 machine's Private IP address. Client-1 will need to use DC-1 as its DNS server in order to be added into its domain. Inside the Azure Portal, go to our Client-1 virtual machine. Under Settings on the left sidebar menu, select Networking. Click on the Network Interface. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/4400603d-a8da-4abb-9d87-1b7c06e2e136)

<p>
  Under Settings on the left sidebar menu, select DNS servers. We'll set DC-1's private IP address as Client-1's DNS server. Select Custom and enter "10.0.0.4" under DNS server. Click Save.
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/6a41b04d-83d9-43bc-b09a-194b4fb0cc08)

<p>
  Go back to the Client-1 virtual machine and click Restart. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/40ff4362-768c-4851-a20a-3385198709b9)

<p>
  Using Remote Desktop, connect into the Client-1 machine. Login using our labuser account. Right-click the Start icon and select System. Then select Rename this PC (advanced) on the right side menu. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/1c1f396a-4d7f-49c5-891a-92598597453f)

<p>
  Inside System Properties, click on Change.... Under Member of, select Domain and enter "mydomain.com". Click OK. When prompted, enter Jane Doe's admin account credentials to get permission to change the domain. For User name, enter "mydomain.com\a-jane.doe" and enter Jane's password. When the domain change has completed, the machine will be prompted to restart. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/2390e418-bf9c-4aae-a10d-b5d9c3612e17)

**Step 6: Setup Remote Desktop for non-administrative users on Client-1.**
<p>
  For now, only domain admins are able to log into Client-1 and use its resources. Next, we'll set it so that non-administrative users can also log into Client-1. Log into Client-1 using Jane Doe's admin account. Right-click on the Start icon and select System. Inside Settings, select Remote Desktop. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/59f07fca-dae5-4488-8de5-b77c72e13936)

<p>
  Under User Accounts, click on Select users that can remotely access this PC. Inside Remote Desktop Users, click Add. Enter "domain users" and click on Check Names. Click OK.
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/9551d655-d283-4823-aaa8-69c133fbc940)

**Step 7: Create additional users and attempt to log into our Client-1 machine as on of the users.**
<p>
  Now that non-administrative users can log into Client-1, we will create additional normal users. Go to the DC-1 VM and open Windows Powershell ISE as an Administrator. Click on the New File icon. Copy and paste the Powershell script from the following GitHub link: [https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1). This powershell script will create 10000 new normal Users in our Active Directory with randomly generated names and a default password. The new Users will be placed in the _EMPLOYEES OU in our Active Directory. Click Run to run the script. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/8b223ad4-32d9-4d79-bc51-d49be75b561a)

<p>
  We can confirm that the new Users have been successfuly created by looking inside the _EMPLOYEES OU in Active Directory Users and Computers. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/74ed57af-54c0-4681-a03a-80a0dbeb1511)

<p>
  Now we'll attempt to log into our Client-1 machine using one of the new User accounts. Log off Client-1 and log in using one of the new User's credentials. 
</p>

![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/2199e2c7-4a1f-43f7-8e80-2a34784a5d17)

 <p>
   We have successfully logged into Client-1 using one of the new User accounts.
 </p>

 ![image](https://github.com/marbienjimeno/configure-ad/assets/29347863/84753782-7ad5-47d1-bbe9-37875f9c0b02)

**Conclusion**
<p>
  In this lab, we successfully implemented an on-premises Active Directory system using Azure Virtual Machines. 
</p>
