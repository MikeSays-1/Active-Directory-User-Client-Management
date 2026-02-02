<p align="center">
<img width="600" alt="Microsoft Active Directory Logo" src="images/active-directory-seeklogo.png" />

</p>

<h1>Active Directory User & Client Management Lab (Azure)</h1>
This project builds on an existing Active Directory environment deployed in Microsoft Azure and focuses on core domain administration tasks, including user and group management, Organizational Unit (OU) structure, client domain joining, Remote Desktop access configuration, and automated user provisioning using PowerShell. The project demonstrates practical identity and access management workflows within a cloud-hosted Windows domain using industry-standard tools and configurations.<br />


<!--
<h2>Video Demonstration</h2>

- ### [YouTube: How to Deploy on-premises Active Directory within Azure Compute](https://www.youtube.com)
-->

<h2>Environments and Technologies Used</h2>

<img src="https://skillicons.dev/icons?i=azure,windows,powershell" />

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (22H2)

<h2>High-Level Deployment and Configuration Steps</h2>

<img src="images/Active Directory Configuration.gif" width="400" alt="Domain Diagram">


With the domain environment already established, the project continues by expanding domain management and user access. After initially accessing the client and domain controller with local credentials, Organizational Units are created for Employees and Administrators, followed by the creation of a dedicated Domain Admin account. The client virtual machine is then joined to the domain and organized within a Clients OU. Remote Desktop access is configured to allow standard domain users to sign in, and PowerShell automation is used to bulk-create user accounts within the Employees OU to simulate real-world provisioning tasks.

**Key Actions**
- Create Organizational Units for Employees and Admins
- Create Domain Admin account
- Join client VM to the domain
- Configure Remote Desktop access for domain users
- Automate bulk user creation with PowerShell

> [!IMPORTANT]
> Each step includes written instructions followed by a corresponding screenshot.
<br>Expand the **See screenshots** section to view the images.


