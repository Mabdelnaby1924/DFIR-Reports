


# QUICK LOGISTICS LLC Incident Timeline

| **Time (UTC)**      | **Source** | **Event**                                                                    | **Evidence**        |
| ------------------- | ---------- | ---------------------------------------------------------------------------- | ------------------- |
| 2023-01-13 09:25:31 | Email      | Initial spear-phishing email delivered to julianne[.]westcott@hotmail[.]com. | `dump.eml` Headers  |
| 2023-01-13 17:10:49 | Host       | User executed the malicious LNK file, spawning hidden PowerShell.            | EventID 4104        |
| 2023-01-13 17:10:54 | Network    | PowerShell downloaded the C2 stager from `files.bpakcaging.xyz`.             | EventID 4104 / PCAP |
| 2023-01-13 17:12:17 | Host       | Attacker attempted to execute <br>`Invoke-Seatbelt.ps1` from GitHub.         | EventID 4104        |
| 2023-01-13 17:13:xx | Host       | Attacker downloaded `sb.exe` and `sq3.exe` to disk for recon.                | EventID 4104 / PCAP |
| 2023-01-13 17:15:xx | Host       | Attacker queried Sticky Notes DB and obtained KeePass Master Password.       | EventID 4104        |
| 2023-01-13 17:17:xx | Network    | `protected_data.kdbx` exfiltrated via DNS to `167.71.211.113`.               | EventID 4104        |
