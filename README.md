


<p align="center">
<img src="https://i.imgur.com/R5OzmdT.png" height="55%" width="55%" alt="Microsoft Active Directory Logo"/>
</p>

<h1>Microsoft Azure - Active Directory (AD) Integration On-Premise</h1>

This demonstration outlines the implementation process of on-premises Active Directory within Azure Virtual Machines.

_<b>NOTE:</b> This demonstration uses materials created in the previous demonstration, ["Creating an Azure Account ➔ Establishing a Virtual Machine"](https://github.com/terikaj/azure-begin?tab=readme-ov-file)._

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machine Resource)
- Microsoft Remote Desktop
- Active Directory Domain Services
- PowerShell ISE

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Setup 2 Virtual Machines in Azure:
   - Domain Controller VM (Windows Server 2022) with a static private IP.
   - Client VM (Windows 10) in the same Resource Group and Vnet as the Domain Controller.
- Login to both VMs using Remote Desktop (RDP).
- Enable Inbound Rules for "Core Networking Diagnostics" in the Domain Controller's Firewall to ensure connectivity with the Client Machine.
- Install Active Directory Domain Services in the Domain Controller VM.
- Create Administrator and Standard User Accounts in Active Directory.
- Link the Client VM to the domain and login using an Administrator account.
- Set up Remote Desktop for Non-Administrative users on the Client VM.
- Create additional users and attempt to log into the Client VM as one of the newly created users.

<h2>Step Process</h2>

<h3>&#9312; Create a Domain Controller VM</h3>

- In the Search Box at the top header, type and select "Virtual machines".
  - If "Virtual machines" is already listed on the front page, then you can simply click on it, rather than manually searching.
- Click "Create", then select "Azure virtual machine".
<p align="center">
<img width="450" alt="AD 1 Step 1 Begins" src="https://github.com/TerikaJ/configure-ad/assets/136477450/b38988c8-5a75-4ceb-888d-c4c873e34fc2">
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
<img width="450" alt="AD 2" src="https://github.com/TerikaJ/configure-ad/assets/136477450/10efb989-a196-404c-9beb-7ec39ff97baa">
<img width="450" alt="AD 3" src="https://github.com/TerikaJ/configure-ad/assets/136477450/a98898b5-d29b-461b-9a71-7ef663a3cb29">
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
<img width="495" alt="AD 4 Step 2 Begins" src="https://github.com/TerikaJ/configure-ad/assets/136477450/ebec22cd-9960-4210-a267-fe4278dfe16c">
</p>

- Make sure that the "Virtual network" is set to the Vnet that the Domain Controller VM automatically created (this example uses **DC-01-vnet**).
- Select the dropdown box for Subnet and confirm the default selection (this example uses **default (10.0.0.0/24)**).
  - Sometimes you will have to manually set the Subnet, otherwise it will not let you proceed.
- Skip everything else and click "Review + create".
- If Validation passed, click "Create".
<p align="center">

<img width="592" alt="AD 5" src="https://github.com/TerikaJ/configure-ad/assets/136477450/1ac78801-b98a-4084-a780-5ee387c5b983">

<img width="587" alt="AD 6" src="https://github.com/TerikaJ/configure-ad/assets/136477450/799f13fb-54a5-46d6-89ac-86ede59f2833">
</p>
<hr>

<h3>&#9314; Assign STATIC to the Domain Controller's Private IP address</h3>

_Later in this demonstration, we'll need users to login using a domain name instead of their standard username, so we'll have to make sure the Domain Controller's NIC Private IP address doesn't get changed in the future:_
- From Azure Portal, go to DC-01 VM Overview page.
- Click on "Networking", then click on the "Network Interface" (this example uses **dc-01505**).
<p align="center">

<img width="274" alt="AD 7 Step 3 Begins" src="https://github.com/TerikaJ/configure-ad/assets/136477450/9b71150d-2019-4c20-80a5-4779f30d8049">
<img width="563" alt="AD 8" src="https://github.com/TerikaJ/configure-ad/assets/136477450/20f4477e-a1f6-4616-9dce-378575f8b3f2">
</p>

