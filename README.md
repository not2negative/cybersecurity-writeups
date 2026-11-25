# Cybersecurity Writeups & Pentesting Labs

![Kali Linux](https://img.shields.io/badge/Kali%20Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)
![Hack The Box](https://img.shields.io/badge/Hack%20The%20Box-9FEF00?style=for-the-badge&logo=hackthebox&logoColor=black)

Technical writeups, vulnerability root-cause analyses, and remediation guides for authorized lab environments and CTF challenges.

---

## Repository Structure

[![map1](https://github-tree-widget.vercel.app/api/svg?user=not2negative&repo=cybersecurity-writeups)](https://github-tree-widget.vercel.app/?user=not2negative&repo=cybersecurity-writeups)
*[Open interactive explorer](https://not2negative.github.io/cybersecurity-writeups/)*

---

## Writeups Index

### ![HTB](https://img.shields.io/badge/-Hack%20The%20Box-141D2B?style=flat-square&logo=hackthebox&logoColor=9FEF00) Hack The Box

| Machine | OS | Difficulty | Key Vectors & Technologies | Date |
| :--- | :---: | :---: | :--- | :---: |
| [**Management**](./HTB/management.htb.md) | Linux | Easy | `LDAP` · `OpenAM (CVE-2026-33439)` · `GLPI (libsodium)` · `rdiff-backup` | Nov 2026 |

---

### ![CTF](https://img.shields.io/badge/-CTFs%20%26%20Eventos-141D2B?style=flat-square&logo=hackaday&logoColor=white) CTFs & Events

| Event / Challenge | Category | Difficulty | Concepts | Date |
| :--- | :---: | :---: | :--- | :---: |
| *USM CTF (In progress)* | `Web / Linux / Crypto` | — | *Upcoming* | Nov 2026 |

---

## Methodology for Writeups

- **Reconnaissance:** Port scanning, service enumeration, and attack surface mapping (`nmap`, fuzzing).
- **Enumeration:** Directory services (`LDAP`), web source inspection, and version fingerprinting.
- **Exploitation:** CVE root-cause analysis, initial access, and credential recovery.
- **Privilege Escalation:** Local permission auditing (`sudo -l`), binary analysis, and misconfigurations.
- **Remediation:** Concrete technical mitigations for each identified vulnerability.

---

> [!NOTE]
> All exercises documented in this repository were conducted strictly in authorized lab environments for educational purposes.