<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>Setting Up On-Premises Active Directory within Azure</h1>
This guide explains the process of deploying on-premises Active Directory using Azure Virtual Machines.<br />

<h2>Utilized Environments and Technologies</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell
- [Script](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1) (For generating names and creating users)

<h2>Operating System Platforms</h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>Overview of Deployment and Configuration Stages</h2>

- Prepare Azure Resources
- Establish Connection Between Client and Domain Controller
- Set up Active Directory
- Generate User Accounts within Active Directory
- Join Client-1 to the Domain
- Generate More Users and Test Login

<h2>Prepare Azure Resources</h2>

<p>
<img width="760" alt="screenshot2" src="https://github.com/user-attachments/assets/ea441d40-0b9f-4690-b763-a604dfdc8649" />
</p>
<p>
1. Establish a Resource Group: Begin by setting up a new Resource Group for organizing your Azure components. Assign a suitable name, for example, "AD-Lab".

2. Provision the Domain Controller VM (DC-1):
   - Assign a name to the VM, like "DC-1", during setup.
   - Select "Windows Server 2022" for the VM's operating system.
   - Verify that the correct Resource Group is chosen and that the Virtual Network is also created.
   - Once the VM deployment is complete, find the Network Interface Card (NIC) linked to the Domain Controller.
   - Configure the NIC's Private IP address to be static. This ensures stable connectivity for the Domain Controller.

3. Provision the Client VM (Client-1):
   - Set up a new virtual machine named "Client-1".
   - Choose "Windows 10" as its operating system.
   - Ensure it is placed within the same Resource Group and Virtual Network as the Domain Controller.

</p>
<br />

<h2>Establish Connection Between Client and Domain Controller</h2>

<p>
<img src="https://i.imgur.com/SEdlbha.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
1. Connect to Client-1 via Remote Desktop: Use Remote Desktop to log into the Client-1 VM. You'll need the correct login details and permissions.