- Click on "IP configurations" on the left.
- You can see that the Private IP address is Dynamic.
  - Click "ipconfig1".
<p align="center">
<img width="563" alt="AD 8" src="https://github.com/TerikaJ/configure-ad/assets/136477450/3ec39d89-0c0e-4a06-aa96-cb59269d034c">
</p>

- Change the assignment to "Static".
- Click "Save".
<p align="center">
<img width="1094" alt="AD 9" src="https://github.com/TerikaJ/configure-ad/assets/136477450/39dd31ef-cca4-4ab5-bfe5-c9a980e8e03c">
<img width="1427" alt="AD 10" src="https://github.com/TerikaJ/configure-ad/assets/136477450/b480923f-48dd-4f4d-9b61-7ad6d78df498">
<img width="429" alt="AD 11" src="https://github.com/TerikaJ/configure-ad/assets/136477450/344a8117-3e0a-47d3-9599-50231a0ca86d">
</p>
<hr>

<h3>&#9315; Confirm Connectivity between the Client and Domain Controller</h3>

- Login to the Domain Controller and Client-01 VM utilizing Remote Desktop:
  - In Azure Portal, go to the Domain Controller and Client-01 "overview" tab (this example starts with **Client-01 VM**).
  - COPY the public IP address (located on the right side of the page).
<p align="center">
<img width="587" alt="AD 12 Step 4 Begins" src="https://github.com/TerikaJ/configure-ad/assets/136477450/00e96cbd-8749-4394-98f3-45bd28e720c9">
</p>

- If you're utilizing a PC you'll press the Windows Key/Button, type and select "Remote Desktop Connection."
- If you're utilizing MacOS you'll click the application for "Microsoft Remote Desktop" and click "add PC."
- Input the virtual machine's Public IP Address and click Connect.
- Enter the username and password, then click "continue."
<p align="center">
<img width="594" alt="AD 13" src="https://github.com/TerikaJ/configure-ad/assets/136477450/38468902-10a1-4010-86ce-7cb956f600e3">
<img width="496" alt="AD 14" src="https://github.com/TerikaJ/configure-ad/assets/136477450/d18e8169-9541-4fbb-8570-3d7715e148b6">
</p>

- A prompt will appear about the certiicate verfification; just press "YES".
<p align="center">
<img width="581" alt="AD 15 " src="https://github.com/TerikaJ/configure-ad/assets/136477450/aaa74691-0d1c-4d36-83a1-848bcfd1b2dd">
</p>

- Minimize the Virtual Machine window and login to the other VM (the Domain Controller).
<p align="center">
<img width="1810" alt="AD 16" src="https://github.com/TerikaJ/configure-ad/assets/136477450/0bc3fd35-6091-4529-9665-654f65e798d5">
<img width="1233" alt="AD 17" src="https://github.com/TerikaJ/configure-ad/assets/136477450/4a904377-8083-49f1-9cb8-35adfe0aad7e">
</p>

- On the Domain Controller VM, press the Windows Key/Button, then type and select "Windows Defender Firewall with Advanced Security".
<p align="center">
<img width="467" alt="AD 18" src="https://github.com/TerikaJ/configure-ad/assets/136477450/078f904c-b315-47d1-a3da-3f6e17e38fc5">
</p>

- Click "Inbound Rules" (on the left sidebar).
- Find the two names "Core Networking Diagnostics - ICMP Echo Request (ICMPv4-In)" (easier to sort by Protocol).
- Select both by utilizing the shift key, then click "Enable Rule" on the right sidebar (or right-click, select).
  - _Please ensure to enable ICMPv4, and NOT ICMPv6!_
<p align="center">
<img width="1572" alt="AD 19" src="https://github.com/TerikaJ/configure-ad/assets/136477450/154eb120-1422-43e8-ab0d-20615352b38c">
</p>

- Once enabled, return to the DC-01 VM Overview page in Azure.
- COPY the Private IP Address.
<p align="center">

</p>

