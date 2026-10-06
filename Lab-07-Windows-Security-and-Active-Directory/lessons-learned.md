# Lessons Learned: Lab 07

## A security control is only meaningful against a specific attack, never "in general"

Account lockout is the clearest example I have hit so far of a control that is simultaneously effective and ineffective, depending entirely on which attack you point it at. Against a brute-force, many passwords against one account, it works perfectly: I watched `jdoe` lock after exactly five bad guesses, and the DC logged it as Event 4740. Against a password spray, one password against many accounts, the very same control does nothing, because each account only ever sees one failed attempt and never reaches the threshold. `administrator` and `asmith` each took a failed logon and stayed open. If I had only run the brute-force test, I would have written "account lockout: effective" and been half wrong. The honest version is that lockout stops brute-force and is structurally blind to spraying, and the thing that actually catches spraying is detection, not prevention. This is the same shape as Lab 06's central lesson, that "enabled" is not "effective," seen from a new angle.

## The defense against the attack lockout cannot stop is detection, not a better lock

Once I accepted that lockout can never stop a careful spray (a 1-attempt-per-account spray is mathematically below any sane threshold), the follow-on was obvious but worth stating: the spray is not invisible, it is just not *blocked*. Every attempt still generates a failed-logon event. So the correct defense is a detection rule, a burst of 4625/4771/4776 across many distinct accounts in a short window, not a stricter lockout policy. That reframes account lockout and audit logging as covering two different attack classes rather than overlapping, which is why Phase 4 mattered as much as Phase 3.

## Not every default is wrong, and a credible assessment says which ones are right

It would have been easy to treat "it's a default, therefore it's weak" as a rule. It is not. The DC required SMB signing out of the box and had SMBv1 uninstalled, both genuinely good, and `Domain Admins` held only the built-in Administrator. Writing those down as verified-correct, instead of quietly skipping them to focus on the weaknesses, is part of what separates an assessment from a list of complaints. The real weaknesses (7-character minimum, lockout disabled) stood out more clearly precisely because the good defaults were acknowledged too.

## Install choices have consequences you only see later

Picking the edition without "Desktop Experience" silently produced a Server Core box with no GUI, which I only recognised when SConfig auto-launched instead of a desktop. And pressing a key at the post-reboot "boot from CD" prompt quietly restarted the whole installer. Neither threw an error; both just led somewhere I did not intend. The lesson is the same one the Fail2ban investigation taught in Lab 06: the absence of an error message is not the presence of success, and a wrong-but-silent outcome costs more time than a loud failure would have.

## Infrastructure constraints shaped the lab, and that is worth stating plainly

Running a Windows DC and Kali together on a 15.6 GB host meant real memory pressure: the guest froze more than once, and the lab only became stable after powering the Ubuntu VM off, closing host applications, and installing Guest Additions. The single-DC-plus-attacker topology (no domain-joined Windows client) is also why the LLMNR hardening could be applied but not demonstrated. Documenting that honestly, "applied, not tested, here is exactly why," is more useful than either omitting it or pretending it was exercised.
