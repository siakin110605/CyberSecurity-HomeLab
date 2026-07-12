# Attack Surface: Before and After

This lab hardens the same Ubuntu Server 24.04 VM built in [Lab 01](../../Lab-01-Infrastructure-and-Secure-Remote-Access/), reusing its static IP (`192.168.56.20`), internal network, and SSH key trust relationship. No new infrastructure was built; the scope of this lab is entirely the security posture of the existing host.

## Mermaid Version (renders natively on GitHub/GitLab)

```mermaid
graph LR
    subgraph Before["Before Hardening"]
        B1["SSH: 22/tcp<br/>open, password auth allowed"]
        B2["DNS stub resolver: 53<br/>loopback only"]
        B3["Avahi/mDNS: 5353<br/>udp, LAN-wide"]
        B4["CUPS: 631/tcp<br/>printing service"]
        B5["No audit logging"]
        B6["Default password aging<br/>(no expiry enforced)"]
    end

    subgraph After["After Hardening"]
        A1["SSH: 22/tcp<br/>open, key-only, X11Forwarding off"]
        A2["DNS stub resolver: 53<br/>loopback only, unchanged"]
        A3["Avahi/mDNS: 5353<br/>DISABLED"]
        A4["CUPS: 631/tcp<br/>DISABLED"]
        A5["auditd active<br/>watch rule on /etc/passwd"]
        A6["Password aging enforced<br/>max 90 / min 1 / warn 14"]
    end

    B1 -.hardened.-> A1
    B2 -.unchanged.-> A2
    B3 -.removed.-> A3
    B4 -.removed.-> A4
    B5 -.added.-> A5
    B6 -.enforced.-> A6
```

## Service Exposure Summary

| Port / Protocol | Service | Before | After | Notes |
|---|---|---|---|---|
| 22/tcp | SSH | Open | Open | Key-based auth only, carried from Lab 01; `X11Forwarding` explicitly disabled |
| 53/udp+tcp | systemd-resolved (DNS stub) | Loopback only | Loopback only | Not attacker-reachable in either state, left unchanged |
| 5353/udp | Avahi (mDNS/DNS-SD) | Open | **Disabled** | Zero-configuration discovery service, unnecessary on a hardened server |
| 631/tcp | CUPS | Open (localhost + `[::1]`) | **Disabled** | Printing service, no legitimate use on a headless server |

Removing Avahi and CUPS reduces the number of listening services from four to two, which directly shrinks the set of remotely reachable attack surfaces without affecting SSH access or DNS resolution.
