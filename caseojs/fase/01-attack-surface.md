# Pertemuan 2 — Pemetaan Attack Surface OJS

## Tujuan Pembelajaran
Setelah pertemuan ini, mahasiswa mampu:
1. Mengidentifikasi semua entry point aplikasi OJS
2. Memetakan alur data (data flow) antar komponen OJS
3. Membuat attack surface diagram menggunakan model ancaman (threat model)
4. Mengklasifikasikan aset berdasarkan tingkat kritikal

---

## 1. Konsep Attack Surface

**Attack surface** adalah totalitas dari semua titik (surface) di mana penyerang yang tidak berwenang dapat mencoba memasukkan data atau mengekstrak data dari sistem.

Komponen attack surface terdiri dari:

```
Attack Surface = Entry Points + Data Stores + Trust Boundaries
```

### Kategori Attack Surface

| Kategori | Contoh pada OJS |
|---|---|
| **Network Attack Surface** | Port terbuka, protokol HTTP/HTTPS |
| **Software Attack Surface** | Form login, upload file, REST API |
| **Human Attack Surface** | Akun admin default, social engineering |
| **Third-party Attack Surface** | Plugin/tema pihak ketiga, library PHP |

---

## 2. Arsitektur OJS

### 2.1 Komponen Utama OJS

```
┌─────────────────────────────────────────────────────────────────┐
│                        PENGGUNA (Browser)                        │
└──────────────────────────┬──────────────────────────────────────┘
                           │ HTTP/HTTPS
┌──────────────────────────▼──────────────────────────────────────┐
│                    WEB SERVER (Apache/Nginx)                      │
│                  /var/www/html/ojs/index.php                      │
└──────────────────────────┬──────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                      APLIKASI OJS (PHP)                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  MVC Layer   │  │  Plugin Sys  │  │  REST API (v1/v2)    │  │
│  │  (Router,    │  │  (Hooks,     │  │  /api/v1/...         │  │
│  │  Controller, │  │   Generic    │  │                      │  │
│  │  Template)   │  │   Plugins)   │  │                      │  │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬───────────┘  │
│         └─────────────────┴──────────────────────┘              │
│                           │                                      │
│  ┌────────────────────────▼───────────────────────────────────┐ │
│  │                    DAL (Data Access Layer)                  │ │
│  └────────────────────────┬───────────────────────────────────┘ │
└───────────────────────────┼─────────────────────────────────────┘
                            │
        ┌───────────────────┴───────────────────┐
        │                                       │
┌───────▼────────┐                    ┌─────────▼────────┐
│  MySQL/MariaDB │                    │  File System     │
│  (ojs_db)      │                    │  /files/         │
│  - users       │                    │  /public/        │
│  - submissions │                    │  uploads, cache  │
│  - journals    │                    │                  │
└────────────────┘                    └──────────────────┘
```

### 2.2 Peran Pengguna (Trust Levels)

| Peran | Trust Level | Akses |
|---|---|---|
| **Guest / Tidak Login** | Paling rendah | Baca artikel publik |
| **Reader** | Rendah | Unduh artikel + akun profil |
| **Author** | Sedang | Submit naskah, upload file |
| **Reviewer** | Sedang | Review naskah, komentar |
| **Section Editor** | Tinggi | Manage submission, assign reviewer |
| **Editor** | Tinggi | Full editorial workflow |
| **Journal Manager** | Sangat tinggi | Konfigurasi jurnal |
| **Site Administrator** | Tertinggi | Full akses sistem |

---

## 3. Identifikasi Entry Points

### 3.1 Recon — Pengumpulan Informasi Awal

**Langkah 1 — Fingerprinting dengan WhatWeb**
```bash
whatweb http://<IP-VPS>/ojs/
```

Contoh output yang diharapkan:
```
http://<IP-VPS>/ojs/ [200 OK] Apache[2.4.52], 
  PHP[7.4.33], OJS[3.3.0-8], ...
```

