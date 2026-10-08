# Commands: Lab 07 - Windows Security & Active Directory

Commands organized by phase. PowerShell commands run on the Windows Server (DC); Kali commands are marked with 🐉.

## Phase 1: Build the Domain Controller

```powershell
# Identify which adapter is the internal (CyberLab) one.
# The internal adapter shows a 169.254.x APIPA address and no gateway; the NAT adapter shows 10.0.2.x with a gateway.
Get-NetIPConfiguration

# Static IP on the internal adapter (here it was "Ethernet 2"); a DC points DNS at itself.
New-NetIPAddress -InterfaceAlias "Ethernet 2" -IPAddress 192.168.56.30 -PrefixLength 24
Set-DnsClientServerAddress -InterfaceAlias "Ethernet 2" -ServerAddresses 127.0.0.1
Get-NetIPConfiguration -InterfaceAlias "Ethernet 2"   # verify 192.168.56.30

# Rename the server, then reboot.
Rename-Computer -NewName "DC01" -Restart
hostname                                              # verify: DC01

# Install AD DS and promote to a brand-new forest/domain.
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Install-ADDSForest -DomainName "CYBERLAB.local" -DomainNetbiosName "CYBERLAB" -InstallDNS
# (prompts for a DSRM/SafeMode password, then reboots automatically)

# After reboot, verify the domain.
whoami                    # cyberlab\administrator
Get-ADDomain             # DNSRoot: CYBERLAB.local, NetBIOSName: CYBERLAB
Get-ADDomainController    # Name: DC01
```

## Phase 2: Default Posture Assessment

```powershell
Get-ADDefaultDomainPasswordPolicy        # baseline: MinPasswordLength 7, LockoutThreshold 0
Get-ADGroupMember "Domain Admins"        # baseline: only Administrator
Get-SmbServerConfiguration | Select RequireSecuritySignature, EnableSecuritySignature, EnableSMB1Protocol
# baseline: signing required (True), SMBv1 disabled (False)
```

## Phase 3: Hardening via Group Policy

GUI (GPMC): `Group Policy Management` -> `Forest: CYBERLAB.local` -> `Domains` -> `CYBERLAB.local` ->
right-click **Default Domain Policy** -> **Edit**.

- **Password Policy** (`Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Password Policy`):
  - Minimum password length: `7` -> **`14`**
- **Account Lockout Policy** (same path, `Account Lockout Policy`):
  - Account lockout threshold: `0` -> **`5`** (accept the suggested 30-minute duration + 30-minute reset)
- **LLMNR** (`Computer Configuration > Policies > Administrative Templates > Network > DNS Client`):
  - "Turn off multicast name resolution" -> **Enabled**

```powershell
# Apply immediately instead of waiting ~90 minutes.
gpupdate /force

# Verify.
Get-ADDefaultDomainPasswordPolicy        # MinPasswordLength 14, LockoutThreshold 5, LockoutDuration 00:30:00
Get-ItemProperty "HKLM:\Software\Policies\Microsoft\Windows NT\DNSClient" -Name EnableMulticast   # 0
```

## Phase 4: Detection (advanced audit policy)

```powershell
# Current state.
auditpol /get /category:"Logon/Logoff","Account Logon","Account Management"

# Enable the categories that matter for the attack replay.
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Credential Validation" /success:enable /failure:enable
auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable

# Re-verify (should now read "Success and Failure").
auditpol /get /category:"Logon/Logoff","Account Logon","Account Management"
```

## Phase 5: Attack Replay

```powershell
# Create two target users (on the DC). Password meets the new 14-char policy.
New-ADUser -Name "John Doe"  -SamAccountName "jdoe"   -UserPrincipalName "jdoe@CYBERLAB.local"   -AccountPassword (ConvertTo-SecureString "<lab-test-password>" -AsPlainText -Force) -Enabled $true -PasswordNeverExpires $true
New-ADUser -Name "Anna Smith" -SamAccountName "asmith" -UserPrincipalName "asmith@CYBERLAB.local" -AccountPassword (ConvertTo-SecureString "<lab-test-password>" -AsPlainText -Force) -Enabled $true -PasswordNeverExpires $true
Get-ADUser -Filter * | Select Name, SamAccountName, Enabled
```

```bash
# 🐉 KALI: confirm network and fingerprint the DC.
ip a                                  # eth1 = 192.168.56.10 (CyberLab)
nmap -Pn 192.168.56.30                # DC fingerprint: 53/88/135/139/389/445/636/3268/3389...

# 🐉 KALI: brute-force one account (triggers lockout at 5).
echo -e "Password1\nWinter2024\nSummer2024\nWelcome1\nCompany123\nP@ssword1\nChangeme123" > ~/passwords.txt
nxc smb 192.168.56.30 -u jdoe -p ~/passwords.txt       # locks after 5 -> STATUS_ACCOUNT_LOCKED_OUT

# 🐉 KALI: password spray one password across many accounts (evades lockout).
echo -e "administrator\nasmith\njdoe" > ~/users.txt
nxc smb 192.168.56.30 -u ~/users.txt -p 'Spring2025!' --continue-on-success
```

```powershell
# DC side: confirm lockout and detection.
Get-ADUser jdoe -Properties LockedOut | Select Name, LockedOut             # True
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4740} -MaxEvents 3 | Format-List TimeCreated, Message
(Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625,4776} -MaxEvents 50 | Measure-Object).Count

# To unlock the test account after the exercise:
# Unlock-ADAccount -Identity jdoe
```
