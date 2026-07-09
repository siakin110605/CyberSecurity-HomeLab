# Network Diagram: Lab 01

## Logical Topology

```
                              Internet
                                  |
                                  |
                            NAT Network
                                  |
                +-----------------+-----------------+
                |                                   |
        +---------------+                   +----------------+
        |     Kali       |                   |     Ubuntu     |
        |  Attack Box    |<----------------->|   SSH Server   |
        | 192.168.56.10  |   Internal Network |  192.168.56.20 |
        |  (CyberLab)    |     "CyberLab"      |   (CyberLab)   |
        +---------------+                   +----------------+
```

## Mermaid Version (renders natively on GitHub/GitLab)

```mermaid
graph TB
    INET["Internet"]
    NAT["NAT Network<br/>(VirtualBox Adapter 1)"]
    INT["Internal Network: CyberLab<br/>(VirtualBox Adapter 2)"]

    KALI["Kali Linux<br/>Attack / Client Box<br/>192.168.56.10"]
    UBUNTU["Ubuntu Server 24.04 LTS<br/>SSH Server (hardened)<br/>192.168.56.20"]

    INET --> NAT
    NAT --> KALI
    NAT --> UBUNTU
    KALI <-- "SSH (TCP/22)" --> INT
    UBUNTU <-- "SSH (TCP/22)" --> INT
    INT -.-> KALI
    INT -.-> UBUNTU
```

## Adapter Summary

| VM | Adapter 1 (NAT) | Adapter 2 (Internal Network) | Role |
|----|------------------|-------------------------------|------|
| Kali Linux | DHCP (internet access, updates) | 192.168.56.10/24 static | Attack / client box |
| Ubuntu Server | DHCP (internet access, updates) | 192.168.56.20/24 static | SSH server (target) |

Both VMs share the Internal Network named **CyberLab**, which is fully isolated from the host's physical LAN. NAT is used only so each VM can independently reach the internet for package updates; all lab traffic between the two machines happens exclusively over the internal network.
