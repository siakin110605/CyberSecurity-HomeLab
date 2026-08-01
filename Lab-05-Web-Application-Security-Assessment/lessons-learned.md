# Lessons Learned: Lab 05

## A disabled service is not a removed service

This lesson from Lab 02 came back for real during this lab's own Phase 0. CUPS and Avahi, both explicitly disabled in Lab 02, were listening again before this lab's technical work even started, despite `systemctl is-enabled` correctly still reporting them as disabled. Something at runtime, almost certainly a routine package upgrade, restarted them independent of their boot configuration. Disabling a service controls what happens at boot; it does not guarantee what happens for the rest of that service's life. The fix this time was to purge the packages outright rather than disable them again, since disabling had already proven insufficient once.

## A UI restriction is not a security control

DVWA's Medium security level replaced a free-text SQL injection field with a dropdown limited to five numeric values. That looked like a fix. It was not one: the dropdown is enforced entirely in the browser, and the actual HTTP request is just `?id=<value>`, which can be set to anything by editing the URL directly, bypassing the dropdown completely. What actually stopped the injection at Medium level was server-side input escaping, verified by testing whether the exact same payload worked when submitted directly, not by trusting that a restricted `<select>` element meant anything on its own.

## A firewall's status output is not proof of actual exposure

`ufw status verbose` said only port 22 was allowed in. Nmap, run independently from Kali, said port 80 was open. Both were technically correct; they were just describing different layers. Docker inserts its own accept rules directly into iptables' `FORWARD` chain, ahead of UFW's own chains, so a published container port bypasses UFW entirely regardless of what UFW's ruleset says. Reading a firewall's own configuration is not the same as verifying what is actually reachable, and this lab, like every lab before it, only trusted the second kind of evidence.

## A security setting stored in a cookie belongs to the attacker too

The most important finding in this lab was not the SQL injection or the command injection, both of which are well-known, expected DVWA exercises. It was realizing that DVWA's security level itself was tracked in a plain client-supplied cookie, not in server-side session state. That meant the "Medium" protection tested and confirmed working earlier in the lab could be switched off entirely from the client side, using Burp Suite's Match and Replace to rewrite the cookie on every request, without ever touching the actual settings page. The general lesson generalizes past this one lab: any security-relevant decision that reads its input from something the client controls, a cookie, a hidden field, a header, is a decision the client can also make on the attacker's behalf.

## Precision in write-ups matters as much as the finding itself

It would have been easy to write Finding 6 up as "authentication bypass," which sounds more dramatic. It is not accurate: login and session identity were never touched. What was actually demonstrated is a security-level enforcement bypass through insecure design, a narrower and more precisely true claim. Overstating a finding's scope in a report is exactly the kind of thing that damages credibility with anyone technical enough to check the claim against the evidence, and getting this distinction right in the README mattered more than making the finding sound bigger than it was.

## What I would do differently

I would start the Burp Suite capture before doing any manual testing, not partway through, so the full sequence (baseline request, Low-level exploitation, Medium-level mitigation, cookie tampering) would exist as one continuous, provable timeline in a single Burp project rather than being reconstructed from separate screenshots taken across sessions.