- Now head into the Client-01 VM.
- Click the Windows key (or Start Button), type and/or select CMD or "Command Prompt" (run as Admin account if preffered).
<p align="center">
<img src="https://i.imgur.com/ca7M8oS.jpg" height="70%" width="70%" alt="Step 3-9"/>
</p>

- Inside of the Command Prompt, type "ping -t {DC-01 Private IP Address}" (this example uses IP address **10.0.0.4**)
  - _This command will send an infinite number of data packets to retrieve a responses from the DC-01 VM._
- In doing this, we can confirm the Client VM's ability to see the Domain Controller VM. Without the response the Client VM would recieve a "Request Timed Out" message.
  - _To stop the ping process press "Ctrl+C", OR close the Command Prompt._
<p align="center">
<img src="https://i.imgur.com/KKT14mt.jpg" height="70%" width="70%" alt="Step 3-10"/>
</p>
<hr>

<h3>&#9316; Install Active Directory Domain Services within Domain Controller VM</h3>

- Login to the DC-01 VM and open "Server Manager" (if not open already).
- Click "Add Roles and Features" (Option Number 2) on the main page (Server Manager).
<p align="center">
<img src="https://i.imgur.com/LKpjSjC.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- Continue to click "Next" until you reach Server Roles tab.
- Checkmark "Active Directory Domain Services", and a prompt will appear:
- Click "Add Features".
- Continue to click "Next" until you reach the Confirmation tab.
- Click "Install", and close once completed.
<p align="center">
<img width="788" alt="AD 21 Step 5 Begins" src="https://github.com/TerikaJ/configure-ad/assets/136477450/eabd2519-311c-420e-ae73-f4e201a014f6">
<img width="780" alt="AD 22" src="https://github.com/TerikaJ/configure-ad/assets/136477450/5147911d-744b-4c6c-9850-74880ad48c58">
<img width="782" alt="AD 23" src="https://github.com/TerikaJ/configure-ad/assets/136477450/810d2974-0607-4663-8c3d-1e9cfca2000c">
<img width="784" alt="AD 24" src="https://github.com/TerikaJ/configure-ad/assets/136477450/7fc5116b-97e3-4834-a81e-0ce34cccc7a2">
</p>

- Back on the Server Manager, click the flag icon with the caution symbol (located at top-right header).
- Click "Promote this server to a domain controller"
<p align="center">

<img width="1677" alt="AD 25" src="https://github.com/TerikaJ/configure-ad/assets/136477450/f5f40d00-1666-4a96-93d6-fd7adac55f6b">
</p>

- In the Deployment Configuration tab, select "Add a new forest".
- Type any domain name you wish to use (this example uses **mydomain.com**)
- Click "Next".
<p align="center">
<img width="781" alt="AD 26" src="https://github.com/TerikaJ/configure-ad/assets/136477450/af19df5e-c8ee-4d71-9c76-62ece039f97d">
</p>

- Create a password of your choice.
- Keep clicking "Next" until the "Install" option is enabled, then click "Install".
  - _Installing will result in restarting the Domain Controller VM._
<p align="center">
<img width="777" alt="AD 27" src="https://github.com/TerikaJ/configure-ad/assets/136477450/6e25e866-e88f-43af-b4d5-6a6797f9d8fa">
<img width="781" alt="AD 28" src="https://github.com/TerikaJ/configure-ad/assets/136477450/f27d85cc-d2c8-43b5-a877-132bf53d7113">
<img width="777" alt="AD 29" src="https://github.com/TerikaJ/configure-ad/assets/136477450/594dddc5-3878-40b9-98ed-d08db29ef3e9">
</p>

- Once completed, log back into the DC-01 VM.
  - _You should not be able use the previous username (Log in requires the domain name)_
- If you're using Windows, you'll select "More Choices", then click "Use a different account".
- If you're using MacOS you'll click connect just as before. 
- Add the domain name to the beginning of the original username (this example used mydomain, thus the username becomes **mydomain\dcuser**)
  - _NOTE: Do not forget to document the password you used when logging into the DC-01 VM!_
<p align="center">
<img width="437" alt="AD 31" src="https://github.com/TerikaJ/configure-ad/assets/136477450/87de5e10-5950-43f7-8499-fbb564ffc078">
</p>
<hr>

