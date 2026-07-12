# Lessons Learned: Lab 02

## On verifying at the same layer as the change

The most useful discipline in this lab was re-running the exact same enumeration command before and after a change, rather than trusting the tool that made the change. `systemctl disable cups` reports success whether or not the socket unit is also handled, so the only real proof that port 631 was actually closed was a fresh `ss -tuln` showing it gone. A `systemctl status` check confirms a unit's state; it does not confirm what is or is not reachable on the network. Those are different questions and need different tools.

## On socket activation as a hidden re-enable path

Disabling `cups.service` alone would not have been enough. Systemd socket activation means `cups.socket` can bring the service back the moment something connects to port 631, even with the service unit itself disabled. The same applies to `.path` units. Getting attack surface reduction right meant handling all three unit types for CUPS, not just the one with the obvious name.

## On policy changes having an immediate, not just future, effect

Applying the password aging policy with `chage -M 90 -m 1 -W 14` triggered an immediate forced password reset on the next `sudo` authentication, documented in the README's Problems Encountered section. I had assumed a policy change like this would only affect future password expiry, not the very next login. On a real system with actual users, that is the kind of thing that generates help desk tickets if it isn't communicated in advance. Lesson: read policy changes as "what happens the next time this account authenticates," not just "what happens N days from now."

## On detection rules needing to survive a reboot

The custom `auditd` rule (`auditctl -w /etc/passwd -p wa -k passwd_changes`) was added directly to the live kernel ruleset and verified working with a real triggered event. What it does not yet do is survive a reboot, since that requires writing the same rule into a file under `/etc/audit/rules.d/` and reloading with `augenrules --load`. The rule works today, but it is not yet durable. That gap is explicitly called out here rather than glossed over, and is the first item to close before treating this control as production-ready.

## On writing detection rules for something specific

`/etc/passwd` was chosen deliberately, not as a generic "watch some file" example. Any technique that creates, modifies, or hides a user account has to touch that file. A detection rule is only as useful as the story it tells about what it catches, and "unauthorized modification of the account database" is a story worth being able to tell.

## Going forward

The additional hardening tools that did not make it into this lab (Fail2Ban, AIDE, Lynis, `unattended-upgrades`, AppArmor, `sysctl` parameters) are intentionally deferred to a smaller, separate lab rather than being bolted onto this one. Making the audit rule persistent belongs there too. Keeping each lab focused on a coherent, explainable scope is worth more than one lab trying to cover everything at once.
