
# Set Static IP

> NOTE: I found out later that the Router/Gateway IP address for VMware's NAT network needs to be `192.168.10.2` not `192.168.10.1`. VMware uses `.1` for the WAN side of the NAT and `.2` for the LAN side that our lab uses. When you see the screenshots below using `.1` take a mental note you should be using `.2` instead.

Next up we need the Domain Controller set up with a static IP address. The easiest way to do that is to go to the `Server Manager > Local Server` and click the blue link next to Ethernet.

![](<attachments/02-building-the-dc.png>)

Click on `Ethernet0 > Properties > Internet Protocol Version 4 (TCP/IPv4) > Properties` to open the setting page for the server's IP address. Here set the following configuration:

IP address: 192.168.10.10
Subnet mask: 255.255.255.0
Default Gateway: 192.168.10.2
Preferred DNS server: 127.0.0.1

![](<attachments/02-building-the-dc-1.png>)

I am setting the DNS server to the Domain Controller's loopback address which is a standard procedure to make sure the server points to itself for DNS queries during installation.

# Change Host Name

Next, we want to change the server's host name to `DC01`. That can be done back on the `Server Manger > Local Server` page by clicking the blue link next to "Computer Name".

![](<attachments/02-building-the-dc-2.png>)

After this the server will need to reboot. 

# Snapshot Checkpoint

Once the server is back up after the reboot we need to take another snapshot before we set up the server to run as our Domain Controller.

> NOTE: Create Snapshot: `Pre-AD DS`

![](<attachments/02-building-the-dc-3.png>)

# Add AD DS Role

To begin the process of configuring the DC01 server as a Domain Controller go to `Server Manager > Add Roles and Features` which will open the Add Roles and Features Wizard. This wizard walk you through the selection process to get started. Below is config used:

- Installation Type: Role-based or feature-based installation
- Destination Server: DC01
- Server Roles: 
	- Active Directory Domain Services: Checked
	- DNS Server: Checked
- Features: Keep Defaults
- Confirm installation selections: Restart the destination server automatically if required: Checked


![](<attachments/02-building-the-dc-4.png>)

Once everything is selected correctly, click Install. This will take some time to run, then we will need to do the post-deployment configuration.

# Post-deployment Configuration

Once the Add Roles and Features Wizard completes there will be a yellow flag icon on the Server Manager page.

![](<attachments/02-building-the-dc-5.png>)

Click the flag, then click the "Promote this server to a domain controller" blue link. This will open the Active Directory Domain Services Configuration Wizard which will guide us through the rest of the domain controller installation. I used the following config for the wizard:

- Deployment Configuration:
	- Add a new forest: Checked
	- Root domain name: busbeecorp.local
- Domain Controller Options:
	- Forest and Domain functional level: Windows Server 2016
	- DSRM password: Something secure
- DNS Options: Ignore warning and continue
- NetBIOS Name: BUSBEECORP
- Paths: Defaults
- Review Options: Click Next

Wait for the Prerequisite Check then click Install. You will also see a warning that the server will automatically reboot at the end of the install. It will take a few minutes for this install to finish promoting our server to domain controller so time to take a break and wait for it to finish. 

# Testing the Domain Controller

Once DC01 finishes installing the domain controller settings and reboots we will want to test that the configuration works as expected. The following PowerShell commands can be used to verify our domain controller is now live.

## NetLogons

```powershell
dcdiag /test:netlogons 
```

![](<attachments/02-building-the-dc-6.png>)

You should see successful connectivity and NetLogons and no errors reported on the partition tests.

## Replications

```powershell
dcdiag /test:replications
```

Similar to the last output, you should see no errors reported from Replications.

## nslookup

```powershell
nslookup busbeecorp.local
```

This will test the DNS capability and make sure our DC01 IP 192.168.10.10 resolves to our domain name `busbeecorp.local`. 

![](<attachments/02-building-the-dc-7.png>)

## NetLogon service

Confirm the NetLogon service is running. This should work after confirming the last NetLogon check but always good to double check.

```powershell
Get-Service Netlogon
```

![](<attachments/02-building-the-dc-8.png>)

## SYSVOL

This should list the SYSVOL and NETLOGON shares.

```powershell
net share
```

![](<attachments/02-building-the-dc-9.png>)

## Sites and Services

In `Server Manager > Tools > Active Directory Sites and Services` check that Default-First-Site-Names contains DC01.

![](<attachments/02-building-the-dc-10.png>)

# Snapshot Checkpoint

Go ahead and create a new checkpoint once we know the domain controller settings are working correctly on DC01 called `Post-DC Promo`.

# Install DHCP Role

Now that we have the domain controller role set up on DC01 we will want it to handle DHCP IP address assignment for us as well. To install the DHCP role go to `Server Manager > Add Roles and Features` and keep clicking next, accepting defaults, until you get to the Server Roles page.

