# Commands Reference: Lab 03

All commands used, in the order they were executed, without much explanation. See `README.md` for context, risk/assessment/conclusion reasoning, and screenshots.

## Environment Confirmation

```bash
ip a
ip route
ip neigh
resolvectl status
```

## Host Discovery

```bash
sudo nmap -sn 192.168.56.0/24
```

## Port Scanning

```bash
sudo nmap -sS -Pn 192.168.56.20
```

## Service Detection

```bash
sudo nmap -sV -Pn 192.168.56.20
```

## SSH Algorithm Enumeration

```bash
sudo nmap -sV --script=banner,ssh2-enum-algos -Pn 192.168.56.20
```

## Full Port Range Scan

```bash
sudo nmap -p- 192.168.56.20
```

## Packet Capture (Wireshark)

```bash
sudo wireshark
```

Capture interface: `eth1` (internal network) for SSH analysis, `eth0` (NAT) for the outbound DNS/TLS capture.

Display filters used during analysis:

```
tcp
tcp.flags.syn == 1
tcp.stream eq 0
ssh
dns
tls
tls.handshake.type == 1
```

Wireshark menu actions used:

- Statistics -> Conversations (per-endpoint packet/byte counts)
- Statistics -> Endpoints
- Statistics -> Protocol Hierarchy
- Right-click packet -> Follow -> TCP Stream

## Traffic Generation (to produce packets for capture)

A DNS lookup (resolving `google.com`) and an outbound HTTPS request (to a host presenting `example.com` as SNI) were made while the capture was running, to generate the DNS and TLS traffic analyzed in the README.
