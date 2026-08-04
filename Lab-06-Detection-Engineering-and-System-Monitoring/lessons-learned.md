# Lessons Learned: Lab 06

## "Active," "configured," and "alerting" are not synonyms for "effective"

This lab found the same underlying lesson three separate times, through three unrelated investigations, which is what convinced me it deserved to be the central theme of the whole lab rather than a one-off observation. `auditd` was `active (running)` with zero rules, providing zero coverage. Fail2ban's `sshd` jail was `active`, correctly pointed at the right log file, and had been since Lab 04, yet had never actually banned anything because of two independent, silent design behaviors (NOFAIL tagging and an IP whitelist) that a surface-level status check would never reveal. And an alert genuinely fired during the cookie-tampering replay, which looked like a working detection, but the alert had nothing to do with the actual attack mechanism being used. Three different words, "active," "configured," "alerting," each of which sounds like it should mean "this is working," and none of which actually do.

## An alert firing does not prove the detection saw the actual attack mechanism

This was the most uncomfortable finding to write up, because it would have been easy to present it as a clean success: "SQL injection alert fired during the cookie-tampering test." That framing is technically true and substantively misleading. The alert fired because a SQL injection pattern happened to be present in the URL, not because anything in the detection script understood or observed the cookie manipulation that was the actual point of the attack. Writing this up honestly meant going back into the script's own source code to confirm, definitively, that no cookie-parsing logic existed at all, rather than assuming the alert meant what it looked like it meant.

## A detection boundary does not automatically align with a virtualization boundary

I expected the command injection inside the DVWA container to be invisible to host-level auditing, the same way the Kali VM's activity is invisible to Ubuntu's own audit log. It was not invisible, because containers share the host's kernel rather than running one of their own. `execve` calls inside the container are still `execve` calls the host kernel executes, and host-level `auditd` watches the kernel, not a particular filesystem namespace. This turned out to be a real defensive advantage of this specific architecture, but it was not something I had reasoned through in advance; I assumed a container boundary would behave like the VM boundaries elsewhere in this homelab; it does not.

## Some gaps are structural, not fixable with a better regex

The Stored XSS miss was not a case of an imperfect pattern that needed refinement. Apache's access log format does not record POST request bodies at all, under any circumstances, so no regex against that log could ever have caught this payload, because the data it would need to match against was never captured by the log in the first place. Recognizing the difference between "my pattern is wrong" and "my data source cannot contain the answer" mattered more here than any specific regex skill, and is the kind of distinction that is easy to miss under the assumption that every detection gap is a tuning problem.

## Debugging a silent failure requires checking multiple independent layers, not stopping at the first plausible fix

Fail2ban showing `0/0` produced no error anywhere: not in `fail2ban.log`, not in the filter test output, nowhere. The first fix (a custom regex) was necessary but not sufficient, and retesting after it still showed `0/0` with the same silence. It would have been easy to conclude the whole approach was wrong and start over. Instead, treating the first fix's continued failure as new information (rather than proof of a bad approach) led to finding the actual second cause: an entirely separate, unrelated whitelist rule. Two root causes stacked on top of each other, and both had to be found before either fix mattered.

## Infrastructure constraints are legitimate obstacles worth documenting plainly

The Oracle Cloud attempt did not fail because of anything done wrong; it failed because a real cloud provider's free tier ran out of physical capacity in the only region available to this account, on every attempted shape configuration, across multiple availability domains. Documenting a real external constraint honestly, including the specific error message and the decision to pivot rather than keep retrying indefinitely, is a more useful record than pretending the lab was always going to use a lightweight custom script from the start. The pivot itself, from a full SIEM to a purpose-built script covering exactly this lab's three attack surfaces, produced a more precisely-scoped, better-understood detection stack than a general-purpose SIEM deployment would have, at a fraction of the resource footprint.
