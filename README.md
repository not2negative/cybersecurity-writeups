# Cybersecurity Writeups & Pentesting Labs

![Kali Linux](https://img.shields.io/badge/Kali%20Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)
![Hack The Box](https://img.shields.io/badge/Hack%20The%20Box-9FEF00?style=for-the-badge&logo=hackthebox&logoColor=black)

En este repositorio almaceno mis writeups con documentación detallada, análisis de vulnerabilidades y guías de remediación de máquinas y retos de ciberseguridad resueltos de forma práctica.

---

## Mapa detallado del repositorio

```text
cybersecurity-writeups/
├── 📁 HTB/
│   └── 📄 [management.htb.md](./HTB/management.htb.md)        # OpenAM RCE (CVE-2026-33439), GLPI & rdiff-backup
├── 📁 CTFs/
│   └── 📁 USM_27_11/                         # Competencias y eventos
│       └── 📄 fuzzer
└── 📄 README.md                    # README Principal del repositorio
```

---

## Índice de Writeups

### ![Hack The Box](https://img.shields.io/badge/-Hack%20The%20Box-141D2B?style=flat-square&logo=hackthebox&logoColor=9FEF00) Hack The Box

| Máquina | Sistema | Dificultad | Vectores y Tecnologías Clave | Fecha |
| :--- | :---: | :---: | :--- | :---: |
| [**Management**](./HTB/management.htb.md) | ![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat-square&logo=linux&logoColor=black) | ![Easy](https://img.shields.io/badge/Easy-2ECC71?style=flat-square) | `LDAP` · `OpenAM (CVE-2026-33439)` · `GLPI (libsodium)` · `rdiff-backup` | Nov 2026 |

---

### ![CTF](https://img.shields.io/badge/-CTFs%20%26%20Eventos-141D2B?style=flat-square&logo=hackaday&logoColor=white) Competencias CTF

| Evento / Reto | Categoría | Dificultad | Conceptos Aplicados | Fecha |
| :--- | :---: | :---: | :--- | :---: |
| *Próximo evento — CTF USM* | `Web / Linux / Crypto` | — | *En preparación* | Nov 2026 |

---

## Metodología Aplicada Constantemente

Cada writeup sigue un flujo estructurado orientado tanto a la explotación como a la defensa:
**Reconocimiento y Escaneo:** Descubrimiento de puertos, servicios, subdominios y superficies de exposición (`nmap`, `fuzzing`).
**Enumeración Profunda:** Análisis de protocolos de directorio (`ldap`), código fuente web y *fingerprinting* de versiones.
**Explotación y Movimiento Lateral:** Análisis de causa raíz de vulnerabilidades (`CVEs`), obtención de sesiones interactivas y recuperación/descifrado de credenciales internas.
**Escalada de Privilegios:** Auditoría de permisos locales (`sudo -l`), binarios y configuraciones inseguras hasta obtener `root`.
**Remediación y Mitigación:** Recomendaciones técnicas concretas para cerrar cada vector identificado.

---

> [!NOTE]
> Todos los ejercicios documentados en este repositorio fueron realizados exclusivamente en entornos de laboratorio autorizados con fines educativos y de aprendizaje profesional.