**Langkah 2 — Port Scanning dengan Nmap**
```bash
nmap -sV -sC -p- --open <IP-VPS> -oN nmap_scan.txt
```

**Langkah 3 — Directory & File Enumeration**
```bash
# Gobuster — menemukan path tersembunyi
gobuster dir -u http://<IP-VPS>/ojs \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,txt,bak,old,zip \
  -o gobuster_output.txt

# Alternatif dengan ffuf
ffuf -w /usr/share/wordlists/dirb/common.txt \
  -u http://<IP-VPS>/ojs/FUZZ \
  -o ffuf_output.json -of json
```

**Langkah 4 — Identifikasi versi via HTTP Header**
```bash
curl -I http://<IP-VPS>/ojs/
curl -v http://<IP-VPS>/ojs/index.php/index/install 2>&1 | grep -i "x-powered\|server\|ojs"
```

---

### 3.2 Daftar Entry Points OJS

Berikut adalah entry point yang harus diidentifikasi dan didokumentasikan tim:

#### A. Authentication & Session

| No | URL / Endpoint | Method | Deskripsi | Risiko Potensial |
|---|---|---|---|---|
| A1 | `/index.php/index/login` | GET/POST | Form login utama | Brute force, credential stuffing |
| A2 | `/index.php/index/login/signIn` | POST | Proses autentikasi | SQL Injection, auth bypass |
| A3 | `/index.php/index/login/lostPassword` | POST | Reset password | Account takeover |
| A4 | `/index.php/index/register` | GET/POST | Registrasi user baru | Mass registration, spam |

#### B. File Upload

| No | URL / Endpoint | Method | Deskripsi | Risiko Potensial |
|---|---|---|---|---|
| B1 | `/index.php/$journal/submission/wizard` | POST | Submit naskah (multi-step) | Malicious file upload |
| B2 | `/index.php/$journal/api/v1/submissions` | POST | REST API submit | Unrestricted file upload |
| B3 | `/index.php/index/admin/settings` | POST | Upload logo/gambar | Path traversal |
| B4 | Plugin manager file upload | POST | Plugin `.tar.gz` | Remote Code Execution |

#### C. User Input / Reflected Data

| No | URL / Endpoint | Method | Deskripsi | Risiko Potensial |
|---|---|---|---|---|
| C1 | `/index.php/$journal/search` | GET | Pencarian artikel | Reflected XSS |
| C2 | `/index.php/$journal/issue/view/$id` | GET | Halaman issue | XSS via metadata |
| C3 | `/index.php/$journal/article/view/$id` | GET | Halaman artikel | XSS via abstract |
| C4 | Form profil user | POST | Edit profil | Stored XSS, HTML injection |

#### D. REST API

| No | Endpoint | Method | Deskripsi | Risiko Potensial |
|---|---|---|---|---|
| D1 | `/api/v1/users` | GET | Daftar user | IDOR, information disclosure |
| D2 | `/api/v1/submissions` | GET/POST | Manajemen submission | IDOR |
| D3 | `/api/v1/contexts` | GET | Daftar jurnal | Information disclosure |

#### E. Admin Panel

| No | URL / Endpoint | Method | Deskripsi | Risiko Potensial |
|---|---|---|---|---|
| E1 | `/index.php/index/admin` | GET | Dashboard admin | Admin access |
| E2 | `/index.php/index/admin/plugins` | GET/POST | Manajemen plugin | RCE via malicious plugin |
| E3 | `/index.php/index/admin/siteSettings` | POST | Pengaturan situs | SSRF, open redirect |
| E4 | `/index.php/index/admin/users` | GET/POST | Manajemen user | Privilege escalation |

---

## 4. Pemetaan Data Flow

### 4.1 Alur Submission Naskah (Berpotensi Vulnerable)

