# Commands Reference: Lab 02

All commands actually used while hardening the Ubuntu VM, grouped by phase. Output shown is representative of what was observed during the build (see `screenshots/` and `README.md` for full context and exact captured output).

## Phase 1: Baseline Assessment

```bash
whoami
hostname
hostnamectl
id
groups
sudo -l
```

## Phase 2: Account Security Assessment

```bash
awk -F: '$7 ~ /bash|sh/ {print $1, $7}' /etc/passwd
sudo passwd -S root
```

## Phase 3: Password Hardening

```bash
sudo chage -M 90 -m 1 -W 14 sia
sudo chage -l sia
```

## Phase 4: Firewall Assessment

```bash
sudo ufw status verbose
```

## Phase 5: Network Service Enumeration

```bash
sudo ufw status numbered
sudo ss -tuln
```

## Phase 6: Attack Surface Reduction

Disabling CUPS (service, socket, and path activation units):

```bash
sudo systemctl stop cups
sudo systemctl disable cups
sudo systemctl disable --now cups.socket
sudo systemctl disable --now cups.path
```

Disabling Avahi (service and socket activation unit):

```bash
sudo systemctl disable avahi-daemon
sudo systemctl disable --now avahi-daemon.socket
systemctl status avahi-daemon
systemctl status avahi-daemon.socket
```

Re-verifying exposure after disabling both services:

```bash
sudo ss -tuln
```

## Phase 7: SSH Assessment and Hardening

Checking current directives:

```bash
sudo grep -E "^(PermitRootLogin|PasswordAuthentication|PubkeyAuthentication|X11Forwarding|PermitEmptyPasswords)" /etc/ssh/sshd_config
sudo sshd -T | grep -E "permitrootlogin|passwordauthentication|pubkeyauthentication|x11forwarding|permitemptypasswords"
```

Backing up the config before editing, then hardening:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo nano /etc/ssh/sshd_config
```

Directive changed:

```
X11Forwarding no
```

Validating and applying (same discipline established in Lab 01, critical on a config that controls remote access):

```bash
sudo sshd -t
sudo systemctl restart ssh
grep X11Forwarding /etc/ssh/sshd_config
```

## Phase 8: Linux Audit Framework

```bash
sudo apt update
sudo apt install auditd audispd-plugins -y
sudo auditctl -s
```

## Phase 9: Detection Engineering

```bash
sudo auditctl -w /etc/passwd -p wa -k passwd_changes
```

Note: `auditctl -w` adds the rule to the live kernel ruleset only. It does not survive a reboot unless it is also added to a file under `/etc/audit/rules.d/` and loaded with `augenrules --load`. That persistence step was not yet done in this lab and is called out in [`lessons-learned.md`](lessons-learned.md) as a known gap to close before this rule can be relied on long-term.

## Phase 10: Detection Verification

Triggering a real, detectable event against the watched file:

```bash
sudo cp /etc/passwd /tmp/passwd.backup
sudo chmod 644 /etc/passwd
sudo touch /etc/passwd
```

Confirming the events were captured:

```bash
sudo ausearch -k passwd_changes
```
