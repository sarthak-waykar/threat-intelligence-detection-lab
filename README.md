# Threat Intelligence Detection Lab

Honours degree project (BCA, JSPM University, Pune). A virtual lab built in VirtualBox to simulate attacks and detect them with open-source security tools.

## Lab Setup
|    VM    |             OS             |           Role           |        IP      |
|----------|----------------------------|--------------------------|----------------|
| kali     | Kali Linux                 | Attacker                 | 192.168.56.103 |
| windows  | Windows 7 Starter (32-bit) | Victim                   | 192.168.56.101 |
| detector | Ubuntu Server 22.04        | Detection server (Wazuh) | 192.168.56.102 |

- Host: Windows PC, VirtualBox
- Network: host-only network with static IPs

## What I Did
- Installed and configured three virtual machines in VirtualBox
- Set static IP addresses and tested connectivity with ping
- Installed the Wazuh all-in-one server on the detector VM
- Installed the Wazuh agent on the Windows 7 VM and registered it with the detector (agent shows Active)
- Ran Nmap scans against the Windows 7 VM to enumerate open ports and services
- Confirmed the Windows 7 VM was vulnerable to MS17-010 (EternalBlue, CVE-2017-0143) using Nmap's smb-vuln-ms17-010 script
- Successfully exploited MS17-010 using Metasploit's ms17_010_psexec module with valid SMB credentials, gaining a SYSTEM-level Meterpreter session
- Verified detection in Wazuh via the Threat Hunting dashboard and MITRE ATT&CK tactic mapping

## Key Findings
- Windows 7 Starter (32-bit), SMB exposed on ports 135, 139, 445
- Confirmed vulnerable to MS17-010 (EternalBlue, CVE-2017-0143) — the same flaw used in the WannaCry ransomware attack
- Successfully exploited using Metasploit, gaining `NT AUTHORITY\SYSTEM` access
- Wazuh's own vulnerability scanner independently flagged the host with 3 Critical, 143 High, and 112 Medium severity findings
- Wazuh detected the attack activity, showing a clear spike in events and MITRE ATT&CK tactics (Privilege Escalation, Defense Evasion, Persistence, Initial Access) around the time of exploitation

## Challenges and Fixes
- The Wazuh deployment command needs PowerShell 3.0+, which Windows 7 doesn't have. I downloaded the MSI on the detector VM, served it over a temporary HTTP server, and installed it from Command Prompt with msiexec.
- The ms17_010_eternalblue module failed because it only supports x64 payloads, but the target was 32-bit Windows 7.
  
  [EternalBlue x64 error]<img width="1920" height="1080" alt="07-eternalblue-x64-error" src="https://github.com/user-attachments/assets/8cb5b834-aa69-4321-bb6b-d1029419c04b" />

- Switched to ms17_010_psexec, which supports both architectures. It initially failed with "Unable to find accessible named pipe" across several common pipe names, and anonymous SMB login attempts returned STATUS_LOGON_FAILURE.
  
  [Auth troubleshooting]<img width="1920" height="1080" alt="08-psexec-auth-troubleshooting" src="https://github.com/user-attachments/assets/6ba20430-b6f4-4529-928b-3d06903f14ff" />
- This was resolved by authenticating with valid local SMB credentials, after which the exploit succeeded.

## Screenshots

### Nmap scan results
[Nmap scan]<img width="1920" height="1080" alt="01-nmap-scan" src="https://github.com/user-attachments/assets/097b204a-20e4-4f1f-b91f-bf7b13278c03" />

### MS17-010 vulnerability confirmed
[MS17-010 vulnerable]<img width="1920" height="1080" alt="02-smb-vuln-ms17-010" src="https://github.com/user-attachments/assets/5355885a-52a1-436b-ab12-d18c25108f8f" />

### EternalBlue exploitation — SYSTEM access obtained
[EternalBlue exploit success]<img width="1920" height="1080" alt="03-eternalblue-exploit-success" src="https://github.com/user-attachments/assets/19e4f333-e775-44d3-9e30-6b4fc2c06e84" />

### Wazuh agent connected
[Wazuh agent active]<img width="1919" height="1001" alt="04-wazuh-agent-active" src="https://github.com/user-attachments/assets/d688e8b8-faa9-487f-9685-b3d041c575ae" />

### Wazuh detecting the attack
[Wazuh threat hunting]<img width="1920" height="1008" alt="05-wazuh-threat-hunting" src="https://github.com/user-attachments/assets/afa302d4-47db-4d00-8fcb-d35408576b91" />

### Wazuh vulnerability scan
[Wazuh vulnerability detection]<img width="1920" height="1008" alt="06-wazuh-vulnerability-detection" src="https://github.com/user-attachments/assets/05f8a25a-f619-46ab-b7b6-8fa5e45ea976" />

## Next Steps
- Enable Sysmon on Windows 7 for deeper process-level visibility
- Simulate additional attack techniques and tune Wazuh detection rules
