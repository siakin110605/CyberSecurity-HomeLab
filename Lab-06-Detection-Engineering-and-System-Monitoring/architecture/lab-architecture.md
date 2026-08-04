# Architecture: Lab 06

Same physical topology as Labs 01-05. This lab adds no new machines; it adds detection capability on top of the existing Ubuntu host and replays attacks from Labs 03-05 to see whether that capability actually works.

```mermaid
graph TB
    KALI["Kali Linux: 192.168.56.10<br/>Attack replay source<br/>Nmap, Burp Suite"]
    UBUNTU["Ubuntu Server 24.04: 192.168.56.20"]
    AUDITD["auditd<br/>lab06-detection.rules<br/>identity, sudo, SSH config, exec_actions"]
    DOCKER["Docker<br/>dvwa container<br/>--log-driver syslog --tag dvwa-apache"]
    JOURNAL["systemd journal<br/>host-level log aggregation"]
    DETECT["detect.py<br/>watch_web / watch_ssh / watch_exec<br/>writes detections.log"]
    F2B["Fail2ban<br/>sshd jail"]

    KALI -- "replayed attacks" --> UBUNTU
    UBUNTU --> AUDITD
    UBUNTU --> DOCKER
    DOCKER -- "syslog driver" --> JOURNAL
    AUDITD --> JOURNAL
    JOURNAL --> DETECT
    UBUNTU --> F2B
```

## What changed from Lab 05

Labs 01-05 built and hardened this host, then assessed a web application running on it. This lab does not add new infrastructure; it adds visibility on top of what already exists: audit rules, a syslog-connected Docker log driver, a custom detection script, and a working Fail2ban jail (the jail existed since Lab 04, but was not actually catching anything until this lab's investigation, see Finding 7).
