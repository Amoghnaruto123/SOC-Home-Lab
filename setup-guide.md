# **SOC Home Lab — Setup Guide** 

## **1. Ubuntu Server VM Setup** 

### **1.1 Virtual Machine Specifications** 

The Wazuh server was deployed on Ubuntu Server 24.04.5 LTS using Oracle VirtualBox. 

|**Setting**|**Configuration**|
|---|---|
|VM Name|SOC-Wazuh|
|Operating System|Ubuntu Server 24.04.5 LTS|
|RAM|6 GB (6144 MB)|
|CPUs|4|
|Storage|70 GB (dynamically allocated)|
|Network Adapter 1|NAT|
|Network Adapter 2|Host-only Adapter|
|Username|socadmin|
|Hostname|soc-wazuh|



### **1.2 Network Configuration** 

The VM was configured with two network adapters: 

- **NAT:** Provides internet access for package installation and updates. 

- **Host-only:** Allows communication between the Wazuh server and the lab machines. 

The host-only network uses the 192.168.56.0/24 subnet. 

|**Interface**|**Network**|**IP Address**|
|---|---|---|
|enp0s3|NAT|10.0.2.15|
|enp0s8|Host-only|192.168.56.1<br>0|



### **1.3 Checking Network Interfaces** 

The following command was used to inspect the network interfaces and their assigned IP addresses. 

ip addr 

The NAT interface, enp0s3 , was assigned 10.0.2.15 . The host-only interface, enp0s8 , was configured with the static IP address 192.168.56.10 . 

### **1.4 Configuring the Static IP Address** 

The network configuration was managed using Netplan. 

The configuration file used was: 

sudo nano /etc/netplan/01-soc-lab.yaml 

Its configuration: 

network: version: 2 ethernets: enp0s3: dhcp4: true 

enp0s8: 

addresses: 

- 192.168.56.10/24 

The NAT interface uses DHCP, while the host-only interface has a static IP address. 

To apply the network configuration: 

sudo netplan apply 

To verify the assigned addresses: 

ip addr 

The host-only interface should show 192.168.56.10/24 . 

### **1.5 Verifying Internet Connectivity** 

Internet connectivity was checked using ping . 

ping -c 4 8.8.8.8 

This checks connectivity to Google's public DNS IP address. 

DNS resolution was also tested: 

ping -c 4 google.com 

This verifies that the system can resolve a domain name and reach it. 

### **1.6 Updating Ubuntu and Installing Prerequisites** 

Before installing Wazuh, the package lists were updated and the required utilities were installed. 

sudo apt update && sudo apt install -y curl gnupg apt-transport-https 

- apt update refreshes the package lists. 

- curl downloads files from URLs. 

- gnupg provides tools for handling cryptographic keys. 

- apt-transport-https enables HTTPS support for APT repositories. 

### **1.7 Installing Wazuh** 

Wazuh version 4.14 was installed as an all-in-one deployment, including the Wazuh manager, indexer, and dashboard. 

The official Wazuh installation script was downloaded using: 

curl -sO <u>https://packages.wazuh.com/4.14/wazuh-install.sh</u> 

The downloaded Bash script was executed with the all-in-one installation option: 

sudo bash ./wazuh-install.sh -a 

The -a option performs an all-in-one installation of the Wazuh components. 

After installation, the Wazuh dashboard was accessed at: 

<u>https://192.168.56.10</u> 

The dashboard credentials were generated during installation. They should be stored securely and must not be committed to the GitHub repository. 

### **1.8 Retrieving Wazuh Dashboard Credentials** 

During the Wazuh installation, a password archive named wazuh-install-files.tar was generated. It contains the credentials required to access the Wazuh dashboard. 

The password file is located inside the archive at: 

wazuh-install-files/wazuh-passwords.txt 

To display the contents of the password file without extracting the entire archive, the following command was used: 

sudo tar -xOf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt 

#### **Command explanation:** 

- sudo : Runs the command with administrative privileges. 

- tar : Utility used to read and extract archive contents. 

- -x : Extracts files from the archive. 

- -O : Prints the extracted contents to the terminal instead of saving them to a file. 

- -f : Specifies the archive file. 

- wazuh-install-files.tar : The generated password archive. 

- wazuh-install-files/wazuh-passwords.txt : The password file inside the archive. 