<h3>&#9317; Create an Admin Account in Active Directory</h3>

- On the Domain Controller VM, go into the Server Manager, click on "Tools" in the top-right header.
- Click "Active Directory Users and Computers".
  - _This can also be searched through clicking the Windows Key/Button._
<p align="center">

<img width="358" alt="AD 32 Step 6 Begins" src="https://github.com/TerikaJ/configure-ad/assets/136477450/574774af-a743-4459-a9af-8a2a5bd4d74d">
</p>

_For this demonstration, we will create 2 new folders within mydomain.com_
_These folders also known as "Organizational Units_
- Right-click "mydomain.com" on the left sidebar.
- Hover your mouse over "New", then click "Organizational Unit".
<p align="center">
<img width="790" alt="AD 33" src="https://github.com/TerikaJ/configure-ad/assets/136477450/d3e27a15-0a34-4724-88c1-9e8b796a1b0e">
</p>

- Name one `_EMPLOYEES` and the other `_ADMINS`.
<p align="center">
<img width="442" alt="AD 34" src="https://github.com/TerikaJ/configure-ad/assets/136477450/10f54e29-9375-4882-a346-a889011d39cc">
</p>

_Next, we'll add a new Admin user account inside the `_ADMINS` folder._
  - Right-click on `_ADMINS`(or any empty space within the folder).
  - Hover your mouse over "New", then click "User".
<p align="center">
<img width="767" alt="AD 36" src="https://github.com/TerikaJ/configure-ad/assets/136477450/4cf8d4f3-2818-4adb-af86-2ce80bf4f8dd">
</p>

- Create a first and last name, as well as a log-on name for the Admin user, then click "Next" (this example uses **Jane Doe** / **jane_admin**)
- Create a password of your choice for the account.
- Uncheck "User must change password at next logon".
- Checkmark "Password never expires".
- Click "Next" until the account is created.
<p align="center">
<img width="439" alt="AD 37" src="https://github.com/TerikaJ/configure-ad/assets/136477450/89784c41-906a-43c4-8e37-1204f298c015">
<img width="445" alt="AD 38" src="https://github.com/TerikaJ/configure-ad/assets/136477450/b88e592a-993c-4248-a9fb-35a03ec24de5">
</p>

_The user account is located inside the folder named `_ADMINS`, however, it doesn't have the admin privileges yet, they must be assigned:_
- Right-click on the new account, click "Properties".
<p align="center">
<img width="758" alt="AD 39" src="https://github.com/TerikaJ/configure-ad/assets/136477450/ce5eff90-fe69-42e3-b97b-ef9d44a8f5fe">
</p>

- Click the "Member Of" tab, then click the "Add" button.
- Type in the word "domain", then click "Check Names", allowing to view all already built-in groups.
- Select "Domain Admins", then "OK".
- Click "Apply", then "OK" again.
<p align="center">
<img width="409" alt="AD 40" src="https://github.com/TerikaJ/configure-ad/assets/136477450/0aa9fe42-8fab-454e-8d01-c34a4c0aa0b3">
<img width="466" alt="AD 41" src="https://github.com/TerikaJ/configure-ad/assets/136477450/f1f816a1-44db-42b1-aae4-aa2f440e8769">
<img width="582" alt="AD 42" src="https://github.com/TerikaJ/configure-ad/assets/136477450/810806f7-a7ce-46e4-8b2b-1ba8d2b49988">
</p>

- Once completed, log-off the Domain Controller VM and log-on the newly created admin account with the domain name (this example uses **mydomain\jane_admin**).
  - _We will utilize the jane_admin account instead of dcuser going forward._
<p align="center">
<img width="431" alt="AD 43" src="https://github.com/TerikaJ/configure-ad/assets/136477450/2e840300-8118-48ce-85d5-4f251aa9862c">
</p>
<hr>

<h3>&#9318; Join Client-01 to the domain (mydomain.com)</h3>

