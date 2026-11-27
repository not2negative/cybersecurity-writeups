# Cybersecurity Writeups & Pentesting Labs

![Kali Linux](https://img.shields.io/badge/Kali%20Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)
![Hack The Box](https://img.shields.io/badge/Hack%20The%20Box-9FEF00?style=for-the-badge&logo=hackthebox&logoColor=black)

Technical writeups, vulnerability root-cause analyses, and remediation guides for authorized lab environments and CTF challenges.

---

## Repository Structure

<!-- REPOMAP:START -->
<p align="center">
  <a href="https://github.com/not2negative/cybersecurity-writeups"><img src="https://github-tree-widget.vercel.app/api/svg?user=not2negative&repo=cybersecurity-writeups&row=0&v=2262ec" width="100%" /></a><br/>
  <a href="https://github.com/not2negative/cybersecurity-writeups/tree/main/CTFs"><img src="https://github-tree-widget.vercel.app/api/svg?user=not2negative&repo=cybersecurity-writeups&row=1&v=2262ec" width="100%" /></a><br/>
  <a href="https://github.com/not2negative/cybersecurity-writeups/tree/main/CTFs/UMBRAL%20CTF%20CYVIAN"><img src="https://github-tree-widget.vercel.app/api/svg?user=not2negative&repo=cybersecurity-writeups&row=2&v=2262ec" width="100%" /></a><br/>
  <a href="https://github.com/not2negative/cybersecurity-writeups/blob/main/CTFs/UMBRAL%20CTF%20CYVIAN/mis_facturas.md"><img src="https://github-tree-widget.vercel.app/api/svg?user=not2negative&repo=cybersecurity-writeups&row=3&v=2262ec" width="100%" /></a><br/>
  <a href="https://github.com/not2negative/cybersecurity-writeups/blob/main/CTFs/UMBRAL%20CTF%20CYVIAN/portal_de_empleados.md"><img src="https://github-tree-widget.vercel.app/api/svg?user=not2negative&repo=cybersecurity-writeups&row=4&v=2262ec" width="100%" /></a><br/>
  <a href="https://github.com/not2negative/cybersecurity-writeups/tree/main/HTB"><img src="https://github-tree-widget.vercel.app/api/svg?user=not2negative&repo=cybersecurity-writeups&row=5&v=2262ec" width="100%" /></a><br/>
  <a href="https://github.com/not2negative/cybersecurity-writeups/blob/main/HTB/management_htb.md"><img src="https://github-tree-widget.vercel.app/api/svg?user=not2negative&repo=cybersecurity-writeups&row=6&v=2262ec" width="100%" /></a><br/>
  <a href="https://github.com/not2negative/cybersecurity-writeups/blob/main/README.md"><img src="https://github-tree-widget.vercel.app/api/svg?user=not2negative&repo=cybersecurity-writeups&row=7&v=2262ec" width="100%" /></a>
</p>
<!-- REPOMAP:END -->

<p align="center">
  <a href="https://not2negative.github.io/cybersecurity-writeups/"><strong>Open Interactive Explorer</strong></a>
</p>

---

## Writeups Index

### ![Hack The Box](https://img.shields.io/badge/-Hack_The_Box-141D2B?style=flat-square&logo=hackthebox&logoColor=9FEF00) Hack The Box

<!-- HTB-INDEX:START -->
| Machine | OS | Difficulty | Key Vectors & Technologies | Date |
| :--- | :---: | :---: | :--- | :---: |
| [**Management**](./HTB/management_htb.md) | Linux | Easy | `LDAP` `OpenAM (CVE-2026-33439)` `GLPI (libsodium)` `rdiff-backup` | Nov 2026 |
<!-- HTB-INDEX:END -->

### ![CTFs](https://img.shields.io/badge/-CTFs_%26_Events-141D2B?style=flat-square&logo=hackaday&logoColor=white) CTFs & Events

| Event / Challenge | Category | Format / Difficulty | Key Concepts & Notes | Date |
| :--- | :---: | :---: | :--- | :---: |
| [**Campo de Marte CTF — INACAP Valparaíso**](https://www.linkedin.com/feed/update/urn:li:activity:7496781759699763201/) | `Attack & Defense` | Live A/D | `Infrastructure Defense` `Vulnerability Analysis` `Network Security` | 21 Aug 2026 |
| *Campo de Marte CTF — USM Valparaíso (In progress)* | `Web` `Linux` `Crypto` | Unknown | *Upcoming* | 27 Nov 2026 |

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