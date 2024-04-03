<p align="center">
<img src="https://i.imgur.com/R5OzmdT.png" height="55%" width="55%" alt="Microsoft Active Directory Logo"/>
</p>

<h1>Microsoft Azure - Active Directory (AD)</h1>

This demonstration outlines the implementation process of on-premises Active Directory within Azure Virtual Machines.

_<b>NOTE:</b> This demonstration uses materials created in the previous demonstration, ["Creating an Azure Account ➔ Establishing a Virtual Machine"](https://github.com/terikaj/azure-begin?tab=readme-ov-file)._

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machine Resource)
- Microsoft Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Setup 2 Virtual Machines within Azure:
  - Domain Controller VM (Windows Server 2022) -- set Private IP Address to STATIC
  - Client VM (Windows 10) -- using same Resource Group and Vnet as the Domain Controller
- Login to both VMs using Remote Desktop (RDP).
- Enable Inbound Rules for "Core Networking Diagnostics" within the Domain Controller's Firewall.
- This is to ensure connectivity between the Client and Domain Controller.
- Install Active Directory Domain Services within the Domain Controller Virtual Machine. 
- Create an Administrator and Standard User Account in Active Directory.
- Link Client Virtual Machine to a domain, then login using original Administrator account.
- Setup the Remote Desktop for Non-Administrative users on the Client Virtual Machine.
- Create additional users and attempt to log into Client VM as one of those newly created users

<h2>Step Process</h2>

<h3>&#9312; Create a Domain Controller VM</h3>

- In the Search Box at the top header, type and select "Virtual machines".
  - If "Virtual machines" is already listed on the front page, then you can simply click on it, rather than manually searching.
- Click "Create", then select "Azure virtual machine".
<p align="center">
<img src="https://i.imgur.com/tiC5aA4.jpg" height="80%" width="80%" alt="Step 1-1"/>
</p>

- Name your Virtual Machine anyway you want (this example uses **DC-01**).
  - Resource Group is automatically given a name when naming the Virtual Machine, but you can change it if you wish (this example uses **DC-01_group**).
- Change the Region that best suites your location (this example uses **(US) West US 3**).
- Change the Image to a Windows Server, as this will become our Domain Controller VM (this example uses **Windows Server 2022 Datacenter: Azure Edition**).
- Make sure the Size is adequate enough to run this server (this example uses **Standard_E2s_v3 - 2 vcpus, 16 GiB memory**).
- Create a username and password of your choice (this example uses **dcuser**).
- Skip everything else and click "Review + create".
  - IF there is a Licensing Checkbox at the end, make sure that is CHECKED!
- If Validation passed, click "Create".
<p align="center">
<img src="https://i.imgur.com/RVHx4nb.png" height="70%" width="70%" alt="Step 1-2"/>
</p>
<hr>

<h3>&#9313; Create a Client VM</h3>

- Follow the same steps as before for creating a virtual machine.
  - However, the Resource Group should be assigned as the same for the Domain Controller VM (this example uses **DC-01_group**).
  - Use a different virtual machine name (this example uses **Client-01**).
  - Change the Administrator Account credentials to differentiate the two VMs (this example uses **clientuser**).
- Change the Image to a Windows OS (this example uses **Windows 10 Pro, version 22H2 - x64 Gen2**).
- Once done, click "Next" until you reach "Networking" (OR you can simply click the Networking tab at the top).
<p align="center">
<img src="https://i.imgur.com/SDbD5nV.jpg" height="70%" width="70%" alt="Step 1-3"/>
</p>

- Make sure that the "Virtual network" is set to the Vnet that the Domain Controller VM automatically created (this example uses **DC-01-vnet**).
- Select the dropdown box for Subnet and confirm the default selection (this example uses **default (10.0.0.0/24)**).
  - Sometimes you will have to manually set the Subnet, otherwise it will not let you proceed.
- Skip everything else and click "Review + create".
- If Validation passed, click "Create".
<p align="center">
<img src="https://i.imgur.com/SiHJ4N0.jpg" height="70%" width="70%" alt="Step 1-4"/>
</p>
<hr>

<h3>&#9314; Assign Domain Controller's Private IP address to STATIC</h3>

_Later in this demonstration, we'll need users to login using a domain name instead of their standard username, so we'll have to make sure the Domain Controller's NIC Private IP address doesn't get changed in the future:_
- From Azure Portal, go to DC-01 VM Overview page.
- Click on "Networking", then click on the "Network Interface" (this example uses **dc-01667**).
<p align="center">
<img src="https://i.imgur.com/OV6YK9L.jpg" height="70%" width="70%" alt="Step 2-1"/>
</p>

- Click on "IP configurations" on the left.
- You can see that the Private IP address is Dynamic.
  - Click on "ipconfig1".
