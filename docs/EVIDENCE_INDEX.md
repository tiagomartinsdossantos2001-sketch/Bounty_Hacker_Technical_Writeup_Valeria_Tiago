# Evidence Register — Bounty Hacker

**Authors:** Tiago Santos & Valeria Mosquera Díaz  
**Collection date:** 12 November 2026  
**Target:** `10.82.180.192` (session-specific TryHackMe address)

The screenshots were captured during the authorized lab. Each item corresponds to a reproducible action, not a generic walkthrough example.

| ID | Suggested screenshot filename | What it establishes |
| --- | --- | --- |
| 01 | `01_nmap_scan.png` | Open ports 21/22/80, service versions and anonymous FTP detection |
| 02 | `02_ftp_anonymous.png` | Anonymous FTP login and the two visible files |
| 03 | `03_ftp_files.png` | `task.txt` signed by lin and contents of `locks.txt` |
| 04 | `04_hydra_ssh.png` | Hydra finding a valid SSH credential |
| 05 | `05_ssh_access.png` | Active SSH account `lin` and user flag file location |
| 06 | `06_user_flag.png` | User flag read from the SSH session |
| 07 | `07_privilege_escalation.png` | Permitted `sudo tar` and subsequent root shell |
| 08 | `08_root_flag.png` | `/root/root.txt` located and read as root |

**Note:** One separate screenshot labeled `Relevant` was excluded because it belonged to another TryHackMe room.

## Documentation

- [Technical Report (DOCX)](../Bounty_Hacker_Technical_Writeup_Valeria_Tiago.docx)
- [Walkthrough Presentation (PDF)](../TryHackMe%20Bounty%20Hacker%20Walkthrough.pdf)

The complete source PNG set can be uploaded under `screenshots/` with the names above. Until those PNG files are uploaded to this repository, refer to the screenshots embedded in the PDF presentation.

**Prepared by Tiago Santos & Valeria Mosquera Díaz.**
