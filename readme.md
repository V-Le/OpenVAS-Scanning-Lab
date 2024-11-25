# Vulnerability Scanning & Remediation With OpenVAS Using VirtualBox

In this homelab, I set up a virtualized environment using Oracle VirtualBox, deploying an OpenVAS Vulnerability Management Scanner VM and a vulnerable Windows 10 VM. I simulated a security testing scenario by intentionally installing outdated software on the Windows 10 VM and performing both unauthenticated and credentialed vulnerability scans with OpenVAS. I then analyzed the scan results, remediated the identified vulnerabilities, and verified improvements through subsequent rescans.

Remember that creating vulnerabilities involves making the system insecure, so it's essential to isolate your lab environment from production networks. This can be done using virtual machines and a separate, isolated network. Also, ensure that you understand the implications of your actions and the importance of ethical conduct in cybersecurity.

This lab provided hands-on experience in vulnerability scanning, system hardening, and patch management.

- Set up a virtualized homelab environment using Oracle VirtualBox, deploying an OpenVAS Vulnerability Management Scanner VM and a vulnerable Windows 10 VM.  
- Developed a vulnerable Windows 10 VM by intentionally installing outdated software and disabling security controls to simulate common real-world vulnerabilities.  
- Performed unauthenticated and credentialed vulnerability scans using OpenVAS, comparing results between both methods to identify security gaps.  
- Analyzed and documented scan results, highlighting the differences in findings between unauthenticated and credentialed scans, and assessed the impact of vulnerabilities.  
- Remediated identified vulnerabilities through system patching, configuration changes, and security updates, then verified successful remediation by rescanning with OpenVAS.  
- Created a comprehensive list of vulnerabilities and remediation steps, simulating realistic vulnerability remediation scenarios for improving system security.

**Skills:**
Vulnerability Scanning · OpenVAS Configuration · Unauthenticated and Authenticated Scanning · Vulnerability Remediation · System Hardening · Network Security · Windows Firewall · Security Documentation · Cyber Risk Management · Penetration Testing · Virtualization

---