<p align="center">
<img src="https://i.imgur.com/AzqdoWb.jpg" height="70%" width="70%" alt="Step 2-2"/>
</p>

- Change the Assignment at the bottom to "Static".
- Click "Save".
<p align="center">
<img src="https://i.imgur.com/S6HX1sJ.jpg" height="70%" width="70%" alt="Step 2-3"/>
</p>
<hr>

<h3>&#9315; Ensure Connectivity between the Client and Domain Controller</h3>

- Login to both the Domain Controller and Client-01 VMs with Remote Desktop:
  - In Azure Portal, go to any VM Overview page (this example starts with **Client-01 VM**).
  - COPY its public IP address (located on the right side).
<p align="center">
<img src="https://i.imgur.com/l1jop0r.jpg" height="90%" width="90%" alt="Step 3-1"/>
</p>

- Press the Windows Key/Button, type and select "Remote Desktop Connection".
- Input the virtual machine's Public IP Address and click Connect.
- Enter the username and password, then click OK.
<p align="center">
<img src="https://i.imgur.com/hcBKjWu.jpg" height="90%" width="90%" alt="Step 3-2"/>
</p>

- A prompt will appear about the identity cannot be verified; just press "YES".
<p align="center">
<img src="https://i.imgur.com/3YxlS2G.jpg" height="70%" width="70%" alt="Step 3-3"/>
</p>

- Minimize the Virtual Machine window and login to the other VM (the Domain Controller).
<p align="center">
<img src="https://i.imgur.com/PpLmQO7.jpg" height="90%" width="90%" alt="Step 3-4"/>
<img src="https://i.imgur.com/2KVQ6D1.png" height="64%" width="64%" alt="Step 3-5"/>
</p>

- On the Domain Controller VM, press the Windows Key/Button, then type and select "Windows Defender Firewall with Advanced Security".
<p align="center">
<img src="https://i.imgur.com/WulKDTC.png" height="64%" width="64%" alt="Step 3-6"/>
</p>

- Click "Inbound Rules" (on the left sidebar).
- Find the two names "Core Networking Diagnostics - ICMP Echo Request (ICMPv4-In)" (easier to sort by Protocol).
- Select them both, then click "Enable Rule" on the right sidebar (or right-click, select).
  - _Make sure that you enable ICMPv4, and NOT ICMPv6!_
<p align="center">
<img src="https://i.imgur.com/M1HM7m4.jpg" height="100%" width="100%" alt="Step 3-7"/>
</p>

- Once enabled, return to the DC-01 VM Overview page in Azure.
- COPY the Private IP Address.
<p align="center">
<img src="https://i.imgur.com/mAvwb9i.jpg" height="100%" width="100%" alt="Step 3-8"/>
</p>

- With that copied, go into the Client-01 VM.
- Press the Windows key (or Start Button), the type and select CMD or "Command Prompt" (you can run as Admin if desired).
<p align="center">
<img src="https://i.imgur.com/ca7M8oS.jpg" height="70%" width="70%" alt="Step 3-9"/>
</p>

- Inside the Command Prompt, type "ping -t {DC-01 Private IP Address}" (this example uses IP address **10.0.0.4**)
  - _This will infinitely sent data packets for response to the DC-01 VM._
- This confirms if the Client VM can see the Domain Controller VM successfully, otherwise you'll recieve a "Request Timed Out" message.
  - _You can either press "Ctrl+C" to stop the ping process, OR you can simply close the Command Prompt._
<p align="center">
<img src="https://i.imgur.com/KKT14mt.jpg" height="70%" width="70%" alt="Step 3-10"/>
</p>
<hr>

<h3>&#9316; Install Active Directory Domain Services within Domain Controller VM</h3>

- Login to DC-01 VM and open "Server Manager" (if not open already).
- Click on "Add Roles and Features" (Number 2) on the front page.
<p align="center">
<img src="https://i.imgur.com/LKpjSjC.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- Keep clicking "Next" until you reach Server Roles tab.
- Checkmark "Active Directory Domain Services", and prompt will appear:
- Click "Add Features".
- Keep clicking "Next" until the Confirmation tab.
- Click "Install" then close once completed.
<p align="center">
<img src="https://i.imgur.com/nRniK70.jpg" height="100%" width="100%" alt="Azure Step 5-5"/>
</p>

- Back on the Server Manager, click on the flag icon with a caution symbol on it (located at top-right header).
- Click "Promote this server to a domain controller"
<p align="center">
<img src="https://i.imgur.com/5SEF3r7.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- In the Deployment Configuration tab, select "Add a new forest".
- Type any domain name you wish to use (this example uses **mydomain.com**)
- Click "Next".
<p align="center">
<img src="https://i.imgur.com/KqLJ1zR.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- Create a password of your choice.
- Keep clicking "Next" until the "Install" option is enabled, then click "Install".
  - _Installing will result in restarting the Domain Controller VM._