2. Test Connectivity with Ping: On Client-1, open Command Prompt or PowerShell and run the command `ping <DC-1_Private_IP> -t` (substituting `<DC-1_Private_IP>` with DC-1's actual private IP). This sends continuous pings to the Domain Controller.

3. Note Initial Failures: You'll likely see the ping requests time out at first, showing no connection between the VMs.

4. Adjust Firewall on DC-1 (Allow ICMPv4):
   - Connect to the DC-1 VM using Remote Desktop and appropriate credentials.
   - Open the Windows Firewall configuration.
   - Find the firewall rule related to ICMPv4 (Internet Control Message Protocol version 4).
   - Activate this rule to permit ICMPv4 traffic.

5. Allow Time for Changes: Give it a short time for the firewall modification to become active.
</p>
<br />

  <h2>Set up Active Directory</h2>

<p>
<img src="https://i.imgur.com/8l3r5un.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
1. Access DC-1: Log into the DC-1 VM using Remote Desktop or another method with the correct credentials.

2. Add the Active Directory Domain Services Role:
   - Launch the Server Manager tool.
   - Select "Add roles and features" to start the wizard.
   - Proceed through the wizard steps, making appropriate selections until you get to the "Server Roles" page.
   - Find and tick the box for "Active Directory Domain Services".
   - Continue with the installation, keeping default settings unless specific changes are needed.
   - Finalize the role installation.

3. Configure a New Forest: After AD DS installation, configure a new forest named "mydomain.com":
   - Go back to the Server Manager.
   - In the notification area, click "Promote this server to a domain controller".
   - Within the Active Directory Domain Services Configuration Wizard, choose "Add a new forest" and type "mydomain.com" for the root domain name.
   - Set the functional levels for the forest and domain according to your needs.
   - Define a password for Directory Services Restore Mode (DSRM).
   - Continue through the wizard, reviewing and confirming your choices.
   - When the setup finishes, AD DS will be operational, and the "mydomain.com" forest created.

4. Reboot and Re-login:
   - Reboot the DC-1 VM for the changes to take effect.
   - Once restarted, log into DC-1 with the username "mydomain.com\labuser" and its password.

</p>
<br />
<h2>Generate User Accounts within Active Directory</h2>

<p>
<img src="https://i.imgur.com/gD6pTwE.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
1. Access DC-1 as Domain User: Log into the DC-1 VM using the "mydomain.com\labuser" account (the username specified earlier).

2. Launch ADUC: Start the "Active Directory Users and Computers" console (often found via Administrative Tools or Start menu search).

3. Establish Organizational Units (OUs):
   - Right-click "mydomain.com", choose "New" > "Organizational Unit".
   - Name the first OU "_EMPLOYEES" and click "OK".
   - Follow the same procedure to create another OU named "_ADMINS".

4. Add a New User Account (Employee):
   - Right-click the "_EMPLOYEES" OU, select "New" > "User".
   - Fill in the required information. For this example, use "Jane Doe" (Full Name) and "jane_admin" (username).
   - Assign a password and adjust any other necessary settings.
   - Finalize the creation of the user.

5. Grant Administrative Privileges to "jane_admin":
   - Find the "Domain Admins" Security Group (usually under the "Builtin" container).
   - Right-click "Domain Admins" and select "Properties".
   - Go to the "Members" tab in the Properties dialog.
   - Click "Add" to include a new member.
   - Type "jane_admin" into the object name field and click "OK".
   - Confirm "jane_admin" appears in the member list, then click "OK".

6. Re-login as the New Admin User:
   - Sign out of the DC-1 session for "mydomain.com\labuser".
   - Sign back in using "mydomain.com\jane_admin" and the password set for that account.
</p>
<br />

<h2>Join Client-1 to the Domain</h2>
<p>
<img width="781" alt="screenshot3 - Copy" src="https://github.com/user-attachments/assets/2076d5ff-c70f-49cf-86b9-5fc14c498e16" />
</p>
<p>
1. Go to the Microsoft Azure Portal

2. Find Client-1 and Adjust its DNS Configuration:
   - Go to the Virtual Machines area in the Azure Portal.
   - Find and choose the Client-1 VM.
   - Within Client-1's settings, go to the Networking options.
   - Locate the DNS server settings and change them to use the Private IP address of DC-1.
   - Apply the modifications.

3. Reboot Client-1: Restart the Client-1 virtual machine so it uses the updated DNS configuration.

4. Reconnect via Remote Desktop: Use Remote Desktop to access both Client-1 and the Domain Controller again.

5. Check for Client-1 in Active Directory:
   - On the Domain Controller, open the ADUC console and look in the "Computers" container at the domain root.
   - Ensure the Client-1 computer object is present in the "Computers" container, confirming its successful domain join.

6. Organize Client-1 into a Dedicated OU:
   - In ADUC, right-click the domain name.
   - Choose "New" > "Organizational Unit", name it "_CLIENTS", and create it.
   - Find the Client-1 object in the "Computers" container.
   - Right-click the Client-1 object and select "Move".
   - Select the "_CLIENTS" OU as the destination.
   - Confirm the action to move the object.
</p>
<br />
<h2>Join Client-1 to the Domain</h2>

<p>
<img src="https://i.imgur.com/jxZ5eKR.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
1. Access Client-1 as Administrator:
   - Log into the Client-1 VM using the "jane_admin" account ("mydomain.com/jane_admin") via Remote Desktop or another method.

2. Enable Remote Desktop Access for Domain Users:
   - Go to "System" properties on Client-1 (e.g., right-click "This PC" -> "Properties").
   - Select the "Remote settings" link (usually on the left).
   - On the "Remote" tab, within the "Remote Desktop" section, click "Select users".
   - Click "Add" to specify users or groups allowed remote access.
   - Enter "domain users", then click "Check Names" to verify.
   - Confirm by clicking "OK" to grant Remote Desktop permission to the "domain users" group.

3. Test Access with a Standard User:
   - Sign out of the Client-1 session as "jane_admin" (mydomain.com/jane_admin).
   - Sign back into Client-1 using the login details of a standard, non-admin domain user.
</p>
<br />
<h2>Generate More Users and Test Login</h2>
<p>
<img src="https://i.imgur.com/2DRjuD8.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>

1. Access DC-1 as Administrator:

2. Launch PowerShell ISE with Admin Rights:

3. Prepare the User Creation Script:
   - In the PowerShell ISE, go to "File" -> "New" to open a blank script file.
   - Copy and paste the provided [script](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1)'s content into this new file.

4. Adjust Script Parameters:
   - Find the section in the script that specifies the quantity of accounts to generate.
   - Change this number to 100 (or your preferred quantity).

5. Execute the Script and Monitor:
   - Watch the PowerShell console output as the script runs and creates the user accounts. Note any messages.

6. Identify Login Credentials for one User:
   - After the script finishes, examine the console output.
   - Find the username and password generated for one specific account.
   - Record these details for the next step.

7. Test Login with a Script-Generated User:
   - Start a Remote Desktop connection to the Client-1 VM.
   - Enter the username and password you recorded from the script's output to log in as that specific user.
</p>
