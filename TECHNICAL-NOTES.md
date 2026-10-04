# Lab methodology and technical reference

Practical reference for the Linux, network-security and access-control exercises in this repository.

## Network visibility

Packet captures provide evidence of traffic visible at the selected interface. Record the capture host, interface, source, destination and timestamps before interpreting a result. Use separate checks for network reachability, listening services and application behaviour.

```bash
ip -br address
sudo ss -lntup
sudo tcpdump -ni any 'tcp and (port 22 or port 80)'
```

HTTP without TLS can expose application content. SSH protects session payload after key exchange, while identification banners, endpoints and timing remain observable. Server authentication logs provide the outcome of login attempts. [SSH transport specification](https://www.rfc-editor.org/info/rfc4253/).

## Firewall policy

A source-restricted HTTP rule for the recorded isolated lab:

```bash
sudo ufw allow from 10.0.0.1 to any port 80 proto tcp
sudo ufw status numbered
```

Adapt addresses to the actual lab. Preserve management access before enabling a firewall. Evaluate default policy, existing rules, ordering and IPv6; validate allowed and denied requests from separate clients. [UFW manual](https://manpages.ubuntu.com/manpages/noble/man8/ufw.8.html).

An Nmap `open` result describes service reachability from the scanner. A `filtered` result indicates that filtering prevented a definitive open/closed result. Interpret scan results alongside configuration and matching host records. [Nmap port states](https://nmap.org/book/man-port-scanning-basics.html).

## Fail2Ban validation

Document the effective jail's filter, logging backend, `maxretry`, `findtime`, `bantime` and ban action. Count matched authentication failures within the configured window; a single SSH connection can contain several password prompts.

```bash
sudo fail2ban-client status sshd
sudo journalctl -u fail2ban --since '10 minutes ago'
```

Connect the matching failed events, banned source address, Ban/Unban record and client behaviour to the same test period. Select file or journal inspection according to the host's logging configuration. [Fail2Ban configuration reference](https://github.com/fail2ban/fail2ban/blob/master/man/jail.conf.5).

## Linux identity and permissions

Use a named account's effective rights when assessing access:

```bash
whoami
id
getent passwd
getent group sudo
sudo -l
sudo visudo -c
```

`getent` consults configured identity sources; enumeration depends on the backend. `sudo -l` describes effective sudo permissions, while `visudo -c` checks policy syntax. Account inventory should be compared with approved changes. [getent reference](https://man7.org/linux/man-pages/man1/getent.1.html).

| Control | Interpretation |
|---|---|
| File mode `600` | Owner read/write; no group/other mode permissions |
| File mode `644` | Owner read/write; group/other read |
| File mode `700` | Owner read/write/execute |
| SUID | When honoured, execution uses the executable owner's identity |
| `NOPASSWD` | Evaluate the exact permitted commands and inputs |

Permissions must be assessed with ownership and effective privilege. Password hashes support verification and require protection against disclosure and offline guessing. [File modes](https://man7.org/linux/man-pages/man1/chmod.1.html), [execution credentials](https://man7.org/linux/man-pages/man2/execve.2.html), [password hashing](https://man7.org/linux/man-pages/man3/crypt.3.html).

Local `su` exercises demonstrate identity switching. Cross-host lateral movement requires evidence of access to another system. Authorised administration, failed authentication and exploited privilege escalation are separate findings. [MITRE lateral movement](https://attack.mitre.org/tactics/TA0008/).

## Authentication investigation

Evaluate failed and successful authentication using the destination host, source, username, authentication method and time window. Confirm expected activity before assigning incident severity. A successful authentication event establishes a login outcome; compromise requires additional evidence.

Distinguish retrospective log searches from live monitoring and manual analysis from an implemented correlation rule. Validate a proposed detection with positive and benign negative cases before reporting its effectiveness.

## Cloud identity principles

Roles provide permissions defined by policy and are not inherently administrative. Temporary sessions have configured lifetimes; revocation and access-policy changes require explicit consideration. [AWS role-session revocation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_revoke-sessions.html).

CloudTrail coverage depends on event category and configuration. Management-event records and separately configured data events provide different visibility. [CloudTrail management events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-management-events-with-cloudtrail.html).

The Linux exercises establish access-control foundations. The cloud section is conceptual study; completed AWS practical case studies are presented separately in the portfolio.

## Documentation standard

Each case study should connect **objective → environment → configuration → test → evidence → result**. Record tool versions, host mappings, timestamps and expected versus observed behaviour. Keep credentials and password hashes out of public evidence.

Original PDFs form the learning archive. This reference provides the current technical interpretation. It consolidates documentation rather than recording a new execution of the VMs. Further validation priorities are source-restricted firewall negative tests, measured Fail2Ban thresholds, and matched authentication sequences with benign controls.
