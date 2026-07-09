# Lessons Learned: Lab 01

## On planning before building

The biggest difference between this lab and a random collection of commands was deciding the network design before touching VirtualBox. Knowing in advance that Kali would sit at 192.168.56.10 and Ubuntu at 192.168.56.20 on a dedicated internal network, with NAT reserved only for internet access, meant every later step (static IP assignment, SSH target, firewall rules, nmap target) had a clear, consistent reference point instead of chasing a DHCP lease that could change on reboot.

## On SSH hardening order of operations

The most valuable lesson from this lab came directly out of a mistake: I disabled password authentication before I had verified that key-based authentication was actually working end to end. That is backwards. The correct order is:

1. Generate and install the key.
2. Test key login while password authentication is still enabled.
3. Only disable password authentication once key login is confirmed working.

Doing it in the wrong order is how people lock themselves out of a remote box. On a local VM this cost me twenty minutes of debugging. On a real server without console access, it would have meant losing access entirely.

## On reading error messages literally

`sudo sshd -t` returned a config error pointing at a specific line number and the exact malformed option name. My first instinct was to assume the problem was something more complex (wrong permissions, wrong key), when the actual cause was a single mistyped character in `sshd_config`. This reinforced a habit I want to keep: validate configuration syntax with the tool built for that purpose before assuming the problem is somewhere else, and read the error text before theorizing.

## On verbose debugging as a default, not a last resort

`ssh -v` and `ssh -vvv` turned out to be the single most useful diagnostic tool in this lab. Instead of guessing whether the failure was network, service, permissions, or key related, the verbose output showed exactly which key was being offered, whether the server accepted it, and where in the authentication sequence things succeeded or failed. I plan to reach for verbose mode earlier next time instead of treating it as a last resort.

## On permissions as a recurring root cause

Several of the checks I ran (`.ssh` directory permissions, `authorized_keys` ownership) exist because SSH silently refuses to use key files that are too permissive. Nothing in the visible error output points directly at file permissions, which makes this an easy thing to overlook if you don't already know to check for it.

## On firewall rules as documentation

Writing an explicit `ufw allow OpenSSH` rule, on top of a default-deny policy, does two things at once: it secures the box, and it documents intent. Anyone looking at `ufw status verbose` later can immediately see that SSH is the only service meant to be reachable, without needing to read every line of `sshd_config` to figure that out.

## Going into Lab 02

This lab was about proving that two machines can talk to each other securely over a controlled network. Lab 02 shifts the focus from "can they connect" to "how hardened is the Ubuntu box itself" (users, sudo policy, auditd, kernel parameters, unattended upgrades). The static IPs, the SSH key trust relationship, and the UFW baseline established here carry forward unchanged into every lab that follows.
