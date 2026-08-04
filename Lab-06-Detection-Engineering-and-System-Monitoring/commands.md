# Commands Reference: Lab 06

All commands used, in the order they were executed, without much explanation. See `README.md` for context, findings, and screenshots.

## Phase 0: Baseline Visibility Audit

```bash
systemctl status auditd
sudo auditctl -l
journalctl --disk-usage
cat /etc/rsyslog.d/*
docker ps -a
docker start dvwa
docker logs dvwa --tail 20
fail2ban-client status sshd
```

## Phase 1: auditd Detection Rules

```bash
sudo tee /etc/audit/rules.d/lab06-detection.rules > /dev/null << 'EOF'
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/sudoers -p wa -k sudo_actions
-w /etc/sudoers.d/ -p wa -k sudo_actions
-w /usr/bin/sudo -p x -k privilege_escalation
-w /usr/bin/su -p x -k privilege_escalation
-w /etc/ssh/sshd_config -p wa -k ssh_config_change
-w /etc/ufw/ -p wa -k firewall_changes
-w /var/run/docker.sock -p rwa -k docker_socket_access
-w /etc/audit/ -p wa -k audit_config_change
-a always,exit -F arch=b64 -S execve -k exec_actions
-a always,exit -F arch=b32 -S execve -k exec_actions
EOF

sudo augenrules --load
sudo auditctl -l
sudo ausearch -k exec_actions -ts recent | tail -20
```

## Phase 2: Docker Logging to Syslog

```bash
docker stop dvwa
docker rm dvwa
docker run -d --name dvwa --log-driver syslog --log-opt tag="dvwa-apache" -p 80:80 vulnerables/web-dvwa

journalctl -t dvwa-apache --since "2 minutes ago"
journalctl -t dvwa-apache --since "1 minute ago" | grep -i "GET\|POST"
```

## Phase 3: Custom Detection Script

```bash
mkdir -p ~/lab06
nano ~/lab06/detect.py
sudo python3 ~/lab06/detect.py
```

Manual test triggers used to confirm each watcher before the real attack replay:

```bash
curl "http://192.168.56.20/vulnerabilities/sqli/?id=1' UNION SELECT user,password FROM users #"
curl "http://192.168.56.20/vulnerabilities/xss_s/?txtName=test&mtxMessage=<script>alert(1)</script>"
ssh -o PubkeyAuthentication=no wrongpassword@192.168.56.20
whoami
cat /etc/passwd
```

Discovering and fixing the SSH watcher's regex gap:

```bash
grep sshd /var/log/auth.log
sed -i 's/OLDPATTERN/Failed password|Connection closed by authenticating user.*preauth/' ~/lab06/detect.py
```

## Phase 4: Attack Replay from Kali

```bash
# Attack 1: Reconnaissance
nmap -p- -sV 192.168.56.20

# Attack 2: SQL Injection (browser, DVWA SQL Injection module)
# id=1' UNION SELECT user,password FROM users #

# Attack 3: Command Injection (browser, DVWA Command Injection module)
# 127.0.0.1 && whoami

# Attack 4: Stored XSS (browser, DVWA Guestbook)
# <script>alert('XSS')</script>

# Attack 5: Cookie tampering (Burp Suite Match and Replace, from Lab 05's technique)
# security=medium -> security=low, then resubmit the SQLi payload
```

## Cross-Lab Validation: Fail2ban Investigation

```bash
# Symptom
fail2ban-client status sshd

# Root cause 1: check whether the default filter even matches this log line
sudo fail2ban-regex /var/log/auth.log sshd

# Fix attempt 1 (failed: wrong pattern order)
sudo tee /etc/fail2ban/filter.d/sshd.local > /dev/null << 'EOF'
[Definition]
failregex = %(known/failregex)s
            ^Connection closed by authenticating user \S+ <HOST> port \d+ \[preauth\]\s*$
EOF
cat /etc/fail2ban/filter.d/sshd.local

# Root cause 2: check the ignoreip whitelist
grep -i ignoreip /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# Final fix: reorder the custom regex before the default include
sudo tee /etc/fail2ban/filter.d/sshd.local > /dev/null << 'EOF'
[Definition]
failregex = ^Connection closed by authenticating user \S+ <HOST> port \d+ \[preauth\]\s*$
            %(known/failregex)s
EOF

# Validate against real log data
sudo fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.local

# Apply and verify live, against a real (non-whitelisted) source IP
sudo systemctl restart fail2ban
ssh -o PubkeyAuthentication=no -o PreferredAuthentications=password sia@192.168.56.20 -p 22
sudo fail2ban-client status sshd
```

## Oracle Cloud Wazuh Attempt (abandoned, see README Problems Encountered)

```bash
free -h
```

Remainder performed via the OCI web console: VCN Wizard ("Create VCN with Internet Connectivity"), Compute -> Create Instance (Ampere A1, attempted at 2 OCPU/12GB and 1 OCPU/6GB across multiple availability domains), each attempt failing with "Out of capacity for shape VM.Standard.A1.Flex".
