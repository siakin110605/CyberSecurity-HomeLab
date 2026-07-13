# Lessons Learned: Lab 03

## Host discovery is not a vulnerability assessment

It was tempting to treat "2 hosts found" as the end of the assessment. It isn't. Host discovery just answers "what's out there." Everything that actually matters, what's listening, what version, what cryptography, whether the exposure is intentional, comes after that first step, not from it. Confusing discovery with assessment would have made this a much shallower lab than it needed to be.

## An open port is not automatically a problem

Port 22 being open on the Ubuntu server isn't a finding to fix, it's the expected, intended state of a server that needs to be administered remotely. The interesting question was never "is a port open," it was "is the *right* port open, running the *right* software, with *no* other ports open alongside it." Learning to ask that more specific question, instead of just flagging every open port as a risk, felt like the real shift in thinking this lab produced.

## Correct interpretation matters as much as the scan itself

The SSH algorithm enumeration in Finding 4 could have been presented as a wall of algorithm names with no conclusion. What made it useful was going through the list and checking it against what's actually considered weak or deprecated (no `diffie-hellman-group1-sha1`, no `arcfour`, no `none` MAC), and being able to say clearly that nothing legacy was on offer. A scan output without that interpretation step is just noise with a timestamp.

## After SSH key exchange, the content is gone, not just hidden

I understood, abstractly, that "SSH is encrypted." Actually watching a live capture, seeing the plaintext banners and algorithm negotiation, and then watching the protocol column keep saying `SSHv2` for the rest of the 189 packets while the payload turned to unreadable ciphertext, made that concrete in a way reading about it never had. There is no visual marker in Wireshark that says "now it's encrypted." You have to know where the key exchange finishes to know where the visibility stops.

## Wireshark shows communication, not just open ports

Nmap answers "what's listening." Wireshark answers a different question: "how are these systems actually talking to each other, right now." Those turned out to be genuinely different skills. Reading a packet capture well means understanding protocol state (has the handshake finished, has encryption started), not just reading source and destination columns.

## What I'd do differently

I'd start the Wireshark capture before running any of the Nmap scans, rather than as a separate step afterward, so the scan traffic itself would also show up in the capture and could be compared against the "normal" SSH session traffic. That comparison, a scan's packet pattern versus a real interactive session's pattern, would have been a natural, low-effort addition to this lab and is a good candidate to fold into a future lab instead.