<p align="center">
<img src="https://i.imgur.com/sOfwFw2.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
<img src="https://i.imgur.com/GGJhNCF.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
<img src="https://i.imgur.com/8eabtaB.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- Once completed, log back into the DC-01 VM.
  - _However, you should not be able use the same username as before, now that it requires the domain name._
- Select "More Choices", then click "Use a different account".
- Change the username to add the domain name at the beginning of the original username (this example used mydomain.com, thus the username becomes **mydomain.com\dcuser**)
  - _NOTE: Don't forget what password to use when logging into the DC-01 VM!_
<p align="center">
<img src="https://i.imgur.com/ruqHto2.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>
<hr>

<h3>&#9317; Create an Admin Account in Active Directory</h3>

- On the Domain Controller VM, in Server Manager, click on "Tools" on the top-right header.
- Click "Active Directory Users and Computers".
  - _This can also be searched from the Windows Key/Button._
<p align="center">
<img src="https://i.imgur.com/qvdckjx.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

_For this demonstration, we will create 2 new folders within mydomain.com (also known as "Organizational Unit")_
- Right-click "mydomain.com" on the left sidebar.
- Hover "New", then click "Organizational Unit".
<p align="center">
<img src="https://i.imgur.com/4Jm0gW5.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- Name one `_EMPLOYEES` and the other `_ADMINS`.
<p align="center">
<img src="https://i.imgur.com/YNpRK8u.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

_Next, we'll add a new Admin user account inside the `_ADMINS` folder._
  - Right-click on `_ADMINS`(or any empty space within the folder).
  - Hover "New", then click "User".
<p align="center">
<img src="https://i.imgur.com/x9FzS23.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- Create a first and last name, as well as a logon name for this Admin user, then click "Next" (this example uses **Jane Doe** / **jane_admin**)
- Create a password of your choice for that account.
- Uncheck "User must change password at next logon".
- Checkmark "Password never expires".
- Click "Next" until the account is created.
<p align="center">
<img src="https://i.imgur.com/vTN4ckZ.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

_The user account is only inside a folder named `_ADMINS`, but doesn't mean it has the privileges as one, so:_
- Right-click on the new account, then click "Properties".
<p align="center">
<img src="https://i.imgur.com/6dV5m0S.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- Click on the "Member Of" tab, the click the "Add" button.
- Type in the word "domain", then click "Check Names", allowing to view all already built-in groups.
- Select "Domain Admins", then "OK".
- Click "Apply", then "OK" again.
<p align="center">
<img src="https://i.imgur.com/ZzPNPHn.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- Once completed, logoff of Domain Controller VM and logon to the newly created admin account with the domain name (this example uses **mydomain.com\jane_admin**).
  - _We use jane_admin account from now on instead of dcuser._
<p align="center">
<img src="https://i.imgur.com/g9bZXb4.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>
<hr>

<h3>&#9318; Join Client-01 to the domain (mydomain.com)</h3>

- From Azure Portal, go to Client-01 VM Overview page.
- Click on "Networking", then click on the "Network Interface" (this example uses **client-01857**).
<p align="center">
<img src="https://i.imgur.com/eHiv28J.jpg" height="70%" width="70%" alt="Step 2-1"/>
</p>

- Go to "DNS servers" on the left sidebar.
- Select "Custom" option under DNS servers.
- Input the DC's Private IP Address (this example uses **10.0.0.4**).
- Click "Save".
- Restart Client-01 VM.
  - _You can also press the Restart button in the Client-01 VM Overview page_.
  - _Logon again to Client-01 VM_.
<p align="center">
<img src="https://i.imgur.com/LaLsiSk.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
<img src="https://i.imgur.com/Sg9pvYY.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- On Client-01 VM, Right-click the Windows Button and select "System".
<p align="center">
<img src="https://i.imgur.com/JBMFeOM.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- Click on "Rename this PC (advanced)"
<p align="center">
<img src="https://i.imgur.com/BnEqRHf.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- Within the System Properties window, click "Change".
  - _This will allow us to use the domain name connected to the Domain Controller._
- Under Member Of, select "Domain" option, then type your domain name (this example uses **mydomain.com**).
- Then click "OK".
  - _A login prompt will appear._
<p align="center">
<img src="https://i.imgur.com/ERxuEiv.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- Enter the Admin user's logon credentials (this example uses **mydomain.com\jane_admin**).
- Click "OK".
- If done correctly, you should see a welcoming window appear to joining the domain.
- Click "OK" again and prompts you to require a restart.
<p align="center">
<img src="https://i.imgur.com/RRCV4F5.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
<img src="https://i.imgur.com/WjWrOQL.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
<img src="https://i.imgur.com/ylYxFqS.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- Use Remote Desktop (RDP) to logon to the Client-01 VM.
- Click "Use a different account", for now we will logon using the jane_admin account.
  - _The admin account is already logged onto the DC-01 VM, but this time we are logging in through the Client-01 VM._
