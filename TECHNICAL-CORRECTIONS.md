# Technical review and corrected explanations

Reviewed: **4 October 2026**. Scope: the early learning PDFs indexed in this repository, plus the Day-01 and Day-02 folder placeholders. This review used the supplied report copies, extracted text and selected original screenshots. It is a documentation review, **not a fresh execution of the VMs**. Later local Wazuh reports are outside this repository's current index.

**Read these corrections alongside the original PDFs. Where explanations conflict, this document supersedes the old interpretation.** Original screenshots remain historical evidence; they have not been edited to manufacture a result. The review and wording were prepared with AI assistance.

## Priority fixes

| Area | Correct interpretation | Validation status |
|---|---|---|
| UFW | Correct source-restricted rule syntax; a successful allowed request does not prove every other source is blocked | Syntax corrected; negative test still required |
| SSH | Encrypted session payload is not a readable wrong password; SSH identification banners can be plaintext | Explanation corrected; selected capture reviewed |
| Linux identity | Local `su` is user switching, not proof of lateral movement | Lab classification corrected |
| Password storage | Hashing differs from reversible encryption; stolen hashes can be guessed offline | Explanation corrected |
| Detection | Failed logins and accepted logins require context; neither alone proves compromise | Conclusions narrowed; correlation retest required |
| Cloud identity | Roles are not inherently administrators; CloudTrail coverage and session expiry have limits | Conceptual claims corrected |

## Report-by-report corrections

Page numbers refer to PDF pages, starting at 1. Day numbers below follow **filenames**, not conflicting headings inside the reports.

### Day-01 and Day-02 folders

The remote folders contain one-line README placeholders, not complete project reports. Day-01 says lab environment setup; Day-02 says network visibility. These are navigation entries, not evidence of completed validation. The detailed network-visibility PDF is at the repository root. A separate local file named `NETWROK VISIBILITY.pdf` is not treated as a published Day-01 report.

### Network visibility — Day 2

Report: `NETWORK VISIBILITY day 2.pdf`.

- Packet capture observes traffic available to the selected interface and capture position. It does not automatically expose every packet on the whole virtual network. Promiscuous mode cannot make an ordinary switched port see all other hosts' traffic.
- A successful ping demonstrates ICMP reachability during that test. It does not prove HTTP/SSH access, application health, or complete network security.
- The section labelled listening-service enumeration actually discusses Wireshark installation. A separate listener check such as `sudo ss -lntup` is needed to support that claim.
- Installing capture tools and setting capture permissions is environment preparation, not attacker privilege escalation.

Retest: record host/interface/IP mappings, capture location, a bounded ping, and separate service/listener checks. Only claim visibility for the traffic actually captured.

### Firewall hardening — Day 3

Report: `Traffic Control & Firewall Hardening day 3.pdf`, pp. 1, 3, 7–11.