The output contains the generated credentials, including the admin account and its password. 

**Security note:** Never publish the password file or its contents in a public GitHub repository. Keep the credentials private and redact them from screenshots. 

### **1.9 Accessing the Wazuh Dashboard** 

After retrieving the generated credentials, the Wazuh dashboard was accessed using a web browser. 

#### **Dashboard URL:** 

<u>https://192.168.56.10</u> 

The following credentials were used: 

**Field Value** Usernam admin e Generated password from the installation Password archive 

The browser may display a certificate warning because the dashboard uses HTTPS with a locally generated certificate. The dashboard was successfully accessed after proceeding to the site. 

Initially, the dashboard displayed the message: 

This instance has no agents registered 

This was expected because the Windows endpoint had not yet been connected to the Wazuh manager. 

## **2. Windows Endpoint Setup** 

### **2.1 Windows Endpoint Configuration** 

The physical Windows PC was configured as the endpoint monitored by the Wazuh server. The Wazuh agent was installed on the PC, and Sysmon was used to collect detailed Windows process and security events. 

|**Component**|**Configuration**|
|---|---|
|Endpoint|Physical Windows PC|
|Hostname|DESKTOP-AF7NJD2|
|Wazuh Agent Name|SOC-Windows|
|Wazuh Agent ID|001|
|Wazuh Manager IP|192.168.56.10|
|Windows Host-only<br>IP|192.168.56.1|
|Monitoring|Wazuh Agent and<br>Sysmon|



### **2.2 Network Connectivity** 

The Windows PC communicated with the Wazuh server through the VirtualBox Hostonly network. 

The network configuration used in the lab was: 

|**Device**|**IP Address**|
|---|---|
|Ubuntu Wazuh|192.168.56.1|
|Server|0|
|Windows PC|192.168.56.1|
|Kali Linux|192.168.56.2<br>0|



The Windows PC was configured to communicate with the Wazuh manager at 192.168.56.10 . 

### **2.3 Wazuh Agent Installation** 

The Wazuh agent was installed on the Windows PC and configured to communicate with the Ubuntu Wazuh manager. 

The agent configuration file is located at: 

C:\Program Files (x86)\ossec-agent\ossec.conf 

The agent was registered with the manager using the name SOC-Windows and assigned agent ID 001 . 

The manager address configured for the agent was: 

192.168.56.10 

### **2.4 Configuring Windows Event Log Collection** 

The Wazuh agent was configured to collect events from the following Windows event channels: 

- Application 

- Security 

- System 

- Microsoft-Windows-Sysmon/Operational 

These logs provide information about Windows activity, authentication events, system events, and process creation. 

The agent configuration file was edited at: 

C:\Program Files (x86)\ossec-agent\ossec.conf 

The Security event collection configuration was also adjusted to ensure that Event ID 5157 was not excluded. This was required to collect Windows Filtering Platform blocked connection events for the network detection exercise. 

### **2.5 Installing and Configuring Sysmon** 

Sysmon was installed on the Windows PC to provide detailed system activity logs. 

The Wazuh agent was configured to collect events from the Sysmon Operational channel: 

Microsoft-Windows-Sysmon/Operational 

Sysmon Event ID 1 (Process Create) was used to detect process creation activity, including the execution of Notepad and PowerShell. 

The exact Sysmon installation command and configuration file used during the lab were not retained in the available notes. 

### **2.6 Enabling Windows Filtering Platform Auditing** 

To collect blocked network connection events, failure auditing was enabled for the Windows Filtering Platform Connection subcategory. 

The following command was executed in an elevated Command Prompt: 

auditpol /set /subcategory:"Filtering Platform Connection" /failure:enable 

#### **Command explanation:** 

- auditpol : Windows command-line utility for managing audit policies. 

- /set : Configures an audit policy. 

- /subcategory : Specifies the audit policy subcategory. 

- Filtering Platform Connection : The Windows Filtering Platform connection auditing category. 

- /failure:enable : Enables auditing of failed connection events. 

The configuration was verified using: 

auditpol /get /subcategory:"Filtering Platform Connection" 

The output confirmed that failure auditing was enabled. 

This allowed Windows to generate Event ID 5157 when a network connection was blocked by Windows Filtering Platform. 

### **2.7 Updating the Wazuh Agent Configuration** 

The Wazuh agent configuration was updated to ensure that Event ID 5157 was not excluded from Security event collection. 