![](<attachments/02-building-the-dc-11.png>)

Check the box for `DHCP Server` which will open a pop-up box. Accept the defaults here and click Add Features. Keep clicking next until you get an Install button and click that too. DHCP will take a minute to install then you should see another yellow flag on the Server Manager page that will want you to complete the DHCP configuration.

![](<attachments/02-building-the-dc-12.png>)

This will bring up another configuration wizard. 

- Authorization:
	- Use the following user's credentials: `BUSBEECORP\Administrator`

Click commit. Now we need to restart the DHCP service for the config to take effect. Go to `Server Manager > DHCP > Services > DC01` and right click to restart the service.

![](<attachments/02-building-the-dc-13.png>)

Now to set the DHCP scope go to `Server Manager > Tools > DHCP` and expand the tree `DC01 > IPv4` and right click it and select New Scope.

![](<attachments/02-building-the-dc-15.png>)

In the New Scope Wizard enter the following configuration:

- Name: `busbeecorp.local`
- Start IP Address: `192.168.10.100`
- End IP Address: `192.168.10.200`
- Subnet mask: `255.255.255.0`
- Router (Default Gateway): `192.168.10.2`
- Parent domain: `busbeecorp.local`

Accept all other defaults and click next through to the end of the wizard then click Finish. This will automatically apply our options. Next we need to click into `Server Options` then click on `Action > Configure Options`. This will bring up a menu to configure the DHCP server options.

![](<attachments/02-building-the-dc-14.png>)

Check the following boxes in this menu and configure these options:

- 003 Router
	- IP address: `192.168.10.2`
	- Click Add
- 006 DNS Servers
	- IP address: `192.168.10.10`
	- Click Add
- 015 DNS Domain Name
	- String value: `busbeecorp.local`

We can test the configuration by rebooting one of our workstation machines and confirming it got an IP address within the address pool we set.

I logged into WIN11-01 and ran IPConfig to get a new IP address.

```powershell
ipconfig /release
ipconfig /renew
```

It then showed that its using the correct gateway and gained an IP address within the address pool.

![](<attachments/02-building-the-dc-16.png>)

And we can also ping DC01 from WIN11-01:

![](<attachments/02-building-the-dc-17.png>)

# Set DHCP Reservations for SRV01 and CTX01

> NOTE: While we are logged in here with each server go ahead and update the computer name

On each of the other servers run this to get the MAC address for it:

```powershell
Get-NetAdapter | Select-Object Name, MacAddress, Status
```

![](02-building-the-dc-23.png)

![](02-building-the-dc-24.png)

Next run the following commands to set those MAC addresses for each IP reservation. If you are following along at home make sure you switch out the numbers here:

```powershell
Add-DhcpServerv4Reservation -ScopeId "192.168.10.0" -IPAddress "192.168.10.20" -ClientId "00-0C-29-C4-65-61" -Description "SRV01 Application Server"

Add-DhcpServerv4Reservation -ScopeId "192.168.10.0" -IPAddress "192.168.10.30" -ClientId "00-0C-29-00-81-C1" -Description "CTX01 Citrix Delivery Controller"
```
# Set DNS Forwarders

Next up we need to set the DNS Forwarders so DC01 can resolve external queries that are not listed manually. Just go to `Server Manager > Tools > DNS > DC01 > right-click > Properties > Forwarders tab`

Add:
- `8.8.8.8` (Google)
- `1.1.1.1` (Cloudflare)

![](02-building-the-dc-20.png)

Verify with:

```powershell 
nslookup google.com
```

You should see several Name Server IP addresses for google.com come back. If this doesn't work, the DNS is broken somewhere and needs to be looked at.

![](02-building-the-dc-21.png)

# Create Reverse Lookup Zone

```powershell
Add-DnsServerPrimaryZone -NetworkID "192.168.10.0/24" -ReplicationScope "Forest"
```

Then create DNS A records for each of the other servers:

```powershell
Add-DnsServerResourceRecordA -Name "SRV01" -ZoneName "busbeecorp.local" -IPv4Address "192.168.10.20" -CreatePtr

Add-DnsServerResourceRecordA -Name "CTX01" -ZoneName "busbeecorp.local" -IPv4Address "192.168.10.30" -CreatePtr
```

Verify with the following lines:

```powershell
nslookup 192.168.10.20
nslookup 192.168.10.30
```

# Building OU Structure

Next up I am going to build out some Organizational Units (OUs) which are a type of entity Active Directory uses to organize other pieces. Think of it as a folder to put things in. In this case, I am creating additional sub-OUs within the BusbeeCorp OU to organize our different business departments and the computers they use. This will help us to apply settings and policies to only specific departments or other groupings so we can target changes to only specific areas of the IT environment.