<p align="center">
<img src="https://i.imgur.com/zM9oOdR.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>
<hr>

<h3>&#9319; Setup Remote Desktop for non-administrative users on Client-01</h3>

- Right-click the Windows Button and select "System", then click "Remote Desktop".
<p align="center">
<img src="https://i.imgur.com/6ddD3c5.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- At the bottom, click "Select users that can remotely access this PC".
<p align="center">
<img src="https://i.imgur.com/C6Bb4qG.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- Next, click "Add...".
- Type in "Domain_Users" into the Object names box.
  - _You can also click Check Names like in Step 6._
- Once assigned, click "OK" and "OK" again.
  - _This now makes Client-01 as a normal, non-administrative user._
<p align="center">
<img src="https://i.imgur.com/PP0Xr1p.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>
<hr>

<h3>&#9320; Create additional users and attempt to log into Client-01 with one of them</h3>

- Login to DC-01 as your admin account, if not already (this example uses **jane_admin**).
- Press the Windows Key/Button and open "PowerShell_ISE" as an Administrator.
  - _Right-click on PowerShell_ISE and select Run as administrator._
<p align="center">
<img src="https://i.imgur.com/js2KU37.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- At the top menu, click on "New Script"
- Using a premade script, copy and paste the code into the box.

<details close>
  <summary> PowerShell Script </summary>
  <p>
 # ----- Edit these Variables for your own Use Case ----- #
$PASSWORD_FOR_USERS   = "Password1"
$NUMBER_OF_ACCOUNTS_TO_CREATE = 100
# ------------------------------------------------------ #

Function generate-random-name() {
    $consonants = @('b','c','d','f','g','h','j','k','l','m','n','p','q','r','s','t','v','w','x','z')
    $vowels = @('a','e','i','o','u','y')
    $nameLength = Get-Random -Minimum 3 -Maximum 7
    $count = 0
    $name = ""

    while ($count -lt $nameLength) {
        if ($($count % 2) -eq 0) {
            $name += $consonants[$(Get-Random -Minimum 0 -Maximum $($consonants.Count - 1))]
        }
        else {
            $name += $vowels[$(Get-Random -Minimum 0 -Maximum $($vowels.Count - 1))]
        }
        $count++
    }

    return $name

}

$count = 1
while ($count -lt $NUMBER_OF_ACCOUNTS_TO_CREATE) {
    $fisrtName = generate-random-name
    $lastName = generate-random-name
    $username = $fisrtName + '.' + $lastName
    $password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force

    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName $firstName `
               -Surname $lastName `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_EMPLOYEES,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
    $count++
}
    </p>
</details close>

- When ready, at the top menu, click the "Run" button (green play button icon).
  - _This script will create 100 accounts using the password "Password1" (variables set at beginning of code).
These accounts will be placed at its set path: `_EMPLOYEES`, listed near the end of the code._
<p align="center">
<img src="https://i.imgur.com/2w34lrh.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
<img src="https://i.imgur.com/sIr018L.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- Open "Active Directory Users and Computers" (from the Server Manager or Windows search)
- Reveal mydomain.com, then reveal `_EMPLOYEES` folder.
  - _You should see all of the randomly created user accounts in this folder._
<p align="center">
<img src="https://i.imgur.com/W8M9O4l.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

_Next, we're going to attempt to login one of those random users to Client-01 VM._
- Copy any randomly created user (this example will use **did.cuta**).
- Logoff of any account currently on Client-01 VM, and attempt to login with the random user.
  - _Remember to start with the domain name before the username._
<p align="center">
<img src="https://i.imgur.com/GHvYS94.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
<img src="https://i.imgur.com/oXORyTs.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

_Whether it's failing at logging into accounts, resetting a password, or protecting against dangerous actors, there will always be a need for assistance to access one's account._
- Double-click the user's account (this example uses **did.cuta**) to access the properties.
  - _You can also Right-Click on the account for the Context Menu._
  - With this you can unlock accounts, reset passwords, and more!
<p align="center">
<img src="https://i.imgur.com/VOUcyyi.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>
<hr>

<h1><p align=center> DONE! Good job! </p></h1>

<h2><p align=center>Next Demonstration:<br><a href="https://github.com/terikaj/azure-network-protocols"> Network Security Groups (NSGs) and Inspecting Network Protocols</a></p></h2>

<p align=right> Please delete & clean up your Azure resources when finished!<br>
If you're unsure of how to do this, please click <a href="https://github.com/terikaj/azure-begin?tab=readme-ov-file">HERE</a>