After saving the changes to ossec.conf , the Wazuh agent service was restarted using an elevated PowerShell session: 

Restart-Service -Name WazuhSvc 

The service status was checked with: 

Get-Service -Name WazuhSvc 

The expected status was: 

Status: Running 

### **2.8 Verifying the Windows Agent** 

After configuration, the Windows agent appeared in the Wazuh dashboard as: 

|**Field**|**Value**|
|---|---|
|Agent|SOC-|
|Name|Windows|
|Agent ID|001|
|Manager|192.168.56.1<br>0|
|Status|Active|



The agent successfully forwarded Windows and Sysmon events to the Wazuh manager. 

### **2.9 Windows Endpoint Validation** 

The endpoint was tested by generating normal process activity, failed logon events, and blocked network connection events. 

The collected events were used to validate the custom Wazuh detection rules: 

|**Activity**|**Windows Even**<br>**ID**|**t**<br>**Custom**<br>**Rule**|
|---|---|---|
|Notepad process creation|1 (Sysmon)|100100|
|PowerShell process<br>creation|1 (Sysmon)|100101|
|Failed Windows logon|4625|100102|
|Blocked network<br>connection|5157|100103|



The resulting alerts were reviewed in the Wazuh dashboard and used as evidence in the investigation reports. 

## **3. Custom Wazuh Detection Rules** 

### **3.1 Overview** 

Custom Wazuh rules were created to detect specific activities on the Windows endpoint. These rules extend the built-in Wazuh rules by matching particular event fields and generating alerts with custom rule IDs and severity levels. 

The following four custom detection rules were implemented: 

|**Rule**<br>**ID**|**Detection**|**Severity**<br>**Level**|**Event**|
|---|---|---|---|
|1001<br>00|Notepad process creation|5|Sysmon Event ID 1|
|1001<br>01|PowerShell process creation|7|Sysmon Event ID 1|
|1001<br>02|Windows failed login|8|Windows Event ID<br>4625|
|1001|Blocked network connection from|8|Windows Event ID|
|03|Kali||5157|



### **3.2 Locating the Custom Rules File** 

The custom detection rules were created in the Wazuh manager's local rules file: 

sudo nano /var/ossec/etc/rules/local_rules.xml 

#### **Command explanation:** 

- sudo : Executes the command with administrative privileges. 

- nano : Opens the Nano text editor. 

- /var/ossec/etc/rules/local_rules.xml : The file containing the custom Wazuh rules. 

The custom rules were added to this file. 

### **3.3 Creating the Custom Rules** 

The following XML configuration contains the four custom detection rules used in the lab. 

<group name="local,windows,sysmon,"> 

<rule id="100100" level="5"> 

- <if_sid>61603</if_sid> 

<field name="win.eventdata.image" type="pcre2">(?i)notepad\.exe$</field> <description>Notepad process creation detected</description> </rule> 

<rule id="100101" level="7"> 

<if_sid>61603</if_sid> 

- <field name="win.eventdata.image" 

type="pcre2">(?i)\\powershell\.exe$</field> <description>PowerShell process creation detected</description> 

- </rule> 

</group> 

- <group name="local,windows,authentication,"> 

- <rule id="100102" level="8"> 

- <if_sid>60122</if_sid> 

- <description>Windows failed login detected</description> 

- <group>authentication_failed,</group> 

- </rule> 

<rule id="100103" level="8"> 

- <if_sid>60104</if_sid> 

- <field name="win.eventdata.sourceAddress">192.168.56.20</field> 

<description>Blocked Network connection from kali</description> <group>network_scan,firewall_block,</group> 

- </rule> 

</group> 

### **3.4 Understanding the Rules** 

#### **Rule 100100 — Notepad Process Creation** 

- Parent rule: 61603 , which identifies Sysmon process creation events. 

- Event field: win.eventdata.image 

- Condition: Matches an image path ending in notepad.exe , case-insensitively. 

- Severity level: 5 

When Notepad is launched on the monitored Windows endpoint, this rule generates a custom alert. 

#### **Rule 100101 — PowerShell Process Creation** 

- Parent rule: 61603 

- Event field: win.eventdata.image 

- Condition: Matches an image path ending in powershell.exe , caseinsensitively. 

- Severity level: 7 

