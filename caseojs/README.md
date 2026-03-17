# Case 1: Vulnerability Assessment pada Open Journal Systems (OJS)

## Deskripsi Mata Kuliah
Mata Kuliah **DevSecOps** membekali mahasiswa dengan kemampuan mengintegrasikan praktik keamanan (Security) ke dalam siklus pengembangan dan operasional perangkat lunak (Development + Operations). Pada UTS ini, mahasiswa melakukan vulnerability assessment terhadap aplikasi nyata berbasis web yang dideploy di lingkungan VPS.

---

## Gambaran Umum Case 1

| Atribut | Detail |
|---|---|
| **Target Aplikasi** | Open Journal Systems (OJS) versi vulnerable |
| **Lingkungan** | VPS Ubuntu 22.04 LTS (disediakan oleh dosen) |
| **Metode** | SAST, DAST, Manual Review, OWASP Framework |
| **Output** | Laporan Vulnerability Assessment + Rekomendasi Mitigasi |
| **Bobot Nilai** | 40% dari total nilai UTS |

---

## Struktur Pertemuan

```
case-1/
├── 00-kickoff.md              → Pertemuan 1 : Kickoff, scope & pembagian tim
├── 01-attack-surface.md       → Pertemuan 2 : Pemetaan attack surface OJS
├── 02-scanning.md             → Pertemuan 3 : Scanning SAST & DAST
├── 03-owasp-analysis.md       → Pertemuan 4 : Analisis OWASP & risk scoring
├── 04-laporan-mitigasi.md     → Pertemuan 5 : Finalisasi laporan & mitigasi
└── templates/
    ├── vulnerability-report-template.md
    ├── risk-register-template.md
    └── rubrik-penilaian.md
```

---

## Timeline

| Pertemuan | Topik | Deliverable |
|---|---|---|
| 1 | Kickoff — Scope & Tim | Dokumen scope, pembagian peran tim |
| 2 | Pemetaan Attack Surface | Attack surface diagram, daftar entry point |
| 3 | Scanning SAST/DAST | Raw output tool (Nikto, OWASP ZAP, Semgrep) |
| 4 | Analisis OWASP & Risk Scoring | Risk register dengan CVSS score |
| 5 | Finalisasi Laporan & Mitigasi | Laporan lengkap PDF + slide presentasi |

---

## Tools yang Digunakan

| Kategori | Tool |
|---|---|
| **DAST** | OWASP ZAP, Nikto, SQLMap, Gobuster |
| **SAST** | Semgrep, SonarQube (community), phpcs-security-audit |
| **Recon** | Nmap, WhatWeb, Wappalyzer |
| **Dokumentasi** | Markdown, draw.io (diagram), CVSS Calculator |

---

## Referensi

- [OJS Official Repository](https://github.com/pkp/ojs)
- [OWASP Top 10 2021](https://owasp.org/Top10/)
- [OWASP Testing Guide v4.2](https://owasp.org/www-project-web-security-testing-guide/)
- [CVSS v3.1 Calculator — NIST](https://nvd.nist.gov/vuln-metrics/cvss/v3-calculator)
- [CVE Database OJS](https://www.cvedetails.com/vendor/17894/Pkp.html)