- From the Azure Portal, navigate to the Client-01 VM Overview page.
- Click "Network Settings", then click the "Network Interface" (this example uses **client-01591**).
<p align="center">
<img width="181" alt="AD 44 Step 7 Begins" src="https://github.com/TerikaJ/configure-ad/assets/136477450/b2abb720-e28a-4fe8-bf24-ab06fec1163d">
<img width="787" alt="AD 45" src="https://github.com/TerikaJ/configure-ad/assets/136477450/253ab65d-5051-4bb5-8686-addd0f789845">
</p>

- Navigate to "DNS servers" on the left sidebar.
- Select the "Custom" option under DNS servers.
- Input the DC's Private IP Address (this example uses **10.0.0.4**).
- Click "Save".
- Restart Client-01 VM.
  - _You can also press the Restart button in the Client-01 VM Overview page_.
  - _Logon again to Client-01 VM_.
<p align="center">
<img width="485" alt="AD 46" src="https://github.com/TerikaJ/configure-ad/assets/136477450/c882bfbc-2737-40e6-902a-8be4f69c5ffa">
<img width="528" alt="AD 47" src="https://github.com/TerikaJ/configure-ad/assets/136477450/59d30043-669b-46e0-93f4-82ff20355f39">
</p>

- On Client-01 VM, Right-click the Windows Button and select "System".
<p align="center">
<img width="330" alt="AD 48" src="https://github.com/TerikaJ/configure-ad/assets/136477450/a5e2305f-87a2-45a1-bc36-62ee376c5707">
</p>

- Click on "Rename this PC (advanced)"
<p align="center">
<img width="907" alt="AD 49" src="https://github.com/TerikaJ/configure-ad/assets/136477450/1f2e9abc-5908-4008-b4ee-a30ad1617b51">
</p>

- Within the System Properties window, click "Change".
  - _This will allow us to use the domain name connected to the Domain Controller._
- Under Member Of, select "Domain" option, then type your domain name (this example uses **mydomain.com**).
- Click "OK".
  - _A login prompt will appear._
<p align="center">
<img width="323" alt="AD 50" src="https://github.com/TerikaJ/configure-ad/assets/136477450/f9737f39-0afc-401e-91ef-97fa8be96e7b">
</p>

- Enter the Admin user's log-on credentials (this example uses **mydomain\jane_admin**).
- Click "OK".
- If done correctly, you should see a welcoming window appear to joining the domain.
- Click "OK" again and you'll be prompted to restart the computer.
<p align="center">
<img src="https://i.imgur.com/WjWrOQL.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
<img src="https://i.imgur.com/ylYxFqS.jpg" height="70%" width="70%" alt="Azure Step 5-5"/>
</p>

- Use Remote Desktop (RDP) to logon to the Client-01 VM.
- Click "Use a different account", for now we will logon using the jane_admin account.
  - _The admin account is already logged onto the DC-01 VM, but this time we are logging in through the Client-01 VM._
<p align="center">
<img width="434" alt="AD 51" src="https://github.com/TerikaJ/configure-ad/assets/136477450/955feb2b-21c4-440b-a416-5b38f5c5eb7b">
</p>
<hr>

<h3>&#9319; Setup Remote Desktop for non-administrative users on Client-01</h3>

- Right-click the Windows Button and select "System", then click "Remote Desktop".
<p align="center">
<img width="187" alt="AD 52 Step 8 Begins" src="https://github.com/TerikaJ/configure-ad/assets/136477450/6e521235-848c-41ac-8297-1b77541b7e8e">
</p>

- Click "Select users that can remotely access this PC".
<p align="center">
<img width="309" alt="AD 53" src="https://github.com/TerikaJ/configure-ad/assets/136477450/3385cba5-eee4-426a-9485-c3810a00249c">
</p>

- Next, click "Add...".
- Type "Domain_Users" into the Object Names box.
  - _You can also click "Check Names" like in Step 6._
- Once assigned, click "OK" and "OK" again.
  - _This now turns Client-01 into a normal, non-administrative user._
