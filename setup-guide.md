SOC Home Lab — Setup Guide
This guide documents the setup of the SOC lab built with Wazuh, Ubuntu Server, a Windows endpoint, and Kali Linux.
> **Lab network**
>
> | Device | Address |
> |---|---|
> | Ubuntu/Wazuh server (host-only) | `192.168.56.10` |
> | Windows endpoint | `192.168.56.1` |
> | Kali Linux | `192.168.56.20` |
> | Host-only subnet | `192.168.56.0/24` |
>
> The Ubuntu VM also uses NAT (`10.0.2.15`) for internet access.
1. Ubuntu Server and Wazuh
1.1 Virtual machine configuration
Setting	Value
VM name	`SOC-Wazuh`
OS	Ubuntu Server 24.04.5 LTS
RAM	6 GB
CPUs	4
Disk	70 GB, dynamically allocated
Adapter 1	NAT
Adapter 2	Host-only
Username	`socadmin`
Hostname	`soc-wazuh`
1.2 Check network interfaces
Run on the Ubuntu server:
```bash
ip addr
```
The NAT interface should have internet connectivity. The host-only interface should use `192.168.56.10/24`.
1.3 Configure the static host-only address
The network was configured with Netplan. Open the Netplan file:
```bash
sudo nano /etc/netplan/01-soc-lab.yaml
```
Example configuration for the lab interfaces:
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses:
        - 192.168.56.10/24
```
Apply the configuration and verify the addresses:
```bash
sudo netplan apply
ip addr
```
> Interface names can differ between systems. Use the names shown by `ip addr` on your own VM.
1.4 Verify internet connectivity
```bash
ping -c 4 8.8.8.8
ping -c 4 google.com
```
The first command checks IP connectivity; the second also checks DNS resolution.
1.5 Install prerequisites
```bash
sudo apt update
sudo apt install -y curl gnupg apt-transport-https
```
1.6 Install Wazuh
Wazuh 4.14 was installed as an all-in-one deployment (manager, indexer, and dashboard). Download and run the installer:
```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```
The `-a` option selects the all-in-one installation.
Open the dashboard in a browser:
```text
https://192.168.56.10
```
The browser may show a certificate warning because the lab dashboard uses a locally generated certificate.
1.7 Retrieve dashboard credentials
The installer creates a password archive. The following command prints the password file from that archive:
```bash
sudo tar -xOf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```
Security: Do not commit the password archive, password file, or credentials to GitHub. Keep credentials private and redact them from screenshots.
2. Windows Endpoint
2.1 Endpoint configuration
Setting	Value
Endpoint	Windows PC
Agent name	`SOC-Windows`
Agent ID	`001`
Wazuh manager	`192.168.56.10`
Windows host-only address	`192.168.56.1`
Log source	Windows Event Logs and Sysmon
The Windows endpoint was connected to the same host-only network as the Wazuh server.
2.2 Install and configure the Wazuh agent
Install the Wazuh agent on Windows and register it with the manager at `192.168.56.10`, using the agent name `SOC-Windows`.
The agent configuration file is:
```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```
The agent was configured to collect the following event channels:
`Application`
`Security`
`System`
`Microsoft-Windows-Sysmon/Operational`
The Security channel configuration was checked to ensure Event ID `5157` was not excluded.
2.3 Sysmon event collection
Sysmon was used to provide detailed Windows activity telemetry. The Wazuh agent collected the Sysmon Operational channel:
```text
Microsoft-Windows-Sysmon/Operational
```
Sysmon Event ID `1` (Process Create) was used to test process-creation detections for Notepad and PowerShell. The exact Sysmon installer command and configuration file used during the lab were not retained, so they are not reproduced here.
2.4 Enable Windows Filtering Platform failure auditing
Run the following in an elevated Command Prompt:
```powershell
auditpol /set /subcategory:"Filtering Platform Connection" /failure:enable
```
Verify the setting:
```powershell
auditpol /get /subcategory:"Filtering Platform Connection"
```
Failure auditing allows Windows to generate Event ID `5157` when Windows Filtering Platform blocks a connection.
2.5 Restart and verify the Wazuh agent
After saving any agent configuration changes, restart the service from an elevated PowerShell window:
```powershell
Restart-Service -Name WazuhSvc
Get-Service -Name WazuhSvc
```
The expected service status is `Running`. In the Wazuh dashboard, verify that agent `SOC-Windows` (ID `001`) is active and forwarding events.
3. Custom Wazuh Detection Rules
3.1 Rule summary
Rule ID	Detection	Severity	Event
`100100`	Notepad process creation	5	Sysmon Event ID 1
`100101`	PowerShell process creation	7	Sysmon Event ID 1
`100102`	Windows failed login	8	Windows Event ID 4625
`100103`	Blocked connection from Kali	8	Windows Event ID 5157
3.2 Open the local rules file
Run on the Ubuntu Wazuh server:
```bash
sudo nano /var/ossec/etc/rules/local_rules.xml
```
Add the following rules to the file:
```xml
<group name="local,windows,sysmon,">
  <rule id="100100" level="5">
    <if_sid>61603</if_sid>
    <field name="win.eventdata.image" type="pcre2">(?i)notepad\.exe$</field>
    <description>Notepad process creation detected</description>
  </rule>

  <rule id="100101" level="7">
    <if_sid>61603</if_sid>
    <field name="win.eventdata.image" type="pcre2">(?i)\\powershell\.exe$</field>
    <description>PowerShell process creation detected</description>
  </rule>
