# Commands Reference: Lab 04

All commands used, in the order they were executed. See `README.md` for context, screenshots, and reasoning.

## Phase 1: Initial Service Discovery

```bash
ping -c 4 192.168.56.20
nmap 192.168.56.20
nmap -sV 192.168.56.20
```

## Phase 2: SSH Enumeration

```bash
nmap --script ssh-auth-methods 192.168.56.20
nmap --script ssh2-enum-algos 192.168.56.20
nmap --script vuln 192.168.56.20
```

## Phase 3: SSH Configuration Review

```bash
sudo sshd -T | grep -E "permitrootlogin|passwordauthentication|pubkeyauthentication|maxauthtries|x11forwarding|permitemptypasswords"
```

## Phase 4: Security Audit

```bash
sudo lynis audit system
```

## Phase 5: System Updates

```bash
sudo apt update
sudo apt upgrade -y
```

## Phase 6: Fail2Ban Deployment

```bash
sudo apt install fail2ban -y
sudo systemctl status fail2ban
fail2ban-client --version
sudo fail2ban-client status
```

## Phase 7: Brute-Force Simulation

From Kali (repeated, deliberately wrong password):

```bash
ssh sia@192.168.56.20
```

From Ubuntu, checking the result:

```bash
sudo grep -a "Failed password" /var/log/auth.log | tail -5
sudo fail2ban-client status sshd
```

## Phase 8: SSH Public Key Authentication

```bash
ssh-keygen -t ed25519 -C "homelab-lab4"
ssh-copy-id -i ~/.ssh/id_ed25519.pub sia@192.168.56.20
ssh sia@192.168.56.20
```

## Phase 9: SSH Hardening

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo nano /etc/ssh/sshd_config
```

Directives changed:

```
PasswordAuthentication no
PermitRootLogin no
```

```bash
sudo sshd -t
sudo systemctl restart ssh
```

Verification (deliberately forcing a non-key attempt, and a root attempt):

```bash
ssh -o PubkeyAuthentication=no sia@192.168.56.20
ssh root@192.168.56.20
```

## Phase 10: External Security Assessment

```bash
sudo apt install ssh-audit -y
ssh-audit 192.168.56.20
```

## Incident Response (Problems Encountered)

```bash
sudo systemctl restart ssh
sudo journalctl -xe
systemctl status ssh.socket
systemctl status ssh.service
sudo ss -ltnp | grep :22
sudo kill 3208
sudo systemctl start ssh
systemctl status ssh
```