> [!NOTE] 
>This project builds upon a prior lab where the Azure environment, virtual machines, Active Directory Domain Services, and DNS configuration were initially deployed. <br /> **[Part 1: On-premises Active Directory Deployed in the Cloud (Azure)](https://github.com/MikeSays-1/azure-config-and-ad-installation)**


<h2>Deployment and Configuration</h2> 

<h3>1. CREATE A DOMAIN ADMIN USER. </h3>
<p>On your local machine, Remote Deskop into "vm-dc-1" using the public IP Address, and with your original admin user created in Azure but with domain credentials, mydomain.com\&ltyourcreatedcredentials&gt and password. In the Windows search bar, find "Active Directory Users and Computers" (ADUC), alternatively you can go Start Menu > Windows Administrative Tools > Active Directory Users and Computers.
Expand the “mydomain.com” domain. Right-click the domain and select New > Organizational Unit. </p>

<details><summary>See screenshots</summary>
<img src="images/Step 1a.PNG" width="60%" >
</details> 

<p>In the Name field, type “_EMPLOYEES”. An underscore is used to differentiate this Organizational Unit from the default Windows containers within the domain tree. Repeat steps to create a new OU for "_ADMINS". Right-click our Admins folder and select New > User. Create user Jane Doe, with a username "jane_admin" and select Next, in the Password screen, unmark the checkbox for "User must change password at next logon", and mark "Password never expires check box", and choose a password. </p>

> [!NOTE] 
> For the purposes of this lab, a simplified password policy is used to streamline setup and testing; however, these settings would not be recommended in a production environment.

<details><summary>See screenshots</summary>
<img src="images/Step 1b.PNG" width="60%" >
</details> 
<p>Even though Jane Doe was created in our _ADMINS folder, it does not automatically designate her as admin, we will need to add her to the built-in Domain Admins Security Group. Right-click user Jane Doe, find tab "Member of" and select "Add..", in the Object names field, type "Domain Admins" and select Check Names, once underlined, we can select OK. Keep the Remote Connection on.

<details><summary>See screenshots</summary>
<img src="images/Step 1c.PNG" width="60%" >
</details> 

<p>Our Jane Doe is now an Admin, log out of the "vm-dc-1" and log back in with Jane Doe's username, using domain credentials, mydomain.com\jane_admin and password.  We will continue using Jane Doe from now on.

<h3>2. JOIN VM CLIENT 1 TO DOMAIN </h3>
<p>On your local machine, create another instance of Remote Desktop Connection, and Remote Deskop into "vm-client-1" using the public IP Address, and with your original admin user created in Azure but with domain credentials, mydomain.com\&ltyourcreatedcredentials&gt and password. Right-click the Windows start menu and select "System". Find and select "Rename this PC (advanced) on the right hand side > Select "Change" beside "To rename this computer or change its domain...". Under "Member of" select Domain, in the text field, type our created domain, "mydomain.com"</p>

<details><summary>See screenshots</summary>
<img src="images/Step 2a.PNG" width="60%" >
</details> 

> [!NOTE] 
>Because we configured client’s DNS to point to "vm-dc-1"," it can resolve the domain name and locate the Active Directory domain controller, which allows the domain login screen to appear.

<p>Admin privileges are required to change this setting. Login using Jane Doe's username with domain credentials. Since Jane has admin rights, it will allow us to join the domain. You'll receive a pop-up that welcomes us to the domain. Click through the prompts, and close the system properties window, and select "Restart Now". And let Client 1 restart.</p>

<p>Back in our Domain Controller VM, reopen the Active Directory Users and Computers settings. Expand the domain tree and verify "vm-client-1" shows up in the Computers folder.</p>

<details><summary>See screenshots</summary>
<img src="images/Step 2b.PNG" width="60%" >
</details> 

<p> Right-click on our domain, and create a new OU "_CLIENTS". Drag "vm-client-1" from the Computers folder into the _CLIENTS folder. </p>

<details><summary>See screenshots</summary>
<img src="images/Step 2c.PNG" width="60%" >
</details> 

> [!NOTE] 
>This OU structure illustrates best practices for directory organization by grouping administrators, employees, and client systems into separate management containers.

<h3>3. ENABLE REMOTE DESKTOP FOR USERS</h3>
<p>Currently, users can sign in only when physically present at the computer; enabling Remote Desktop allows them to securely log in from another workstation and access their environment, which is a common practice in many organizations for flexibility and support.</p>

<p>Login to VM Client 1, with Jane Doe using domain credentials. Right-click the Windows start menu, and select "System". Find "Remote Desktop" on the right hand side. Under User accounts, select "Select users that can remotely..." > Select "Add..", in the Object names field, type "Domain Users" and select Check Names, once underlined, we can select OK.

<details><summary>See screenshots</summary>
<img src="images/Step 3a.PNG" width="60%" >
</details>
<p>Non-admin users can now log in. We will create those accounts in the next step. For now, sign out of "vm-client-1".

> [!NOTE] 
>“Domain Users” is a built-in security group in Active Directory that automatically includes all standard user accounts created within the domain. 

> [!NOTE] 
>Normally, Remote Desktop access is enabled through Group Policy, which allows administrators to configure this setting across multiple client machines at once.

<h3>4. CREATE EMPLOYEE USERS WITH AUTOMATION</h3>
<p>We will automate the creation of multiple user accounts to simulate a real-world scenario where an IT administrator receives a list of new employees from HR. The script will generate these accounts in bulk while automatically placing them into the _EMPLOYEES OU.</p>

<p>Copy the folder "ad-script" from repository and place it on the desktop of "vm-dc-1". Open PowerShell ISE as Administrator, and open the AD PS Script file.</p>

<details><summary>See screenshots</summary>
<img src="images/Step 4a.PNG" width="60%" >
</details> 

Note the line ```$PASSWORD_FOR_USERS   = "Password1"```
this indicates all accounts will be created with the same password. Run the script and observe the accounts being generated. When the script finishes, open Active Directory Users and Computers and verify the new users appear in the _EMPLOYEES OU.

<details><summary>See screenshots</summary>
<img src="images/Step 4b.PNG" width="60%" >
</details>

<p>Lastly, let's test the new accounts, choose any username and note the password from the script. Remote Desktop back into "vm-client-1" using the newly created credentials and confirm the account works successfully!</p>
