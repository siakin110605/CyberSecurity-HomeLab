# Lessons Learned: Lab 08

## A strong password policy does not stop a weak password choice

The Lab 07 hardening forced 14-character, complex passwords, and I believed that was the password problem solved. Then `svc-backup` fell to a spray on the very first realistic guess, `Welcome2025!!!`, which is 14 characters, has upper/lower/digit/symbol, and sails through the policy while being exactly the kind of thing a rushed admin types for a service account. The policy constrains the shape of a password, not its guessability. The spray also stayed under the five-attempt lockout threshold by design, so neither of Lab 07's two headline controls (length policy, lockout) stopped the foothold. The real gaps the policy cannot close are human choice and the structural blind spot of lockout against spraying, which is why detection had to carry the rest.

## Offline attacks live where none of my prevention controls can reach

AS-REP roasting and Kerberoasting both ended with a plaintext password on the attacker's machine, and the cracking happened entirely offline: no connection to the DC, no failed-logon events, no lockout, no firewall in the path. My instinct from the earlier labs was to reach for a prevention control, but there is nothing to lock or block once the attacker has the hash. The only things that actually help are making the secret uncrackable (a 25+ character random password or a gMSA) and detecting the one on-DC moment the attack touches, the ticket request (4768 with pre-auth type 0, or 4769 with RC4). That reframed detection for me: it is not a backup for when prevention fails, it is the primary control for a whole class of attacks prevention simply cannot see.

## An intrusion can be fully reconstructed from the Security log, if the logging was set up first

The investigation phase felt almost unfair in how much it recovered: the attacker's IP, every compromised account, each technique, and an ordered timeline, all from one log. But that was only possible because Lab 07 turned on the right audit subcategories before any of this happened. If I had tried to enable auditing during the incident, the evidence for everything already done would simply not exist. The detection work in Lab 07 and the IR work here are the same decision viewed from two different days, which is the entire argument for instrumenting an environment before you need it.

## Containment has to be verified from the attacker's side, not just the defender's

Disabling `svc-backup` showed `Enabled: False` on the DC, which looks like done. But the check that actually mattered was re-running the attacker's command from Kali and seeing `STATUS_ACCOUNT_DISABLED`. It is the same lesson as Lab 06's Fail2ban investigation: the defender's console saying a control is in place is not proof the attacker is actually locked out, and the only honest confirmation is to test from where the adversary stands.

## The whole series rhymes: "on" is not "effective," and honesty is the deliverable

Every lab in this series eventually ran into the same wall from a new direction. `auditd` was running with no rules (Lab 06). Fail2ban was active but silently catching nothing (Lab 06). Lockout was enabled but blind to spraying (Lab 07, confirmed live here). A password policy was enforced but satisfied by a guessable password (here). Each time, the thing that looked like security was not, until it was tested against the specific attack and the result written down honestly, including the misses. That habit, test the claim and record what is actually true, is the real output of this homelab, more than any single configuration.
