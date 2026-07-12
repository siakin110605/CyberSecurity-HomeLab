# Cyber Security Homelab

A self-directed, hands-on cybersecurity homelab built from scratch in VirtualBox: infrastructure, hardening, network security, web security, monitoring, and SOC-style analysis, each documented as a standalone lab with real command output, screenshots, and a written record of what actually went wrong and how it was diagnosed.

This is not a list of tutorials followed step by step. Each lab is designed, built, verified independently (not just assumed to work because a config file was written), and documented the way a real infrastructure or security change would be documented on the job.

## Why this exists

I created this homelab to deepen my practical cybersecurity skills through hands-on experimentation. My goal is to understand how secure systems are designed, configured, hardened, verified, and maintained while documenting the complete engineering process,from planning and implementation to troubleshooting and validation.Rather than simply following tutorials, this repository documents the reasoning behind every decision, the challenges encountered, and the lessons learned throughout the project.

## Lab Index

| Lab | Topic | Status |
|-----|-------|--------|
| [Lab 01](Lab-01-Infrastructure-and-Secure-Remote-Access/) | Secure Infrastructure & Remote Access | ✅ Complete |
| [Lab 02](Lab-02-Linux-Hardening/) | Enterprise Linux Hardening & Security Baseline | ✅ Complete |
| Lab 03 | Network Security & Traffic Analysis | 🟡 Planned |
| Lab 04 | Web Application Security Assessment | 🟡 Planned |
| Lab 05 | Detection Engineering & System Monitoring | 🟡 Planned |
| Lab 06 | Windows Security & Active Directory | 🟡 Planned |
| Lab 07 | Enterprise Attack Simulation & Incident Response | 🟡 Planned |

Each lab folder is self-contained and includes:

- `README.md`: a full technical write-up (overview, objectives, architecture, implementation, verification, problems encountered, lessons learned)
- `commands.md`: every command used, organized by phase
- `lessons-learned.md`: a reflective write-up of what the lab actually taught
- `screenshots/`: the evidence for every claim made in the README
- `architecture/`: network/architecture diagrams for that lab

## Environment

- **Hypervisor:** Oracle VirtualBox
- **Virtual machines:** Kali Linux (attack / client box), Ubuntu Server 24.04 LTS (hardening target)
- **Networking:** isolated internal network for all inter-VM lab traffic, NAT reserved only for internet/package access
- **Approach:** every configuration change is independently verified (`ping`, `ssh -v`, `nmap`, `ufw status`, etc.) rather than assumed to have taken effect

## License

See [`LICENSE`](LICENSE). All rights reserved; this repository is public for portfolio and evaluation purposes only.