<p align="center">
<img width="291" alt="AD 55" src="https://github.com/TerikaJ/configure-ad/assets/136477450/fe8e4ad6-cbb6-47a8-bece-6a84edd23475">
<img width="238" alt="AD 56" src="https://github.com/TerikaJ/configure-ad/assets/136477450/c21cb7dc-5781-4bff-ade2-5f556d2c4ca1">
</p>
<hr>

<h3>&#9320; Create additional users and attempt a log-on to Client-01</h3>

- Login to DC-01 as your admin account, if not already (this example uses **jane_admin**).
- Click the Windows Key/Button and open "PowerShell_ISE" as an Administrator.
  - IMPORTANT: Run the program as an **ADMINISTRATOR**
  - _Right-click on PowerShell_ISE and select Run as administrator._
<p align="center">
<img width="775" alt="AD 58 Step 9 Begins" src="https://github.com/TerikaJ/configure-ad/assets/136477450/537d8519-adad-4f48-9bcc-0ae169937c21">
</p>

- At the top menu, click on "New Script"
- Using a premade script, copy and paste the code into the box.

<details close>
  <summary> PowerShell Script </summary>
  <p>
https://github.com/TerikaJ/name-user-generator/blob/main/name-user-generator 
    </p>
</details close>

- When ready, at the top menu, click the "Run" button (green play icon).
  - _This script will create 100 accounts using the password "Password1" (variables set at beginning of code).
These accounts will be placed at its set path: `_EMPLOYEES`, listed near the end of the code._
<p align="center">
<img width="1135" alt="AD 60" src="https://github.com/TerikaJ/configure-ad/assets/136477450/4f54437a-97c7-4c4b-9c6d-8ba1fbc63dad">
<img width="1133" alt="AD 61" src="https://github.com/TerikaJ/configure-ad/assets/136477450/69c65939-e6ed-4eac-bb65-28688449a624">
</p>

- Open "Active Directory Users and Computers" (from the Server Manager or Windows search)
- Reveal mydomain.com, then reveal `_EMPLOYEES` folder.
  - _You should see a list of randomly created user accounts in this folder._
<p align="center">


<img width="776" alt="AD 57" src="https://github.com/TerikaJ/configure-ad/assets/136477450/bf2f28c6-36ab-4083-9db4-9ca53aa2a88a">
<img width="395" alt="AD 62" src="https://github.com/TerikaJ/configure-ad/assets/136477450/5a56e516-e0c9-4a1f-b7bd-df27bf9e9784">
</p>

_Next, we're going to attempt to login one of those random users to Client-01 VM._
- Copy any randomly created user (this example will use **cudi.pom**).
- Log-off of any account currently on Client-01 VM, and attempt to login with the random user.
  - _Remember to start with the domain name before the username._
<p align="center">
<img width="536" alt="AD 63" src="https://github.com/TerikaJ/configure-ad/assets/136477450/fc9f87fb-f433-4f1f-b3dc-ebc81f00c2ad">
<img width="430" alt="AD 64" src="https://github.com/TerikaJ/configure-ad/assets/136477450/06b7ffd5-9bc6-42a7-b520-79fcdbe15995">
<img width="1212" alt="AD 65" src="https://github.com/TerikaJ/configure-ad/assets/136477450/876955a2-28d0-40d1-87a5-3fce0786048b">
</p>

_Whether it's struggling to log in, resetting a forgotten password, or safeguarding against online threats, the necessity for account assistance remains constant._
- Double-click the user's account (this example uses **cudi.pom**) to access the properties.
  - _You can also Right-Click on the account for the Context Menu._
  - With this you can unlock accounts, reset passwords, and more!
<p align="center">
<img width="403" alt="AD 66" src="https://github.com/TerikaJ/configure-ad/assets/136477450/75c88390-c00f-4885-a496-476a1b05d523">
</p>
<hr>

<h1><p align=center> DONE! Good job! </p></h1>

<h2><p align=center>The Next Demonstration:<br><a href="https://github.com/terikaj/dns-azure"> Implementing and Managing DNS Services </a></p></h2>

<p align=right> Please delete & clean up your Azure resources when finished!<br>
If you're unsure of how to do this, please click <a href="https://github.com/terikaj/azure-begin?tab=readme-ov-file">HERE</a>