This rule detects the launch of PowerShell. The alert indicates process creation; it does not by itself establish that PowerShell was used maliciously. 

#### **Rule 100102 — Windows Failed Login** 

- Parent rule: 60122 , a built-in Wazuh rule for Windows logon failures. 

- Severity level: 8 

- Additional group: authentication_failed 

This rule generates a custom alert when the parent rule identifies a failed Windows login event. 

#### **Rule 100103 — Blocked Network Connection from Kali** 

- Parent rule: 60104 , a built-in Wazuh rule for Windows audit failure events. 

- Event field: win.eventdata.sourceAddress 

- Condition: Matches the Kali host's IP address, 192.168.56.20 . 

- Severity level: 8 

- Additional groups: network_scan , firewall_block 

This rule detects matching blocked network connection events originating from the Kali machine. 

### **3.5 Validating the Custom Rules** 

After editing the XML file, the Wazuh analysis engine was used to validate the configuration. 

sudo /var/ossec/bin/wazuh-analysisd -t 

#### **Command explanation:** 

- sudo : Runs the command with administrative privileges. 

- /var/ossec/bin/wazuh-analysisd : Wazuh's analysis engine. 

- -t : Tests the configuration and checks for errors. 

The validation completed successfully after the XML configuration was corrected. 

### **3.6 Restarting the Wazuh Manager** 

After validating the rules, the Wazuh manager was restarted to load the updated configuration. 

sudo systemctl restart wazuh-manager 

The manager service status was then checked: 

sudo systemctl status wazuh-manager 

The service was confirmed to be active after the restart. 

### **3.7 Verifying the Custom Rules** 

The rules were tested using controlled activities on the Windows endpoint and Kali Linux machine. 

|**Rule ID**|**Test Activity**|**Verification**|
|---|---|---|
|100100|Launch Notepad|Custom alert generated|
|100101|Launch PowerShell|Custom alert generated|
|100102|Generate a failed Windows<br>login|Custom alert generated|
|100103|Run an Nmap scan from Kali|Blocked connection alerts<br>generated|



The alerts were reviewed in the Wazuh dashboard and in the manager's alert log. 

To inspect recent alerts from the terminal: 

sudo tail -n 20 /var/ossec/logs/alerts/alerts.json 

To filter alerts for a specific custom rule, for example rule 100103 : 

sudo grep '"id":"100103"' /var/ossec/logs/alerts/alerts.json 

The alert log was used to confirm that the custom rules generated alerts during testing. 

### **3.8 Verifying Blocked Connection Events** 

For the Nmap detection, Windows Filtering Platform Event ID 5157 was collected and forwarded to the Wazuh manager. 

The following command was used to inspect archived events: 

sudo grep '"eventID":"5157"' /var/ossec/logs/archives/archives.json | tail -5 

This displayed the last five matching archived events, allowing the forwarded event data to be inspected. 

The custom rule 100103 was then verified by checking for its alerts in the Wazuh dashboard and alert log. 

### **3.9 Results** 

All four custom rules were successfully tested in the lab: 

- Rule 100100 generated alerts for Notepad process creation. 

- Rule 100101 generated alerts for PowerShell process creation. 

- Rule 100102 generated alerts for failed Windows logins. 

- Rule 100103 generated alerts for blocked network connections from Kali. 

These tests confirmed that the custom rules were loaded and that matching events generated alerts in Wazuh. 

## **4. Kali Linux Setup** 

### **4.1 Kali Linux Configuration** 

Kali Linux was used as the testing machine to perform network scans against the Windows endpoint and validate the custom Wazuh detection rules. 

Kali was connected to the same VirtualBox Host-only network as the Wazuh server and Windows endpoint. 

|**Component**|**Configuration**|
|---|---|
|Operating<br>System|Kali Linux|
|Purpose|Network scanning and detection<br>testing|
|Network|VirtualBox Host-only|
|IP Address|192.168.56.20|
|Subnet|192.168.56.0/24|
|Target|Windows endpoint (192.168.56.1)|



### **4.2 Network Configuration** 

The Kali machine used the Host-only network to communicate with the Windows endpoint and Wazuh server. 

The lab network was configured as follows: 

#### **Device IP Address** 

|Wazuh Server|192.168.56.1<br>0|
|---|---|
|Windows<br>Endpoint|192.168.56.1|
|Kali Linux|192.168.56.2<br>0|



