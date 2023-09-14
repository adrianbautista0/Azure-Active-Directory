# Azure-Active-Directory
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
- Windows 10 (22H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Set up a virtual machine running Windows 10 in Azure
- Set up a virtual machine running Windows Server in Azure
- Set up Active Directory
- Configure Domain Services
- Join a vritual machine to your domain
- Create Organizational units and users
- setup remote desktop for your virtual machine
- Create adittional users

<h2>Deployment and Configuration Steps</h2>


### Setting Up resources in azure 
- To get started, go to your computer and head over to portal.azure.com 
- From there, create a resource group

- While in Azure, create a virtual machine  
- Name it “DC-1” for Domain Controller
- Place it in the resource group that was created earlier
- Set the region of this virtual machine in the same region as the resource group
- For the image, set it the latest version of Windows Server 
- For size, you want a minimum of 2 virtual CPUs and 16 GB of memory
- Set a username and password
- Make sure to check both boxes under licensing at the bottom of the basic page and click create

- When finished, go back to virtual machines page and create another virtual machine 
- Place it the same resource group that you created earlier 
- Name it “Client-1”
- Set the region of this virtual machine in the same region as the resource group
- Select Windows 10 Pro, version 22H2 for the image
- For size, you want a minimum of 2 virtual CPUs and 16 GB of memory
- Set a username and password 
- Make sure to check both boxes under licensing at the bottom of the basic page
- Go back to the virtual machine page 
- Select DC-1 > Networking > Click on Network Interface > IP configurations > Set Private IP Address to static

### Ensuring connectivity between the client and Domain Controller
- On Azure, find the public IP address of Client-1 and copy it
- Go to start on your computer (If on Windows) and run Microsoft Remote Desktop (if on mac download it from the app store)
- Login in with the credentials you set for Client-1 then connect

- On your computer, go back to the Azure portal, find the private IP address of DC-1 and copy it
- Then go back your virtual machine
- Go to start in the virtual machine and run command prompt
- While on command prompt, type in “ping -t “ with a space followed by the private IP address of DC-1
- You will see “Request timed out” because DC-1’s firewall is blocking ICMPV4 traffic

- With DC-1’s public IP address copied, run another instance of Remote Desktop Connection and paste the IP address, then login with the credentials
- Once logged > go to start  > search for WF.MSC 
- Select inbound rules > expand > sort by protocol 
- Find ICMPV4 and enable both versions of Core Networking Diagnostics -ICMP Echo Request (ICMPv4 in)

- Go back to the instance of Client-1 and observe the changes in the command prompt
- Close command prompt when finished 

### Setting up Active Directory 
Go back to the instance of DC-1
Once on there, go to start and find server manager 
Click on Add roles and features > next(x3)
Add Active Directory Domain Services and keep pressing next and install

Once Installed, you can close it 
On the upper right hand corner, you will notice a yellow triangle with a exclamation point on it, click on it and select Promote this server to domain controller 

When on deployment configuration, select add new forest and add a domain name 
To keep it generic, I would call it “mydomain.com”

Then hit next and set a password for DRSM
Keep clicking next and install
Once it has finished, you will be logged out as it will be setting up as a Domain Controller 
Log back into DC-1 in the context of the domain you just created
For example: 
Username: mydomain.com\labuser 
Password: Password1


### Create an Admin and Normal User account in Active Directory 
Once logged on the domain controller, go to server manager > click tools > open Active Directory Users and Computers 
Right click “mydomain.com” or the domain name you chose > select new > select Organization Units 
To differentiate it from the rest, call it “_EMPLOYEES” 
Create another organization unit called “_ADMINS” 

You’re going to create a different admin account and login with it after
Right click on admins > select new > select user
Fill out the boxes > click next > set a password > make sure every box is unchecked
Once created, go to the admins folder, look for the user you just created > right click on said user > properties
Go to member of and add to admins by typing in “Domain Admins”
Check names then click ok and apply
Log off the computer

### Joining Client 1 to your domain
Go back to Azure portal, go to virtual machines, find DC-1’s private IP address in the networking tab and copy it 
Within the Azure portal, go back to Client-1 > networking > click network interface > DNS Servers > Custom > copy DC-1’s private IP Address then save

Once finished, restart Client-1 in the Azure portal 
Log back into Client-1 as the original local admin 
Right click start > click on system > In about section, click rename this pc > click change > click domain domain > type in the domain name you chose 
A login prompt will appear and login within the context of the domain, proceed by logging in with the admin account you created earlier on DC-1

You will be greeted saying you’ve joined the domain, it will ask you to restart the computer, restart the computer for the changes to be in full effect

### Setup Remote desktop for non-administrative users on Client-1
 Log back into Client-1 in the context of your domain (log back into the same admin account you just used before you were logged off) 
Once logged in, right click start  > click system > remote desktop > select users that can remotely access this pc 
Click Add > Type in “Domain Users” > check names and click ok
 
This will allow anyone that are non-administrative users of the domain to log into Client-1 and have access to that machine 

### Create a bunch of Additional Users and attempt to login into Client 1 with one of the users 
Login into DC-1 as the same administrative user account you logged into Client-1 with
Go to start > search for Windows Powershell ISE > right click and run as administrator
Create a new file > go to the new file > paste this script > run script (this will create new users in the _EMPLOYEES Organizational Unit for the domain)
Go to start and search for Active Directory Users and Computers, find _EMPLOYEES, right click and refresh and you will see new users 

You can select one of the users, write down their login information
Go back into Client-1 and log in with that user in the context of the domain 
You will notice you will successfully login into Client-1 as a domain user!

