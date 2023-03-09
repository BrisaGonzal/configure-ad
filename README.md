<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This lab outlines the implementation of on-premises Active Directory within Azure Virtual Machines. This will demonstrate how to setup Active Directory and create users who will be able to login to the domain through a virtual machine that is connected to the domain controller through DNS.<br />

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Setup Resources in Azure
- Ensure Connectivity between the client and Domain Controller
- Install Active Directory
- Create an Admin and Normal User Account in AD
- Join Client-1 to your domain (mydomain.com)
- Setup Remote Desktop for non-administrative users on Client-1
- Create a bunch of additional users and attempt to log into client-1 with one of the users

   <h2>Deployment and Configuration Steps</h2>

<p>
<img src="https://i.imgur.com/DKdQcxW.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Setup Resources in Azure:

Create a new virtual machine in Azure with the settings above. Allow for the new resource group to be created and name it AD-Lab. Name the virtual machine "DC-1". This is going to be the domain controller. A domain controller is a server or computer that has active directory installed on it.
</p>
<br />

<p>
<img src="https://i.imgur.com/ok0qqqF.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Next, create a new virtual machine that is going to be used as the client with the same credentials as the previous VM. Make sure they are both in the same region.
</p>
<br />

<p>
<img src="https://i.imgur.com/jbIUSW9.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Now, set the domain controller's NIC Private IP address to be static. This makes sure the public IP address doesn't change.
  
Virtual Machines > DC-1 > Networking > Network Interface: dc-1853 > IP Configurations > ipconfig1 > Assignment > change to "Static" > Save
</p>
<br />

<p>
<img src="https://i.imgur.com/2nCH6Ba.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Ensure Connectivity between the client and Domain Controller:

Login to Client-1 and ping DC-1's IP address with ping -t (perpetual ping)
  
Copy the public IP address of the Client-1 VM and log in using Microsoft Remote Desktop > copy DC-1's private IP address to be able to ping it from the client > open Command Propmpt in CLient-1 VM > type " ping -t 10.0.0.4" 

10.0.0.4 is DC-1's private IP address. Notice how it continues to time out. This is because DC-1's Windows firewall is blocking ICMP traffic.
</p>
<br />

<p>
<img src="https://i.imgur.com/arUTXXe.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Login to the DC-1 VM and enable ICMPv4 (ICMPv4 is the protocol that ping uses) in on the local windows firewall.
  
Log in to the VM > start menu > type "wf.msc" > Inbound Rules > Click "Protocol" > right click and enable both ICMP Echo Requests
</p>
<br />

<p>
<img src="https://i.imgur.com/7BFItsJ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Check back at Client-1 to see the ping succeed. You can see Client-1 is now receiving a response from DC-1 because we enabled the IMCP traffic. If you use "ctrl-c" in the command prompt it will cancel the perpetual ping.
</p>
<br />

<p>
<img src="https://i.imgur.com/gOcavQO.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Install Active Directory:
  
In DC-1: click start > type "Server Manager" > Add roles and features > continue until you get to "Server Roles" in the installation screen > check "Active Directory Domain Services" > Add Features > "Next" through any of the dependencies > Install
</p>
<br />

<p>
<img src="https://i.imgur.com/IiSyTby.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Now a domain needs to be set up. 
  
Click on the yellow triangle on the upper right corner that will show up once Active Directory Domain Services is installed > click on "Promote this server to a domain controller" > Add a new forest > name the Root domain "mydomain.com" > next > Create a password "Password1" > next through any dependencies > Install
  
DC-1 will automatically restart when it is finished installing. Log back in to the DC-1 vm in the remote desktop using mydomain.com\labuser in the username space to log in.
</p>
<br />

<p>
<img src="https://i.imgur.com/fcMzwoi.png" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Create an Admin and Normal User Account in AD:
  
Server Manager > Dashboard > Tools > Active Directory Users and Computers > right click "mydomain.com" > New > Organizational Unit 
  
Create Organizational units: _EMPLOYEES and _ADMINS
</p>
<br />

<p>
<img src="https://i.imgur.com/hQZ2gBk.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
To create a new admin account: _ADMINS > right click empty space > New > User > name "Jane Doe" > user logon name: "jane_admin" > Next > Set a password and set it to "never expires" for practice pursposes > Next > Finish
</p>
<br />

<p>
<img src="https://i.imgur.com/Je9JygR.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
In order to actually make the user a domain admin, we must assign it to the domain admins group. 
  
_ADMINS > right click "Jane Doe" > Properties > Member of > Add > type in "Domain" > check names > Domain Admins > OK > Apply > OK
  
Log out of DC-1 and log back in as "mydomain.com\jane_admin" with the credentials used to create the admin account. Use jane_admin as your admin account from now on.
</p>
<br />

<p>
<img src="https://i.imgur.com/QldFnXo.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
Join Client-1 to your domain (mydomain.com):

We need to use DC-1 as the DNS.

Azure > Virtual Machines > Client-1 > Networking > Network interface: > DNS servers > click "custom" > insert DC-1's private IP Address (in this case 10.0.0.4) > Save

Restart Client-1 : Virtual Machines > Client-1 > Restart

Log back in to Client-1 VM with labuser.
</p>
<br />

<p>
<img src="https://i.imgur.com/lPq9H88.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Client-1 vm > right click start > System > Rename this PC (advanced) > Change > click "Domain:" > type "mydomain.com" > OK > type in "mydomain.com\jane_admin" and pass word > OK
  
Client-1 will restart. Now Client-1 is joined to the domain. Now it is possible to log into Client-1 vm with Jane Doe's admin account because Client-1 is now a member of the domain.
</p>
<br />

<p>
<img src="https://i.imgur.com/wVeZMlI.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Setup Remote Desktop for non-administrative users on Client-1:
  
Log back in to Client-1 as jane_admin.
  
Right click start > System > Remote Desktop > Select users that can remotely access this PC > Add > type in "domain users" > Check names > OK > OK
</p>
<br />

<p>
<img src="https://i.imgur.com/PWLSwXQ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Create a bunch of additional users and attempt to log into Client-1 with one of the users:
  
Login to DC-1 as jane_admin > open powershell_ise as an administrator > copy the contents of this script https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1 > create a new File in Powershell_ise and paste the contents of the script into it > Run the script
  
Observe the accounts being created. 
  

</p>
<br />

<p>
<img src="https://i.imgur.com/AgHovq7.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Attempt to log in as one of the users created by the script.
  
Server Manager > Tools > Active Directory Users and Computers > mydomain.com > _EMPLOYEES > choose a random generated employee and double click their name > Account > copy their User logon name
  
Take note of the employee password created for all of the employees in the script. Now it is possible to login to Client-1 as employees that were created.
</p>
<br />
