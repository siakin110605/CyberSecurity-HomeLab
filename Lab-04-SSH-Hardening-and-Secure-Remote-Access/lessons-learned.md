# Lessons Learned: Lab 04

## Hardening a service means hardening several independent things, not one setting

Going in, "harden SSH" felt like it might come down to one or two config lines. It didn't. Authentication method, brute-force response, root access policy, and algorithm selection are four separate controls that each needed their own check, their own fix where needed, and their own verification. Fixing one doesn't imply the others are fine. Password authentication being disabled says nothing about whether root login is still reachable, and neither says anything about whether a brute-force attempt gets noticed at all.

## A tool saying something is configured is not the same as proof it works

Fail2Ban reporting `Active: active (running)` and a jail list containing `sshd` is a configuration fact. It is not evidence of protection. The only thing that actually proved the protection worked was running a real brute-force attempt from Kali and watching `fail2ban-client status sshd` show `Banned IP list: 192.168.56.10`, my own attacking machine, in the output. That distinction, "is it configured" versus "does it actually do the thing," is the same lesson Lab 03 taught about encrypted traffic, just applied to a different control.

## Reading past the first line of an error message

`A dependency job for ssh.service failed` sounds like a generic systemd complaint and could easily be misdiagnosed as a config syntax error. The real cause, a leftover `sshd` process from an earlier run still holding port 22, only showed up by going one layer deeper with `journalctl -xe` and then `systemctl status` on both `ssh.socket` and `ssh.service` separately. The first error message is a starting point for the investigation, not the diagnosis itself.

## Socket activation makes things more resilient, not immune

I expected systemd socket activation (`ssh.socket` handing connections to `ssh.service`) to make SSH restarts basically foolproof. It mostly does, but it assumes a clean handoff. An orphaned process outside that lifecycle can still sit on the port and break the whole chain. Understanding the mechanism well enough to know where it can still fail turned out to matter more than just knowing the mechanism exists.

## Independent tools catch different things than the tools you configured with

`ssh-audit`, run against the finished, hardened configuration, still flagged something: the `ecdh-sha2-nistp*` key exchange algorithms, over their disputed cryptographic provenance. Nothing in the hardening steps I applied would have caught that on its own, because none of those steps were looking for it. Running a second, independent tool against your own finished work is not redundant, it checks things your first tool wasn't built to check.

## Validating identity, not just configuration

The ED25519 host key fingerprint `ssh-audit` reported for the Ubuntu server matched, exactly, the fingerprint Kali recorded the very first time it connected to this host back in Lab 01. That was a small thing to notice, but it's a real security check: it confirms this is still the same host, not a different machine that happens to answer on the same IP. It also made the whole lab series feel connected in a way a fresh screenshot alone wouldn't have.

## What I'd do differently

I'd run the Lynis audit before installing Fail2Ban rather than in the same general block of work, and use its specific SSH-related recommendations as an explicit checklist item in Phase 3, instead of reviewing the SSH config from general knowledge first and only cross-referencing Lynis's output afterward. The order I actually used worked, but treating Lynis as the starting checklist rather than a parallel check would have made Phase 3 and Phase 4 feel less like two separate reviews of the same thing.