## Prerequisites 
- Downloaded and installed VirtualBox [Download](https://www.virtualbox.org/wiki/Downloads)
- Greenbone OpenVAS OVA downloaded [Download](https://www.greenbone.net/en/greenbone-free/)
- Windows 10 ISO downloaded [Download](https://www.microsoft.com/en-us/software-download/windows10) 
## Setting up Greenbone OpenVAS VM
- Start VirtualBox
- Select *File > Import Appliance*
	- Click on ![](images/Icon-SelectFile.png) and navigate to and select the downloaded Greenbone OpenVAS OVA file
	- Click Finish
- Click Settings
	- *Network > Adapter 1 > Enable Network Adapter > Attached to: Bridged Adapter > Promiscuous Mode: Allow All*
		- This allows the VM to communicate with other VMs that have Bridge Adapter enabled (same subnet)
		- ![|450](images/Network_adapter1.png)
	- *Network > Adapter 2 > Enable Network Adapter > Attached to: NAT*
		- This allows the VM to communicate to the internet through the host machine's IP
		- ![|450](images/Network_adapter2.png)
	- Click *OK* to apply settings
- Click Start and log in
	- Username: admin
	- Password: admin
	- When prompted to complete setup, select *Yes*
	- When Prompted to create a web admin, select *Yes*
	- When prompted to create new admin, create an Account name and Password
	- Select *OK*
	- When prompted to Upload Subscription Key Now, select *Skip > OK*
- Once in the main Greenbone OS Administration menu, we need to update the feed.
	- ![](images/Greenbone-OpenVAS-Menu.png)
- Select *Maintenance > Feed > Update*
	- The feed update will take up to 30-60 minutes. You can check the progress by going to *About* in the main menu. If it is updating still, it will say "System Status: The system operation 'Update Feed' is running currently"
	- Once it is completed, it will say "System Status: No system operation is running currently."
- You can shutdown OpenVAS VM by selecting *Maintenance > Power > Shutdown*
- Note: Now would be a good time to take a snapshot of the base VM to revert back to if you accidentally misconfigure something.
	- On the Oracle VirtualBox Manager, select the VM to take a snapshot of.
	- Click on ![|40](images/VirtualBox_Take.png)
		- Snapshot Name: Baseline
		- Snapshot Description: Fresh installation
## Setting up Windows 10 VM
- Start VirtualBox
- Select *Machine > New*
	- *Name and Operating System*
		- Name: Windows10-VM
		- Under ISO Image, click on ![](images/ISO_image_dropdown.png) and navigate to and select the downloaded Windows 10 ISO file
			- ![](images/Windows10_Name-and-OS.png)
	- Click on *Unattended Install*
		- Type in a Username and Password
	- Click on *Hardware*
		- Base Memory: 4096 MB or more
		- Processors: 2 CPU or more
	- Click *Hard Disk*
		- Hard Disk Size: at least 10 GB, no more than 15 GB
	- Click Finish
- Note: Windows 10 VM will automatically start. Let it start and if it gives you an error "*Windows cannot read the \<ProductKey> setting from the unattend answer file.*", follow the instructions below. Otherwise, skip these steps.
	- Click *OK* to close dialogue
	- On the Virtual Machine menu bar, click *File > Close... > Power off the machine > OK*
	- On Oracle VirtualBox Manager, select the Windows10-VM, then click Settings
	- Under *System > Boot Order*, uncheck Floppy
	- Under *Storage > Devices*, select the item under *Controller: Floppy*, then click Remove Attachment ![](images/Remove_Attachment.png)
	- Click *OK*
- On Oracle VirtualBox Manager, click Settings
	- *Network > Adapter 1 > Enable Network Adapter > Attached to: Bridged Adapter > Promiscuous Mode: Allow All*
		- This allows the VM to communicate with other VMs that have Bridge Adapter enabled (same subnet)
		- ![|450](images/Network_adapter1.png)
	- *Network > Adapter 2 > Enable Network Adapter > Attached to: NAT*
		- This allows the VM to communicate to the internet through the host machine's IP
		- ![|450](images/Network_adapter2.png)
	- Click *OK* to apply settings
- Click Start to power up Windows 10 VM
- Set up Windows 10 installation
	- Click *Next > Install Now > I don't have a product key*
	- For the operating system selection, select *Windows 10 Pro*, and click *Next*
	- Check *I accept the license terms > Next*
	- Click *Custom: Install Windows only (advanced)*
	- Click *Drive 0 Unallocated Space > New > Apply > OK*
	- Click *Drive 0 Partition 2 > Next*
	- Let Windows 10 install on the virtual machine
- After Windows 10 is installed, go through basic setup
	- Click *Yes > Yes > Skip second keyboard layout > Set up for personal use > Next > Offline account > Limited experience*
	- Go through the next few screens of setting up your Username, Password, and security questions.
	- Click *Not now > Accept > Skip > Not now*
- After you get past all the menus, you can shutdown the Windows10 VM
	- Click on *Start > Power > Shutdown*
- Note: Now would be a good time to take a snapshot of the base VM to revert back to if you accidentally misconfigure something.
	- On the Oracle VirtualBox Manager, select the VM to take a snapshot of.
	- Click on ![|40](images/VirtualBox_Take.png)
		- Snapshot Name: Baseline
		- Snapshot Description: Fresh installation
## Enabling vulnerabilities on Windows 10 VM
- Boot up the Windows 10 VM
- Once logged in, disable the Windows Firewall
	- Click *Start* and type *wf.msc* to open the Widows Firewall
	- In the *Windows Defender Firewall with Advanced Security on Local Computer* panel under *Overview*, Click on *Windows Defender Firewall Properties*
	- Click *Domain Profile > Firewall state: Off*
	- Click *Private Profile > Firewall state: Off*
	- Click *Public Profile > Firewall state: Off*
	- Click *Ok*
- Install Old Software
	- Install an Old Version of Firefox: Firefox Setup 97.0b5.exe
	- Install an Old Version of VLC Player: vlc-1.1.7-win32.exe
	- Install an Old Version of Adobe Reader: 10.0_AdbeRdr1000_en_US_1_.exe
- Restart the Windows 10 VM
-  Right Click on the Windows icon![](images/WindowsTaskbar.png) on the bottom left, then click on *Run*
	- In the text field, type *CMD* and press enter to opening the command line
	- Type "*ipconfig /all*"
	- If you had set your networking to bridge mode. Under Ethernet adapter Ethernet 2, write down the *IPv4 Address*
	- ![](images/Ethernet%20adapter.png)
## Configuring unauthenticated scan in OpenVAS
- Boot up the OpenVAS VM and log in
- Click on *Assets* > *Hosts* > *New Host*
	- Add the Client VM PRIVATE IP Address
- Create a New Target from the Host, name it “Azure Vulnerable VMs”.
	- Take note of the credentials. We will add SMB credentials later.
- Create a new Task
	- Name & Comment: “Scan - Azure Vulnerable VMs”
	- Scan Targets → “Azure Vulnerable VMs”
	- Save the Task
## Running unauthenticated scan against Windows 10 VM
- “Start” the “Scan - Azure Vulnerable VMs” Task
	- Take note of the Status:
## Observing unauthenticated scan results
- Once the scan is finished, click the date under “Last Report” to see the results
	- Take note of Tabs, specifically the “Results” tab. Even though we installed a super old version of Firefox, note that it does not show up here.
	- Note that this is because we aren’t running a credentialed scan so the scanner could not discover it. We will configure credential scans next
## Configuring authenticated scan in OpenVAS with credentials
- Go to Configuration → Credentials → New Credential
	- Name / Comment → “Azure VM Credentials”
	- Allow Insecure Use: Yes
	- Username: azureuser
	- Password: Cyberlab123!
	- Save
- Go to Configuration → Targets → CLONE the Target we made before
	- NEW Name / Comment: “Azure Vulnerable VMs - Credentialed Scan”
	- Ensure the Private IP is still accurate
	- Credentials → SMB → Select the Credentials we just made: Azure VM Credentials
	- Save

## Reconfiguring Windows 10 VM to allow authenticated scan
- Disable User Account Control
- Enable Remote Registry
- Set Registry Key
	- Launch Registry Editor (regedit.exe) in “Run as administrator” mode and grant Admin Approval, if requested
	- Navigate to HKEY_LOCAL_MACHINE hive
	- Open SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System key
	- Create a new DWORD (32-bit) value with the following properties:  Name: LocalAccountTokenFilterPolicy  Value: 1
	- Close Registry Editor  
- Restart the VM
## Running authenticated scan against Windows 10 VM
- Within Greenbone / OpenVAS, go to Scans → Tasks
- CLONE the “Scan - Azure Vulnerable VMs” Task, then Edit it:
	- Name / Comment → “Scan - Azure Vulnerable VMs - Credentialed”
	- Targets: Azure Vulnerable VMs - Credentialed Scan
	- Save
- Click the Play button to launch the new Credentialed Scan, wait for it to finish
	- It will take longer than the last one. Wait for it to finish


## Observing authenticated scan results
- After the credentialed scan finishes, you can immediately see the difference in findings:
- Check SMB Login under “Results”
- Further inspect the individual vulnerabilities and see all the Criticals from the out-of-date FireFox
## Remediating software vulnerabilities
- Log back into your Win10-Vulnerable VM
- Uninstall Adobe Reader, VLC Player, and Firefox
- Restart the VM
- Re-initiate the “Scan - Azure Vulnerable VMs - Credentialed” scan and observe the results to verify remediations.

## Further Recommendations