```
Author (Browser)
    │
    │ POST multipart/form-data (file upload)
    ▼
Apache → PHP OJS Controller (SubmissionHandler)
    │
    │ Validasi tipe file? ← [TITIK KRITIS: apakah hanya cek ekstensi?]
    ▼
File disimpan di /var/www/html/ojs/files/journals/1/articles/
    │
    │ Path disimpan di database
    ▼
MySQL: INSERT INTO submission_files ...
    │
    │ File dapat diakses via URL?
    ▼
http://<IP-VPS>/ojs/files/... ← [RISIKO: Direct Object Reference]
```

### 4.2 Alur Autentikasi

```
User → POST /login/signIn {username, password}
         │
         ▼
    UserDAO::getByUsername($username)  ← [SQL Injection vector?]
         │
         ▼
    password_verify($password, $hash)
         │
    ┌────┴────┐
  Gagal    Berhasil
    │         │
    ▼         ▼
 Error    SessionManager::createSession()
 msg           │
          Cookie SESSION dikirim
```

---

## 5. Identifikasi Aset Kritis

Klasifikasikan aset OJS berdasarkan CIA Triad:

| Aset | Confidentiality | Integrity | Availability | Nilai Kritis |
|---|---|---|---|---|
| Data login admin | Tinggi | Tinggi | Sedang | **Kritis** |
| Naskah unpublished | Tinggi | Tinggi | Sedang | **Kritis** |
| Data reviewer | Sedang | Tinggi | Sedang | **Tinggi** |
| Artikel published | Rendah | Tinggi | Tinggi | **Tinggi** |
| File konfigurasi (config.inc.php) | Tinggi | Tinggi | Sedang | **Kritis** |
| Database credentials | Tinggi | Tinggi | Rendah | **Kritis** |
| Log file server | Sedang | Sedang | Sedang | **Sedang** |

---

## 6. Threat Model — STRIDE

Gunakan kerangka **STRIDE** untuk memetakan ancaman:

| Threat | Singkatan | Contoh pada OJS | Entry Point |
|---|---|---|---|
| **S**poofing | Pemalsuan identitas | Login dengan akun orang lain | A1, A2 |
| **T**ampering | Modifikasi data | Edit metadata artikel via API | D2 |
| **R**epudiation | Penyangkalan tindakan | Hapus log aktivitas | E3 |
| **I**nformation Disclosure | Kebocoran info | Ekspos path file, error verbose | semua |
| **D**enial of Service | Gangguan layanan | Upload file besar berulang | B1-B4 |
| **E**levation of Privilege | Eskalasi hak akses | Author mengakses fitur Editor | D1, E4 |

---

## 7. Deliverable Pertemuan 2

| No | Deliverable | Format | Dikumpulkan Via |
|---|---|---|---|
| 1 | Attack surface diagram (arsitektur + entry points) | PNG/SVG (draw.io) | GitHub |
| 2 | Tabel entry points lengkap (isi semua kolom A–E) | MD / Excel | GitHub |
| 3 | Data flow diagram untuk 2 alur kritis | PNG/SVG | GitHub |
| 4 | Tabel aset kritis dengan penilaian CIA | MD | GitHub |
| 5 | Threat model matrix STRIDE | MD / tabel | GitHub |

---

## 8. Pertanyaan Diskusi

1. Bagaimana cara membedakan **attack surface** dan **attack vector**? Berikan contoh pada OJS!
2. Mengapa endpoint upload file (B1–B4) memiliki risiko lebih tinggi dibanding endpoint baca (GET)?
3. Pada alur autentikasi OJS, di titik mana kemungkinan terbesar terjadinya SQL Injection? Jelaskan!
4. Sebutkan minimal 3 informasi sensitif yang mungkin bocor melalui HTTP response headers OJS!

---

## Referensi
- OWASP Testing Guide v4.2 — OTG-INFO (Information Gathering)
- Microsoft STRIDE Threat Model
- OWASP Attack Surface Analysis Cheat Sheet