</group>

<group name="local,windows,authentication,">
  <rule id="100102" level="8">
    <if_sid>60122</if_sid>
    <description>Windows failed login detected</description>
    <group>authentication_failed,</group>
  </rule>

  <rule id="100103" level="8">
    <if_sid>60104</if_sid>
    <field name="win.eventdata.sourceAddress">192.168.56.20</field>
    <description>Blocked Network connection from kali</description>
    <group>network_scan,firewall_block,</group>
  </rule>
</group>
```
These rules use built-in parent rules to match the relevant event types. Rule `100103` additionally matches the Kali source address.
3.3 Validate the rules
```bash
sudo /var/ossec/bin/wazuh-analysisd -t
```
A successful validation indicates that the analysis configuration passed the syntax check.
3.4 Restart the Wazuh manager
```bash
sudo systemctl restart wazuh-manager
sudo systemctl status wazuh-manager
```
Confirm that the manager is active after the restart.
3.5 Review generated alerts
To inspect the latest alerts:
```bash
sudo tail -n 20 /var/ossec/logs/alerts/alerts.json
```
To filter for the Kali network detection rule:
```bash
sudo grep '"id":"100103"' /var/ossec/logs/alerts/alerts.json
```
To inspect archived Windows Filtering Platform events:
```bash
sudo grep '"eventID":"5157"' /var/ossec/logs/archives/archives.json | tail -5
```
4. Kali Linux and Network Scan
4.1 Kali configuration
Kali Linux was used to perform a controlled scan against the Windows endpoint.
Setting	Value
Network	VirtualBox Host-only
Kali address	`192.168.56.20`
Subnet	`192.168.56.0/24`
Target	Windows endpoint (`192.168.56.1`)
Check Kali's network address:
```bash
ip addr
```
4.2 Check connectivity
```bash
ping -c 4 192.168.56.1
```
A missing ping response does not necessarily mean the endpoint is unreachable; Windows Firewall may block ICMP.
### 4.3 Run the Nmap Scan

The lab scan targeted selected Windows ports:

```bash
nmap -sS -Pn -p 135,139,445,3389 192.168.56.1
```

| Option | Meaning |
|---|---|
| `-sS` | TCP SYN scan |
| `-Pn` | Skip host discovery and treat the target as online |
| `-p` | Specify ports to scan |
| `135,139,445,3389` | Selected ports |
| `192.168.56.1` | Windows endpoint |

The selected ports were reported as filtered in the lab. Windows Filtering Platform generated Event ID `5157` for blocked connections. The Wazuh agent forwarded matching events, and custom rule `100103` generated alerts for the Kali source address.

---

## 5. Final Verification

| Component or test | Verification |
|---|---|
| Wazuh manager | Service active |
| Wazuh dashboard | Accessible in browser |
| Windows agent | `SOC-Windows` (ID `001`) active |
| Notepad | Rule `100100` alert generated |
| PowerShell | Rule `100101` alert generated |
| Failed login | Rule `100102` alert generated |
| Kali Nmap scan | Rule `100103` alert generated |
| Investigation reports | Failed-login and Nmap reports documented |

The lab demonstrated centralized Windows event monitoring, Sysmon process monitoring, custom rule creation and validation, failed-login detection, blocked-connection detection, and dashboard-based alert review.

---

## 6. Related Documentation

- [Project README](README.md)
- [Failed Login Investigation](investigation/failed-login-investigation.md)
- [Nmap Network Scan Investigation](investigation/nmap-scan-investigation.md)