```powershell
$domain = "DC=busbeecorp,DC=local"
$root   = "OU=BusbeeCorp,$domain"

# Create root OU
New-ADOrganizationalUnit -Name "BusbeeCorp" -Path $domain -ProtectedFromAccidentalDeletion $true

# Top-level OUs
$topLevel = @("Users", "Computers", "Groups", "Service Accounts")
foreach ($ou in $topLevel) {
    New-ADOrganizationalUnit -Name $ou -Path $root -ProtectedFromAccidentalDeletion $true
}

# User sub-OUs
$departments = @("IT", "Finance", "HR", "Sales", "Engineering", "Marketing", "Executive")
foreach ($dept in $departments) {
    New-ADOrganizationalUnit -Name $dept -Path "OU=Users,$root" -ProtectedFromAccidentalDeletion $true
}

# Computer sub-OUs
$computerOUs = @("Workstations", "Servers")
foreach ($ou in $computerOUs) {
    New-ADOrganizationalUnit -Name $ou -Path "OU=Computers,$root" -ProtectedFromAccidentalDeletion $true
}
```

You can verify the OUs took effect by going to `Server Manager > Tools > Active Directory Users and Computers` and checking out the folders under the domain.

![](<attachments/02-building-the-dc-18.png>)


# Importing Users

Next up we need to import the user database. You can use the command below to download the file from my GitHub repo: 

```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/mbusbee505/ADCitrixLab/main/Lab-Data/busbeecorp_user_import.csv" -OutFile "C:\Users\Administrator\Downloads\busbeecorp_user_import.csv"
```

And now that we have it downloaded the following PowerShell can be used to import the .csv file to Active Directory. The script will sort each user into its respective OU based on their department listed in the csv.

```powershell

$domain = "DC=busbeecorp,DC=local"
$password = ConvertTo-SecureString "Welcome1!" -AsPlainText -Force

Import-Csv "C:\Users\Administrator\Downloads\busbeecorp_user_import.csv" | ForEach-Object {
    $ou = "OU=$($_.Department),OU=Users,OU=BusbeeCorp,$domain"
    Write-Host "Creating $($_.SamAccountName) in $ou"
    try {
        New-ADUser `
            -GivenName        $_.FirstName `
            -Surname          $_.LastName `
            -Name             "$($_.FirstName) $($_.LastName)" `
            -DisplayName      "$($_.FirstName) $($_.LastName)" `
            -SamAccountName   $_.SamAccountName `
            -UserPrincipalName "$($_.SamAccountName)@busbeecorp.local" `
            -Department       $_.Department `
            -Title            $_.JobTitle `
            -Company          "BusbeeCorp" `
            -Path             $ou `
            -AccountPassword  $password `
            -Enabled          $true `
            -ChangePasswordAtLogon $false
        Write-Host "  OK" -ForegroundColor Green
    } catch {
        Write-Host "  ERROR: $_" -ForegroundColor Red
    }
}
```

Now we can verify the user count to confirm the import was successful.

```powershell
Get-ADUser -Filter * -SearchBase "OU=Users,OU=BusbeeCorp,DC=busbeecorp,DC=local" | Measure-Object
```

![](02-building-the-dc-22.png)
# Creating Security Groups

Now we need to create security groups for our users so we can apply security permissions across each department. Once this is in place, if a user transfers to another department they will shed their old permissions and gain those of the new department. Similarly, any new employees hired and added to a department will automatically get the correct permissions.

```powershell
$root = "OU=BusbeeCorp,DC=busbeecorp,DC=local"
$departments = @("IT","Finance","HR","Sales","Engineering","Marketing","Executive")

foreach ($dept in $departments) {
    New-ADGroup -Name "GRP_$dept" `
        -GroupScope Global `
        -GroupCategory Security `
        -Path "OU=Groups,$root" `
        -Description "$dept department security group"
}
```

The following script will add users to their matching groups:

```powershell
$domain = "DC=busbeecorp,DC=local"

foreach ($dept in @("IT","Finance","HR","Sales","Engineering","Marketing","Executive")) {
    $users = Get-ADUser -Filter * -SearchBase "OU=$dept,OU=Users,OU=BusbeeCorp,$domain"
    Add-ADGroupMember -Identity "GRP_$dept" -Members $users
}
```

And now verify it worked:

```powershell
Get-ADGroup -Filter * -SearchBase "OU=Groups,OU=BusbeeCorp,DC=busbeecorp,DC=local" | ForEach-Object {
    $count = (Get-ADGroupMember $_.Name).Count
    Write-Host "$($_.Name): $count members"
}
```

![](02-building-the-dc-19.png)

# Screenshot Checkpoint

Now would be a good time to take another snapshot since we are basically done with DC01 setup. Snapshot name: `Post-Network-OU-Setup`

With that all done we are ready to move on to adding the workstations to the BusbeeCorp domain.