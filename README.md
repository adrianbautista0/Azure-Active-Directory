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
1. To get started, go to your computer and head over to portal.azure.com 
2. From there, create a resource group

3. While in Azure, create a virtual machine  
4. Name it “DC-1” for Domain Controller
5. Place it in the resource group that was created earlier
6. Set the region of this virtual machine in the same region as the resource group
7. For the image, set it the latest version of Windows Server 
8. For size, you want a minimum of 2 virtual CPUs and 16 GB of memory
9. Set a username and password
10. Make sure to check both boxes under licensing at the bottom of the basic page and click create

11. When finished, go back to virtual machines page and create another virtual machine 
12. Place it the same resource group that you created earlier 
13. Name it “Client-1”
14. Set the region of this virtual machine in the same region as the resource group
15. Select Windows 10 Pro, version 22H2 for the image
16. For size, you want a minimum of 2 virtual CPUs and 16 GB of memory
17. Set a username and password 
18. Make sure to check both boxes under licensing at the bottom of the basic page
19. Go back to the virtual machine page 
20. Select DC-1 > Networking > Click on Network Interface > IP configurations > Set Private IP Address to static

### Ensuring connectivity between the client and Domain Controller
1. On Azure, find the public IP address of Client-1 and copy it
2. Go to start on your computer (If on Windows) and run Microsoft Remote Desktop (if on mac download it from the app store)
3. Login in with the credentials you set for Client-1 then connect

4. On your computer, go back to the Azure portal, find the private IP address of DC-1 and copy it
5. Then go back your virtual machine
6. Go to start in the virtual machine and run command prompt
7. While on command prompt, type in “ping -t “ with a space followed by the private IP address of DC-1
8. You will see “Request timed out” because DC-1’s firewall is blocking ICMPV4 traffic

9. With DC-1’s public IP address copied, run another instance of Remote Desktop Connection and paste the IP address, then login with the credentials
10. Once logged > go to start  > search for WF.MSC 
11. Select inbound rules > expand > sort by protocol 
12. Find ICMPV4 and enable both versions of Core Networking Diagnostics -ICMP Echo Request (ICMPv4 in)

13. Go back to the instance of Client-1 and observe the changes in the command prompt
14. Close command prompt when finished 

### Setting up Active Directory 
1. Go back to the instance of DC-1
2. Once on there, go to start and find server manager 
3. Click on Add roles and features > next(x3)
4. Add Active Directory Domain Services and keep pressing next and install

5. Once Installed, you can close it 
6. On the upper right hand corner, you will notice a yellow triangle with a exclamation point on it, click on it and select Promote this server to domain controller 

7. When on deployment configuration, select add new forest and add a domain name 
8. To keep it generic, I would call it “mydomain.com”

9. Then hit next and set a password for DRSM
10. Keep clicking next and install
11. Once it has finished, you will be logged out as it will be setting up as a Domain Controller 
12. Log back into DC-1 in the context of the domain you just created
- For example:
  Username: mydomain.com\labuser 
    Password: Password1


### Create an Admin and Normal User account in Active Directory 
1. Once logged on the domain controller, go to server manager > click tools > open Active Directory Users and Computers 
2. Right click “mydomain.com” or the domain name you chose > select new > select Organization Units 
3. To differentiate it from the rest, call it “_EMPLOYEES” 
4. Create another organization unit called “_ADMINS” 

5. You’re going to create a different admin account and login with it after
6. Right click on admins > select new > select user
7. Fill out the boxes > click next > set a password > make sure every box is unchecked
8. Once created, go to the admins folder, look for the user you just created > right click on said user > properties
9. Go to member of and add to admins by typing in “Domain Admins”
10. Check names then click ok and apply
11. Log off the computer

### Joining Client 1 to your domain
1. Go back to Azure portal, go to virtual machines, find DC-1’s private IP address in the networking tab and copy it 
2. Within the Azure portal, go back to Client-1 > networking > click network interface > DNS Servers > Custom > copy DC-1’s private IP Address then save

3. Once finished, restart Client-1 in the Azure portal 
4. Log back into Client-1 as the original local admin 
5. Right click start > click on system > In about section, click rename this pc > click change > click domain domain > type in the domain name you chose 
6. A login prompt will appear and login within the context of the domain, proceed by logging in with the admin account you created earlier on DC-1

7. You will be greeted saying you’ve joined the domain, it will ask you to restart the computer, restart the computer for the changes to be in full effect

### Setup Remote desktop for non-administrative users on Client-1
1. Log back into Client-1 in the context of your domain (log back into the same admin account you just used before you were logged off) 
2. Once logged in, right click start  > click system > remote desktop > select users that can remotely access this pc 
3. Click Add > Type in “Domain Users” > check names and click ok
 
- This will allow anyone that are non-administrative users of the domain to log into Client-1 and have access to that machine 

### Create a bunch of Additional Users and attempt to login into Client 1 with one of the users 
1. Login into DC-1 as the same administrative user account you logged into Client-1 with
2. Go to start > search for Windows Powershell ISE > right click and run as administrator
3. Create a new file > go to the new file > paste this script > run script (this will create new users in the _EMPLOYEES Organizational Unit for the domain)
4. Go to start and search for Active Directory Users and Computers, find _EMPLOYEES, right click and refresh and you will see new users 

5. Select one of the users, write down their login information
6. Go back into Client-1 and log in with that user in the context of the domain 
- You will notice you will successfully login into Client-1 as a domain user!

