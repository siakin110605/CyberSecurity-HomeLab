# Lab 04 Flow: Assessment to Hardening to Validation

Same two-VM topology as Labs 01-03 (Kali `192.168.56.10`, Ubuntu `192.168.56.20`). This lab adds no new infrastructure; it takes the one exposed service on the target, OpenSSH, and carries it through a complete security lifecycle.

```mermaid
graph TB
    A["Discover<br/>ping + nmap"] --> B["Enumerate<br/>auth methods, algorithms,<br/>version, vuln scan"]
    B --> C["Review Configuration<br/>sshd -T baseline"]
    C --> D["Audit<br/>Lynis system scan"]
    D --> E["Update<br/>apt update / upgrade"]
    E --> F["Deploy Fail2Ban<br/>brute-force protection"]
    F --> G["Simulate Attack<br/>repeated failed logins"]
    G --> H["Observe Ban<br/>Fail2Ban blocks the attacker IP"]
    H --> I["Deploy Key Auth<br/>ED25519 keypair"]
    I --> J["Harden<br/>disable password + root login"]
    J --> K["Validate<br/>independent ssh-audit review"]
```

Each stage depends on the one before it: hardening before the config is understood is guesswork, and disabling password authentication before key-based login is confirmed working is how you lock yourself out (a mistake already made once, and fixed, in Lab 01).
