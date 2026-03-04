# Active Directory & Citrix Lab

## About This Project

This lab is an end-to-end walkthrough of building a production-style on-premises Windows environment from scratch using **Windows Server 2022**, **Active Directory**, **Citrix Virtual Apps and Desktops**, and **SQL Server**. The goal was to simulate the kind of infrastructure a sysadmin at a small company would set up and manage day to day, from the domain controller up through application virtualization.

To keep things realistic, I generated a synthetic 25-user dataset with randomized names, departments, and job titles to simulate a small company I called BusbeeCorp. That CSV, along with exported AD and Citrix configurations, all live in this repo so anyone can clone and reuse them in their own environment.

Everything runs locally on VMware Workstation Pro across five virtual machines: a domain controller, an application server, a Citrix delivery controller, and two Windows 11 workstations.

## What This Lab Covers

- Windows Server 2022 domain controller setup with AD DS, DNS, and DHCP
- Active Directory OU structure, bulk user import, and security group management
- Group Policy configuration for security baselines, drive mapping, and workstation lockdown
- File server setup with DFS namespaces and per-department NTFS permissions
- Internal PKI using AD Certificate Services and SSL cert deployment via GPO
- IIS web hosting for an internal intranet site
- SQL Server 2022 installation and database setup using the BusbeeCorp user dataset
- Citrix Virtual Apps and Desktops deployment with StoreFront and Workspace App
- Application publishing through Citrix including a simulated clinical app workflow
- AD user lifecycle operations: onboarding, password resets, account lockouts, and offboarding
- Environment monitoring with Event Viewer, Performance Monitor, and PowerShell health checks

## Skills Demonstrated

- Windows Server administration and Active Directory management
- Group Policy design and troubleshooting
- DNS, DHCP, and internal PKI configuration
- File share and DFS namespace setup with least-privilege permissions
- IIS web server administration and SSL certificate management
- SQL Server installation, configuration, and basic database administration
- Citrix Virtual Apps and Desktops deployment and session management
- PowerShell scripting for AD user and group automation
- Sysadmin operations: user lifecycle, event log triage, performance monitoring
- Lab artifact export and configuration version control

## Lab Writeup

Each section below links to a dedicated page with full steps and screenshots.

1. [**Set up the lab environment**](Writeup/01-setting-up-the-lab.md)
   1. Plan VM topology and network layout
   2. Spin up VMs in VMware Workstation Pro
   3. Obtain eval licenses
2. [**Build the domain controller**](Writeup/02-building-the-dc.md)
   1. Install AD DS and promote to domain controller
   2. Configure DHCP and DNS
   3. Build the OU structure and bulk import users
   4. Create security groups
3. [**Join workstations to the domain**](Writeup/03-joining-workstations.md)
   1. Configure DNS and join the domain
   2. Verify domain authentication
4. [**Configure Group Policy**](Writeup/04-group-policy.md)
   1. Security baseline GPO
   2. Workstation config GPO
   3. Department-specific GPOs
   4. Roaming profiles and folder redirection
5. [**Set up the application server**](Writeup/05-application-server.md)
   1. File server and DFS namespace
   2. AD Certificate Services and internal CA
   3. IIS and intranet site with SSL
   4. SQL Server and BusbeeCorpDB
6. [**Deploy Citrix Virtual Apps**](Writeup/06-citrix.md)
   1. Install Delivery Controller, StoreFront, and License Server
   2. Install VDA on the application server
   3. Create Machine Catalog and Delivery Group
   4. Publish applications
   5. Configure StoreFront and Workspace App
7. [**Sysadmin operations practice**](Writeup/07-operations.md)
   1. User onboarding and offboarding
   2. Password resets and account lockouts
   3. Event Viewer triage
   4. Performance monitoring
8. [**Conclusion and artifact export**](Writeup/08-conclusion.md)

## Repo Contents

| Folder / File | What it is |
|---|---|
| `Writeup/` | Step-by-step lab documentation with screenshots |
| `ADBackup/` | Exported AD users, groups, and GPO reports |
| `CitrixExport/` | Exported Citrix policies and site configuration |
| `SQLExport/` | BusbeeCorpDB schema and table scripts |
| `busbeecorp_user_import.csv` | Synthetic user dataset used to populate the environment |

> **Note:** All user data in this repo is synthetic. No real credentials, personal information, or production configurations are included. That said, in a real environment you would not want to publicly share GPO configurations or Citrix policy exports, as they give a detailed picture of how the environment is locked down.

## Lab Environment

| VM | Role | IP | RAM |
|---|---|---|---|
| DC01 | Domain Controller, DNS, DHCP, CA | 192.168.10.10 | 4GB |
| SRV01 | File Server, IIS, SQL Server, Citrix VDA | 192.168.10.20 | 16GB |
| CTX01 | Citrix Delivery Controller, StoreFront | 192.168.10.30 | 16GB |
| WIN11-01 | Domain-joined workstation | DHCP | 4GB |
| WIN11-02 | Domain-joined workstation | DHCP | 4GB |

All VMs run on a Host-Only VMware network (192.168.10.0/24) with NAT available for internet access during setup.

## Helpful Links

### Download and Eval

- **Windows Server 2022 eval:** https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022
- **SQL Server 2022 Developer Edition:** https://www.microsoft.com/en-us/evalcenter/evaluate-sql-server-2022
- **SQL Server Management Studio:** https://aka.ms/ssmsfullsetup
- **Citrix Virtual Apps and Desktops trial:** https://www.citrix.com/downloads/citrix-virtual-apps-and-desktops/
- **Citrix Workspace App:** https://www.citrix.com/downloads/workspace-app/