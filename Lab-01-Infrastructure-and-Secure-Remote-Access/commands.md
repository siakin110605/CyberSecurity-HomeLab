# Commands Reference: Lab 01

All commands actually used while building this lab, grouped by phase. Output shown is representative of what was observed during the build (see `screenshots/` and the main `README.md` for full context).

## 1. Network Configuration (Ubuntu)

Checking network interfaces before and after configuring the internal network adapter:

```bash
ip a
```

Assigning a static IP on the internal network adapter (`enp0s8`) using NetworkManager:

```bash
sudo nmcli connection modify Internal ipv4.addresses 192.168.56.20/24 ipv4.method manual
sudo nmcli connection up Internal
```

Confirming the static IP was applied:

```bash
ip addr show enp0s8
hostname -I
```

## 2. Connectivity Verification

From Ubuntu to Kali:

```bash
ping -c 4 192.168.56.10
```

From Kali to Ubuntu:

```bash
ping -c 4 192.168.56.20
```

## 3. SSH Server Installation (Ubuntu)

Initial check (before the package was installed, this is expected to fail):

```bash
systemctl status ssh
```

Installing OpenSSH server:

```bash
sudo apt update
sudo apt install openssh-server -y
```

Enabling and starting the service:

```bash
sudo systemctl enable --now ssh
systemctl status ssh
```

Confirming the daemon is listening on port 22:

```bash
ss -tln | grep :22
```

## 4. First Connection (Password Authentication)

From Kali:

```bash
ssh sia@192.168.56.20
```

## 5. SSH Key-Based Authentication

Generating an ED25519 key pair on Kali:

```bash
ssh-keygen -t ed25519 -C "sia-homelab"
```

Copying the public key to the Ubuntu server:

```bash
ssh-copy-id sia@192.168.56.20
```

Verifying the key was installed correctly:

```bash
cat ~/.ssh/authorized_keys
```

Testing key-based login with verbose output (used heavily during troubleshooting, see `lessons-learned.md`):

```bash
ssh -v sia@192.168.56.20
ssh -vvv sia@192.168.56.20
```

Expected success indicator in the verbose output:

```
debug1: Authenticated to 192.168.56.20 ([192.168.56.20]:22) using "publickey".
```

## 6. SSH Hardening

Editing the server configuration:

```bash
sudo nano /etc/ssh/sshd_config
```

Key directives set:

```
PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin without-password
PermitEmptyPasswords no
```

Validating the configuration syntax before restarting the service (critical step, catches typos before they lock you out):

```bash
sudo sshd -t
```

Applying the new configuration:

```bash
sudo systemctl restart ssh
```

## 7. Firewall Configuration (UFW)

Setting default policies:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allowing SSH explicitly:

```bash
sudo ufw allow OpenSSH
```

Enabling the firewall:

```bash
sudo ufw enable
```

Checking the resulting ruleset:

```bash
sudo ufw status verbose
```

## 8. Final Verification (from Kali)

Confirming the host is reachable and only the intended service is exposed:

```bash
nmap 192.168.56.20
```

## 9. Snapshot

Taking a VirtualBox snapshot of both VMs after the lab was completed and verified, to preserve a known-good baseline before starting Lab 02:

```
VirtualBox Manager -> select VM -> Snapshots tab -> Take
Snapshot name: Lab-01-Completed
```