- Page 1: `-p 22,80` means ports **22 and 80**, not 20 and 80. `-sS` is a TCP SYN scan; it can be detected and is not a guarantee of stealth. Use lowercase `nmap`; raw SYN scanning normally needs suitable privileges.
- An open port establishes an exposed listening service from that scan location. It is not, by itself, proof of a vulnerability. `filtered` means Nmap could not determine whether the port was open because probes or responses were filtered; use host rules/logs and before/after tests to attribute the effect to UFW. [Nmap port states](https://nmap.org/book/man-port-scanning-basics.html).
- Page 7 has `from` twice. The corrected lab rule is:

```bash
sudo ufw allow from 10.0.0.1 to any port 80 proto tcp
sudo ufw status numbered
```

This is an example for the recorded lab address, not a command to apply blindly to another server. Preserve administrative access before enabling a firewall, particularly over SSH. Examine existing broad allow rules, order, defaults and IPv6. [UFW manual: syntax and remote management](https://manpages.ubuntu.com/manpages/noble/man8/ufw.8.html).

- Pages 8–11: a successful `curl` from Kali supports allowed-source access only. Replace the conclusion with: **“HTTP was reachable from the recorded allowed source; denial from another source remains to be tested.”**

Retest: use a second authorised lab client not covered by the allow rule. Save both clients' IPs, UFW configuration, HTTP results and matching timestamps. Do not claim everyone else is blocked until the negative control is recorded.

### Fail2Ban — filename Day 4

Report: `Automated Server Defense with Fail2Ban   day 4   .pdf`. Its internal heading says Day 5; that numbering mismatch is editorial.

- `maxretry` is interpreted with `findtime` and the matching filter. Record both, plus `bantime`, backend and ban action. Three SSH commands do not necessarily equal three matched authentication failures: a connection can contain multiple password prompts.
- A `systemd` backend is appropriate when reading the relevant journal. It is not a universal Ubuntu-version fix; confirm the actual logging source and journal match. Prefer a small `.local` override over copying and maintaining the whole packaged configuration. [Fail2Ban configuration manual](https://github.com/fail2ban/fail2ban/blob/master/man/jail.conf.5).
- A timeout or refusal alone does not prove a Fail2Ban ban. Combine the jail's banned-IP output, matching Ban/Unban logs, and client behaviour for the same address and time. A failed login simulation does not mean the attacker took control.

Retest: capture the effective jail settings, measured failed events within the window, ban evidence, and access restoration after expiry/unban. Keep this restricted to the isolated lab.

### Authentication-log analysis — filename Day 5

Report: `Log Analysis & Incident Response Operations  day 5  .pdf`. Its internal heading says Day 4; use the topic and filename to avoid confusing the two exercises.

- Listing `/var/log` does not prove that all security events are collected. Verify a known test event appears in the configured destination. `/var/log/auth.log` depends on logging configuration; a journal-based system can require journal inspection instead.
- Repeated failed passwords are evidence of authentication failures. In this controlled exercise the test is known; outside the lab investigate source, target user, frequency, time and legitimate activity before calling it malicious brute force.
- `Invalid user` records an attempted nonexistent account, not proof of deliberate username enumeration. A source IP identifies a network endpoint, not a person.
- `grep -c` counts matching lines in the supplied file, not automatically distinct attempts in the test window. Scope by host/source/time and account for rotation. Reading an existing log is not live monitoring; `tail -f` follows newly appended lines.

### Protocol inspection — Day 6

Report: `Network Traffic Analysis  Protocol Verification  Day 6.pdf`, pp. 2–3, 6–8, 10–11.

- Use `wireshark --version` with two ASCII hyphens. Ctrl+C stops the foreground process; it does not log out or refresh group membership. Capture permissions are separate from running the whole GUI as root.
- HTTP without TLS can expose content in the capture. SSH protects session payload after key exchange; the SSH transport protocol runs over TCP. Initial protocol identification banners and some negotiation are observable. Encryption is not caused by entering the wrong password. [SSH transport specification](https://www.rfc-editor.org/info/rfc4253/).
- The selected screenshot on p. 7 shows an SSH protocol identification string. It does **not** reveal the entered password or establish that the selected frame contains it. Replace the explanation with: **“The capture shows SSH traffic and visible protocol metadata; it does not let me read the password from the encrypted session.”**
- Encrypted traffic still exposes metadata such as endpoints, timing and packet sizes. Capture metadata alone does not prove an attack, a successful login, or the decrypted activity.
- `any` refers to interfaces available on the capturing host. Port 80 is a filter by port, not a guarantee that the traffic is HTTP. Likewise, a port-22 filter is not authentication evidence.

Retest: capture HTTP content and SSH identification/key-exchange/encrypted phases separately, then match SSH authentication outcomes to server logs. A ping or port check alone cannot establish application security.

### Detection and response — Day 7

Report: `incident Response Detection Correlation and Automated Defense  day 7.pdf`, pp. 1–11.

- A scan measures the resulting port states; it does not guarantee that 22 and 80 are open. `systemctl enable` configures startup on boot; `start` starts a unit now. For checking a TCP listener use `sudo ss -lntp 'sport = :80'` rather than a loose text match.
- UFW applies configured packet rules, not an automatic judgement of suspicious intent. Logging level, rate limits and logging destination affect what is recorded. Enabling logs after a test cannot recover earlier unlogged events. [UFW logging](https://manpages.ubuntu.com/manpages/noble/man8/ufw.8.html).
- Use an explicit capture expression:

```bash
sudo tcpdump -ni any 'tcp and (port 22 or port 80)'
```

- The SSH loop generates test connections with interactive prompts, not a measured number of failures or a successful takeover. Ban attribution needs the Fail2Ban evidence described above.
- Correct correlation explanation: **“Match source, destination, host identity and time across packet, authentication, firewall and ban records to test a specific sequence of events.”** Merely using multiple tools is not a implemented SIEM correlation rule.

### Local identity and lateral-movement awareness — Day 8

Report: `Detecting Lateral Movement and Privilege Escalation day 8 .pdf`, pp. 5–14.

- `su - username` on one machine is local identity switching. It does not establish lateral movement. A valid lab description is **“local user-switching and remote authentication awareness.”** Lateral movement concerns accessing other systems after a foothold; a separate cross-host scenario and evidence would be needed. [MITRE lateral movement](https://attack.mitre.org/tactics/TA0008/).
- Failure to switch/log in is a result to diagnose, not automatic proof that a security control prevented an attacker. Wrong credentials, DNS, routing, service state and policy are different failure causes.
- `/etc/passwd` inventories local file-based accounts; it does not include every possible external identity source. Group membership alone does not fully describe effective authority.
- The p. 11–14 definitions confuse hashing with reversible encryption. Password verification uses password hashes; disclosure enables offline guessing and is still sensitive. Do not publish `/etc/shadow` contents as proof. [Password hashing function](https://man7.org/linux/man-pages/man3/crypt.3.html).
- The duplicated command tables and conversational assistant text are drafting artefacts, not extra experiments or an approved operational SOP.

Retest: document who switched identity, whether this was authorised, host name, authentication method and matching logs. Do not label normal administrator activity an exploited escalation.

### Privilege awareness — Day 9

Report: `Privilege Escalation Detection Labs  day 9  .pdf`, pp. 3–6, 8–15.

- `NOPASSWD` can increase risk, but assess allowed commands, arguments and user-controlled inputs. It is not automatically a universal root key.
- `find / -perm -4000 -type f` inventories files with the set-user-ID bit. SUID uses the executable owner's identity when honoured; that owner is not necessarily root. Legitimate SUID files are not automatically backdoors. Inventory alone is not a completed vulnerability assessment. Suppressed permission errors mean the result may be incomplete. [Linux execution credentials](https://man7.org/linux/man-pages/man2/execve.2.html).
- Inspect effective sudo permissions and included policy files; reading only `/etc/sudoers` cannot prove the absence of all privilege risks. `sudo visudo -c` checks syntax, not least privilege or absence of compromise.
- Failed root SSH authentication is not proof of privilege escalation. An authorised `sudo` operation is also not evidence of an exploit. Match success/failure to actual permissions and intent.
- A SIEM detects/alerts according to configured rules. Blocking requires a configured response or human action. Fail2Ban blocks matching sources under its jail settings; it does not prevent every escalation path.

### Linux access-control audit — Day 10

Report: `IAM Audits day 10.pdf`, pp. 2–4, 8–10, 14–16.

- This is a Linux identity/access-control exercise, not evidence of an AWS IAM deployment. A newly observed account requires comparison with approved changes; it does not automatically prove compromise.
- `sudo groups` commonly reports the target root user's groups. To audit a named account use `id <username>`, `getent group sudo` and that account's effective `sudo -l` output. `getent passwd` consults configured NSS sources; completeness still depends on the directory backend's enumeration support.
- `chmod 600` gives owner read/write and removes group/other mode permissions. It is not encryption and does not protect against an attacker acting as the owner or with sufficient privilege. `644` permits other users to read; `700` adds owner execute, which is usually unnecessary for an ordinary text file. [chmod modes](https://man7.org/linux/man-pages/man1/chmod.1.html).
- Page 14 labels Kali as `10.0.0.2`, conflicting with the earlier `.1` Kali / `.2` Ubuntu mapping. Treat attribution as **unresolved until matched to that run's host/interface records**; do not silently relabel an ambiguous screenshot.
- A `grep` of existing failed-password records is retrospective analysis, not by itself real-time detection or proven malicious attribution.

### Cloud IAM concepts — Day 11

Report: `Cloud IAM Foundations & Detection Mindset Day 11.pdf`.

- IAM is an essential control, not the entirety of cloud security. Network, workload, data protection and monitoring still matter.
- An IAM role is an assumable identity whose policies define permissions. It can have narrowly scoped read access; it is not inherently an administrator. Temporary credentials are not guaranteed to expire “very quickly”; duration varies.
- Removing a user or the ability to assume a role is not a blanket guarantee that an already issued role session immediately stops working. Assess session expiry, current permissions and explicit revocation/deny controls. [AWS role-session revocation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_revoke-sessions.html).
- Long-lived access keys are primarily at risk through disclosure/misuse, not ordinary guessing. Prefer federation for people and suitable roles for workloads; this PDF is conceptual learning, not a completed cloud implementation.
- CloudTrail is not a recording of every console click. Coverage depends on event category, service and configuration; data-event logging requires separate configuration. [CloudTrail management events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-management-events-with-cloudtrail.html).
- Impossible travel and privileged activity are investigation signals, not automatic proof of compromise. VPN use, approved administration and other context matter.

### Manual correlation concepts — Day 12

Report: `SIEM Correlation day 12.pdf`, pp. 2–7.

- Firewalls do not necessarily log every attempt. High frequency suggests automation but does not independently prove malicious intent.
- `Accepted password` proves that authentication succeeded, not that the system was hacked. Success can also use public-key authentication. Failures followed by success must be matched by host, source, identity and an explicit time window, then checked against authorised activity.
- “Ten failures plus success” is a proposed triage heuristic, not a validated detection threshold. A single failure is not always harmless. The report's “imagine” exercise does not demonstrate a deployed SIEM correlation rule.
- Replace the fixed High confidence/incident conclusion with: **“Authentication sequence requiring triage; compromise, confidence and severity depend on matched evidence and impact.”** Root involvement can increase impact, but does not automatically establish an incident.
- Blocking, credential changes and escalation follow an approved playbook and evidence assessment. A fixed 24–48-hour observation period cannot certify a system clean.

## Retest checklist — not completed by this review

1. Record VM names, OS/tool versions, network mode, interface addresses and timezone. Preserve originals and redact secrets before sharing.
2. Repeat UFW allowed and denied tests from two authorised lab clients against the same destination and rule set.
3. Record effective Fail2Ban threshold/window, matching failed events, Ban entry, actual banned IP and restoration evidence.
4. Capture HTTP and SSH separately; distinguish visible banners/metadata from encrypted application payload. Match authentication outcomes to the server log.
5. Audit a named Linux account's effective rights. Treat account switching, authorised administration and exploited escalation as different results.
6. For correlation, define source/user/host fields, time window and threshold; test a positive sequence and benign negative controls. Save rule/configuration and matched raw events before claiming automated detection.
7. Publish each retest as a dated supplement with expected versus observed results. Keep unresolved items labelled unresolved.

## Evidence standard for future reports

Use: **question → configuration → test → matching evidence → interpretation → limitation**. Separate what the screenshot shows from what the narrative assumes. Screenshots establish a captured state; they do not replace missing negative tests, prove a production deployment, or demonstrate an unperformed exploit.
