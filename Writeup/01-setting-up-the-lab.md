
# Quick Lab Overview

To get things started we need to create 5 new virtual machines: 3 Windows Servers and 2 Windows Workstations. We will need to grab the downloads from the Microsoft Evaluation Center, take a quick look at VMware's network settings to make sure we have NAT enabled, then build all of the VMs. Lastly we'll take a look at signing up for a Citrix free trial to get access to their software.
# Download ISOs

Before you can create virtual machines in VMware Workstation you need to have the ISO images for the installers to boot from. Below are the links I used to download Windows Server 2022,  Windows 11 Enterprise, and Windows SQL Server from the Windows Evaluation Center:

[Windows Server 2022](https://go.microsoft.com/fwlink/p/?LinkID=2195280&clcid=0x409&culture=en-us&country=US)

[Windows 11 Enterprise](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise) [Alternate Link](https://go.microsoft.com/fwlink/?linkid=2334167&clcid=0x409&culture=en-us&country=us)

[Windows SQL Server](https://www.microsoft.com/en-us/evalcenter/evaluate-sql-server-2022) [Alternate Link](https://go.microsoft.com/fwlink/?linkid=2215202&clcid=0x409&culture=en-us&country=us)

Microsoft may ask you to fill out a form to access the download link. I believe they do this to prevent automated bot downloads because they don't seem to care when you submit generic form data. 

It will take some time to download given the large file size so give it a few minutes and come back when it's done.
# Overview of VMware Network Topology

Now I debated how I wanted to have the network setup for this lab. I was torn between isolating the lab entirely to achieve a similar setup to a real environment using VMware's Host-Only setting, or using NAT and letting my machines have access to the internet through sharing my computer's IP address. 

I ultimately chose to use NAT so as to not overcomplicate the project. If I wanted to practice good security I would isolate them in a Host-Only network, and possibly allow a few of them restricted internet access via a second VMware network interface that used NAT and had strict firewall rules enabled.

![](<attachments/01-setting-up-the-lab.png>)

Notice that I have configured `VMnet8`, currently set to NAT external connection type, to use the Subnet Address: 192.168.10.0 and Subnet Mask: 255.255.255.0 which gives us a dedicated space for the lab machines and plenty of room to work with address-wise. 

I also unchecked "Use local DHCP service to distribute IP addresses to VMs". This tells VMware that I don't want it to provide NAT IPs to my machines, since I want this to be handled by our domain controller.

With that decided I moved on to building the virtual machines.
# Create and Configure VMs

Basically for this section I am going to create 3 Windows Servers and 2 Windows Workstations. The initial setup process will be just about the same for each respectively, except for some slight differences with regards to resource allocation, naming, and other specific identity items. Below is a summary of the installations.

![](<attachments/01-setting-up-the-lab-1.png>)

It's worth noting that in a real production environment you would want to make extremely secure passwords for each Administrator account on the server machines and not use a local admin account on the workstations, instead having the user login to the computer for the first time using their work login. You could also consider a more advanced setup that uses Intune and Autopilot to automatically run a Out-Of-The-Box configuration to join the device to the domain and connect to the 
## Snapshots

Before beginning I wanted to take a second to talk about virtual machine snapshots. Shapshots are a way users of virtual machines can save the state of the machine as a backup in the event of error or corruption. They can also be useful in testing when you want to roll back to a previous configuration when something doesn't work. 



For this lab I will be taking snapshots often with my machines, particularly in a few places such as when I have completed the initial OS installation, when I have correctly configured systems such as the domain controller or applications server, or when I have made a dramatic change to the system or environment such as joining the workstation devices to the domain. I will make an attempt to point out when I make snapshots for any of the machines.
## DC01 (Domain Controller)

- vCPU: 2
- RAM: 4GB
- Disk: 60GB (thin provisioned)
- Network: VMnet8 (NAT)
- OS: Windows Server 2022 Standard Evaluation Desktop Experience
- Full System Update
- Snapshot #1 Name: Fresh OS Install
## SRV01 (Application Server)

- vCPU: 4
- RAM: 16GB
- Disk: 100GB (thin provisioned)
- Network: VMnet8 (NAT)
- OS: Windows Server 2022 Standard Evaluation Desktop Experience
- Full System Update
- Snapshot #1 Name: Fresh OS Install
## CTX01 (Citrix Delivery Controller)

- vCPU: 4
- RAM: 16GB
- Disk: 80GB (thin provisioned)
- Network: VMnet8 (NAT)
- OS: Windows Server 2022 Standard Evaluation Desktop Experience
- Full System Update
- Snapshot #1 Name: Fresh OS Install

## WIN11-01 and WIN11-02

- vCPU: 2
- RAM: 4GB
- Disk: 60GB (thin provisioned)
- Network: VMnet8 (NAT)
- OS: Windows 11 Enterprise
- Full System Update
- Snapshot #1 Name: Fresh OS Install

# Citrix 