The Kali machine's IP address can be checked using: 

ip addr 

This command displays the network interfaces and their assigned IP addresses. 

### **4.3 Verifying Network Connectivity** 

Connectivity between the Kali machine and the Windows endpoint can be checked using: 

ping -c 4 192.168.56.1 

The command sends four ICMP echo requests to the Windows endpoint. A response indicates that the endpoint is reachable over ICMP. A lack of response does not necessarily mean the host is unreachable, since Windows Firewall may block ping requests. 

### **4.4 Nmap Network Scanning** 

Nmap was used from Kali Linux to scan selected ports on the Windows endpoint. 

The following command was used during the lab: 

nmap -sS -Pn -p 135,139,445,3389 192.168.56.1 

#### **Command explanation:** 

- nmap : Network exploration and port scanning tool. 

- -sS : Performs a TCP SYN scan. 

- -Pn : Skips host discovery and treats the target as online. 

- -p : Specifies the ports to scan. 

- 135,139,445,3389 : The selected Windows ports. 

- 192.168.56.1 : The Windows endpoint's IP address. 

The scan was performed against the Windows endpoint in the lab. 

The selected ports were: 

|**Port**|**Common Service**|
|---|---|
|135|Microsoft RPC|
|139|NetBIOS Session Service|
|445|SMB|
|3389|Remote Desktop Protocol<br>(RDP)|



The scan results showed that the selected ports were filtered. 

### **4.5 Verifying Network Scan Detection** 

During the scan, Windows Filtering Platform generated blocked connection events with Event ID 5157 . 

The Wazuh agent forwarded these events to the Wazuh manager, where custom rule 100103 matched events originating from the Kali IP address, 192.168.56.20 . 

The resulting alerts were reviewed in the Wazuh dashboard and the manager's alert log. 

This verified the network detection workflow from Kali to Windows and then to Wazuh. 

## **5. Final Lab Verification** 

After configuring the Ubuntu Wazuh server, Windows endpoint, and Kali Linux machine, the lab was tested to verify that the monitoring and detection components were working together. 

### **5.1 Verification Checklist** 

|**Component**|**Verification**|**Result**|
|---|---|---|
|Wazuh Manager|Service status checked|Active|
|Wazuh Dashboard|Accessed through browser|Accessible|
|Windows Agent<br>Sysmon|Agent registered with manager<br>Process creation events<br>collected|Active<br>Verified|
|Notepad Detection|Custom rule100100tested|Alert generated|
|PowerShell Detection|Custom rule100101tested|Alert generated|
|Failed Login|Custom rule100102tested|Alert generated|



Detection 

Alerts Nmap Detection Custom rule 100103 tested generated Alerts and visualizations Dashboard Verified reviewed 

### **5.2 Reviewing Custom Detection Alerts** 

The custom detection alerts were reviewed in the Wazuh dashboard. 

The following rule IDs were used to identify the test alerts: 

100100 - Notepad process creation 

100101 - PowerShell process creation 100102 - Windows failed login 

100103 - Blocked network connection from Kali 

The alerts were inspected to review their timestamps, rule IDs, severity levels, agent information, and event details. 

### **5.3 Reviewing Alert Logs** 

The Wazuh manager's alert log was used to inspect generated alerts. 

sudo tail -n 20 /var/ossec/logs/alerts/alerts.json 

To search for alerts generated by a particular rule, the following command can be used: 

sudo grep '"id":"100103"' /var/ossec/logs/alerts/alerts.json 

This searches the alert log for the custom Nmap detection rule. 

### **5.4 Final Outcome** 

The lab successfully demonstrated the collection and analysis of Windows endpoint events using Wazuh. 

Four custom detection rules were implemented and tested using controlled activities. The resulting alerts were reviewed in the Wazuh dashboard, and investigation reports were created for failed login activity and the Nmap network scan. 

The lab demonstrates the following capabilities: 

- Centralized Windows event monitoring 

- Sysmon process creation monitoring 

- Custom Wazuh rule creation and validation 

- Windows failed login detection 

- Blocked network connection detection 

- Alert investigation and evidence documentation 

- Dashboard-based security monitoring 

## **6. Related Documentation** 

- <u>Project README</u> 

- <u>Failed Login Investigation</u> 

- <u>Nmap Network Scan Investigation</u> 

