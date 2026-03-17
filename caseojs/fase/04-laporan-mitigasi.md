# Pertemuan 5 — Finalisasi Laporan & Rekomendasi Mitigasi

## Tujuan Pembelajaran
Setelah pertemuan ini, mahasiswa mampu:
1. Menyusun laporan vulnerability assessment yang profesional dan terstruktur
2. Merumuskan rekomendasi mitigasi yang konkret dan dapat diimplementasikan
3. Mempresentasikan temuan kepada pemangku kepentingan non-teknis
4. Memvalidasi efektivitas mitigasi (patch verification)

---

## 1. Struktur Laporan Vulnerability Assessment

Laporan final harus mengikuti struktur berikut:

```
Laporan Vulnerability Assessment — OJS
│
├── 1. Executive Summary (1–2 halaman)
│   ├── Latar belakang pengujian
│   ├── Ringkasan temuan (daftar kritis)
│   └── Kesimpulan & urgensi tindakan
│
├── 2. Metodologi (1 halaman)
│   ├── Scope & batasan
│   ├── Tools yang digunakan
│   └── Timeline pengujian
│
├── 3. Temuan Detail (isi utama)
│   └── Per temuan: deskripsi, bukti, CVSS, dampak, rekomendasi
│
├── 4. Risk Register & Summary
│   ├── Tabel risk register lengkap  
│   └── Risk matrix visual
│
├── 5. Rekomendasi Mitigasi
│   ├── Short-term (0–7 hari)
│   ├── Medium-term (7–30 hari)
│   └── Long-term (30–90 hari)
│
├── 6. Patch Verification (opsional/bonus)
│   └── Bukti sebelum & sesudah perbaikan
│
└── Lampiran
    ├── Raw output tools
    ├── Screenshot bukti
    └── Daftar referensi
```

---

## 2. Panduan Penulisan Executive Summary

Executive Summary ditulis untuk **pembaca non-teknis** (Rektor, Dekan, atau manajemen universitas).

### Template Executive Summary

```markdown
## Executive Summary

### Latar Belakang
[Nama Tim] melaksanakan vulnerability assessment terhadap aplikasi 
Open Journal Systems (OJS) yang dioperasikan oleh [nama institusi/jurnal]
pada periode [tanggal mulai] hingga [tanggal selesai]. 

Pengujian dilakukan atas dasar persetujuan tertulis dari [pemilik sistem]
dengan tujuan mengidentifikasi kerentanan keamanan sebelum ditangani oleh
aktor jahat.

### Ringkasan Temuan

Dari total [N] kerentanan yang ditemukan:

| Tingkat | Jumlah | Contoh Temuan |
|---------|--------|---------------|
| 🔴 Critical | X | SQL Injection pada fitur pencarian |
| 🟠 High    | X | SSRF melalui pengaturan jurnal |
| 🟡 Medium  | X | Stored XSS, IDOR pada API |
| 🟢 Low     | X | Informasi versi terekspos |

### Risiko Bisnis Utama

1. **Kebocoran data reviewer dan naskah belum-terbit** — SQL Injection 
   dapat memberi akses penuh ke database, membahayakan integritas proses 
   peer-review dan kepercayaan penulis.

2. **Pengambilalihan akun** — XSS yang berhasil dapat mencuri session 
   token Editor atau Administrator, memberikan akses penuh ke sistem.

3. **Kompromi infrastruktur** — SSRF memungkinkan akses ke layanan 
   internal yang tidak seharusnya dapat diakses dari internet.

### Rekomendasi Segera

Kami merekomendasikan tiga tindakan yang harus dilakukan dalam **72 jam**:
1. Upgrade OJS ke versi terbaru (≥ 3.3.0-16)
2. Nonaktifkan fitur upload plugin dari antarmuka web
3. Aktifkan HTTPS dan perbarui konfigurasi HTTP security headers
```

---

## 3. Penulisan Temuan Detail

Setiap temuan harus ditulis mengikuti format standar berikut:

### Template Temuan Detail

```markdown
---

## [VUL-XXX] Nama Kerentanan

### Informasi Umum

| Field | Nilai |
|---|---|
| **ID** | VUL-XXX |
| **Nama** | [nama kerentanan] |
| **Kategori OWASP** | A0X — [nama kategori] |
| **CWE** | CWE-XXX |
| **CVE** | CVE-XXXX-XXXXX (jika ada) |
| **CVSS v3.1 Vector** | CVSS:3.1/AV:X/AC:X/PR:X/UI:X/S:X/C:X/I:X/A:X |
| **CVSS Base Score** | X.X ([Rating]) |
| **Tanggal Ditemukan** | [DD/MM/YYYY] |
| **Ditemukan Oleh** | [Nama anggota tim] |

---

### Deskripsi

[Jelaskan kerentanan secara teknis namun dapat dipahami. Apa vulnerability-nya,
mengapa terjadi, dan di mana lokasinya dalam aplikasi.]

### Dampak Bisnis

[Jelaskan konsekuensi jika kerentanan dieksploitasi: apa data yang bisa 
diakses/dimodifikasi, siapa yang terdampak, dan apa implikasinya bagi 
organisasi.]

### Langkah Reproduksi (Proof of Concept)

**Prasyarat:**
- [kondisi yang diperlukan, misalnya: akun dengan peran Author]

**Langkah-langkah:**
1. [Langkah pertama]
2. [Langkah kedua, dst.]

**Payload/Tool yang Digunakan:**
\`\`\`
[kode, command, atau request HTTP]
\`\`\`

**Bukti (Screenshot):**
![Deskripsi screenshot](../screenshots/VUL-XXX-proof.png)

**Response/Output yang Diperoleh:**
\`\`\`
[output atau response]
\`\`\`

---

### Rekomendasi Mitigasi

#### Jangka Pendek (0–7 hari)
- [Tindakan darurat/workaround yang bisa dilakukan segera]

#### Jangka Menengah (7–30 hari)
- [Perbaikan permanen pada konfigurasi atau patch]

#### Jangka Panjang (> 30 hari)
- [Perubahan arsitektur atau kebijakan keamanan]

#### Contoh Kode Perbaikan (jika aplikatif)

**Sebelum (Vulnerable):**
\`\`\`php
// Kode yang rentan
\`\`\`

**Sesudah (Fixed):**
\`\`\`php
// Kode yang sudah diperbaiki
\`\`\`

### Referensi
- [link referensi CVE / OWASP / vendor advisory]
```

---

## 4. Rekomendasi Mitigasi per Temuan

### VUL-001: Stored XSS — Abstrak Artikel

#### Penyebab Root Cause
OJS tidak melakukan sanitasi output (`htmlspecialchars`) secara konsisten pada field abstrak sebelum ditampilkan di halaman editorial.

#### Mitigasi

**Jangka Pendek:**
```
- Nonaktifkan fitur rich-text editor yang memperbolehkan HTML mentah
  pada field abstrak (Settings → Workflow → Submission)
- Aktifkan Content Security Policy (CSP) header (lihat konfigurasi di bawah)
```

**Jangka Menengah:**
```php
// Tambahkan sanitasi output di template Smarty OJS
// File: templates/submission/form/step3.tpl
{$abstract|escape:"html"}  // ← pastikan semua variabel user-input menggunakan |escape

// Untuk Rich Text: gunakan DOMPurify di sisi client
<script src="dompurify.min.js"></script>
<script>
  var clean = DOMPurify.sanitize(dirtyInput, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p', 'br'],
    ALLOWED_ATTR: []
  });
</script>
```

**Konfigurasi CSP di Apache:**
```apache
Header always set Content-Security-Policy \
  "default-src 'self'; \
   script-src 'self' 'nonce-{RANDOM}'; \
   style-src 'self' 'unsafe-inline'; \
   img-src 'self' data:; \
   frame-ancestors 'none';"
```

---

### VUL-002: SQL Injection

#### Penyebab Root Cause
Query database dibuat dengan string concatenation tanpa menggunakan prepared statement.

#### Mitigasi

**Jangka Pendek:**
```
- Nonaktifkan akses publik ke endpoint bermasalah sementara (jika memungkinkan)
- Aktifkan WAF (ModSecurity) di level Apache
```

**ModSecurity untuk Apache:**
```bash
sudo apt install libapache2-mod-security2 -y
sudo cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf
sudo sed -i 's/SecRuleEngine DetectionOnly/SecRuleEngine On/' /etc/modsecurity/modsecurity.conf

# Aktifkan OWASP Core Rule Set
sudo apt install modsecurity-crs -y
sudo systemctl restart apache2
```

**Jangka Menengah — Perbaikan Kode:**
```php
// ❌ VULNERABLE — string concatenation
$result = $this->retrieve(
    "SELECT * FROM users WHERE username = '$username' AND password = '$password'"
);

// ✅ FIXED — Prepared statement (OJS DAO pattern)
$result = $this->retrieve(
    'SELECT * FROM users WHERE username = ? AND password = ?',
    [$username, $hashedPassword]
);
```

---

### VUL-003: SSRF — CVE-2021-27188

#### Mitigasi

**Jangka Pendek:**
```
- Upgrade OJS ke versi ≥ 3.3.0-9 yang sudah patch CVE-2021-27188
- Atau: nonaktifkan fitur custom stylesheet di pengaturan jurnal
```

**Jangka Menengah — Network Level:**
```bash
# Blokir outbound request dari server ke metadata endpoint
# (untuk VPS cloud seperti AWS/GCP/Azure)
sudo iptables -A OUTPUT -d 169.254.169.254 -j DROP
sudo iptables -A OUTPUT -d 10.0.0.0/8 -j DROP  # internal network

# Simpan rules
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

**Jangka Menengah — Validasi URL di kode:**
```php
// Validasi URL sebelum diproses server-side
function isAllowedUrl(string $url): bool {
    $parsed = parse_url($url);
    $host = $parsed['host'] ?? '';
    
    // Tolak IP private / loopback
    $resolvedIp = gethostbyname($host);
    $privateRanges = [
        '10.0.0.0/8', '172.16.0.0/12', '192.168.0.0/16',
        '127.0.0.0/8', '169.254.0.0/16', '::1/128'
    ];
    
    foreach ($privateRanges as $range) {
        if (Network::ipInCidr($resolvedIp, $range)) {
            return false;
        }
    }
    return true;
}
```

---

### VUL-004: Plugin Upload tanpa Validasi Integritas

#### Mitigasi

**Jangka Pendek:**
```
- Nonaktifkan fitur upload plugin via web UI (hanya izinkan via CLI)
- Tambahkan whitelist plugin yang diizinkan
```

**Konfigurasi OJS untuk disable plugin upload:**
```php
// File: config.inc.php
; Nonaktifkan instalasi plugin via gallery dan upload
installed_plugins_dir = "{$pkpLib}/plugins"
disable_install_plugins = On
```

**Jangka Menengah:**
```bash
# Verifikasi integritas plugin dari PKP:
# Download SHA256 checksum dari situs resmi PKP
sha256sum plugin-name.tar.gz
# Bandingkan dengan nilai yang tertera di situs PKP
```

---

### VUL-005–VUL-010: Konfigurasi & Hardening

#### Security Headers (Apache)
```apache
# /etc/apache2/conf-available/security-headers.conf

Header always set X-Content-Type-Options "nosniff"
Header always set X-Frame-Options "SAMEORIGIN"
Header always set X-XSS-Protection "1; mode=block"
Header always set Referrer-Policy "strict-origin-when-cross-origin"
Header always set Permissions-Policy "geolocation=(), microphone=(), camera=()"
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"

# Sembunyikan versi server
ServerTokens Prod
ServerSignature Off

# Nonaktifkan directory listing
Options -Indexes
```

#### Cookie Security
```php
// config.inc.php
session.cookie_httponly = On
session.cookie_secure = On
session.cookie_samesite = Lax
```

#### Rate Limiting Login (dengan fail2ban)
```bash
sudo apt install fail2ban -y

# Buat filter OJS
sudo nano /etc/fail2ban/filter.d/ojs-auth.conf
```

```ini
[Definition]
failregex = ^<HOST> .* "POST /ojs/index.php/index/login/signIn HTTP.*" 200
ignoreregex =
```

```bash
# Tambahkan jail
sudo nano /etc/fail2ban/jail.local
```

```ini
[ojs-auth]
enabled  = true
port     = http,https
filter   = ojs-auth
logpath  = /var/log/apache2/ojs_access.log
maxretry = 5
bantime  = 3600
findtime = 600
```

```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status ojs-auth
```

---

## 5. Patch Verification

Setelah mitigasi diimplementasikan, lakukan **verifikasi** untuk memastikan kerentanan sudah diperbaiki.

### Metode Verifikasi

| No | Temuan | Metode Verifikasi | Bukti yang Diperlukan |
|---|---|---|---|
| VUL-001 | Stored XSS | Re-test manual dengan payload sama | Screenshot: script tidak dieksekusi |
| VUL-002 | SQL Injection | Re-run SQLMap dengan parameter sama | Screenshot: "not vulnerable" |
| VUL-003 | SSRF | Re-test dengan URL internal | Screenshot: request diblokir |
| VUL-005 | Dir. Listing | Akses URL `/ojs/cache/` | Screenshot: 403 Forbidden |
| VUL-007 | Version info | `curl -I` terhadap URL target | Screenshot: header tidak menampilkan versi |

### Upgrade OJS ke Versi Terbaru

```bash
# Backup terlebih dahulu!
sudo mysqldump -u ojs_user -p ojs_db > ojs_db_backup_$(date +%Y%m%d).sql
sudo tar -czf ojs_files_backup_$(date +%Y%m%d).tar.gz /var/www/html/ojs/

# Download versi terbaru
cd /tmp
wget https://pkp.sfu.ca/ojs/download/ojs-3.4.0-7.tar.gz

# Jalankan upgrade via CLI
cd /var/www/html/ojs
php tools/upgrade.php upgrade
```

---

## 6. Panduan Slide Presentasi

Presentasi final (10–15 menit) harus mencakup slide berikut:

| Slide | Konten | Durasi |
|---|---|---|
| 1 | Cover: nama tim, nama proyek, tanggal | — |
| 2 | Executive Summary — ringkasan temuan | 1 menit |
| 3 | Scope & Metodologi | 1 menit |
| 4 | Attack Surface Diagram | 2 menit |
| 5 | Top 3 Temuan Kritis (demo jika memungkinkan) | 4 menit |
| 6 | Risk Matrix Visual | 2 menit |
| 7 | Rekomendasi Mitigasi Prioritas | 2 menit |
| 8 | Kesimpulan & Lesson Learned | 1 menit |
| 9 | Q&A | — |

### Tips Presentasi

- Gunakan **bahasa yang dapat dipahami non-teknis** pada slide 2, 7, 8
- Sertakan **demo live** atau rekaman video untuk 1–2 PoC terbaik
- Tampilkan **sebelum dan sesudah** untuk temuan yang sudah di-mitigasi
- Hindari tampilkan password atau data sensitif asli dalam slide

---

## 7. Deliverable Final Pertemuan 5

| No | Deliverable | Format | Bobot | Dikumpulkan Via |
|---|---|---|---|---|
| 1 | Laporan VA lengkap | PDF (min. 20 hal.) | 40% | LMS |
| 2 | Risk Register final | Excel / MD | 20% | GitHub |
| 3 | Slide presentasi | PPTX / PDF | 20% | LMS |
| 4 | Repository GitHub (tools output, screenshot) | ZIP / GitHub | 10% | GitHub |
| 5 | Video demo PoC (opsional/bonus) | MP4 ≤ 5 menit | +10% | LMS |

---

## 8. Checklist Laporan Final

### Kelengkapan Konten
- [ ] Executive Summary ditulis dari perspektif non-teknis
- [ ] Semua temuan memiliki CVSS score yang dihitung
- [ ] Semua temuan memiliki screenshot/bukti
- [ ] Rekomendasi mitigasi bersifat konkret dan actionable
- [ ] Risk register mencakup semua temuan
- [ ] Risk matrix visual disertakan

### Kualitas Laporan
- [ ] Tidak ada TYPO atau kesalahan ejaan signifikan
- [ ] Nomor halaman, tanggal, dan nama tim tertera
- [ ] Format konsisten (font, heading, tabel)
- [ ] Referensi dicantumkan untuk setiap CVE/CWE yang disebutkan
- [ ] Tidak ada data sensitif (password asli) yang terekspos dalam laporan

### Teknis
- [ ] Semua perintah/payload yang digunakan tercatat di lampiran
- [ ] Raw output tools tersimpan di repository
- [ ] Screenshot memiliki kualitas yang cukup untuk dibaca

---

## 9. Pertanyaan Diskusi Final

1. Jelaskan perbedaan antara **Fix** dan **Workaround** dalam konteks mitigasi kerentanan! Kapan workaround boleh digunakan?

2. Jika Anda menemukan SQL Injection yang memungkinkan akses ke database, apakah Anda wajib melaporkan ke dosen / pemilik sistem sebelum melanjutkan eksploitasi? Jelaskan dari sudut pandang etika dan hukum!

3. Mengapa **patch verification** penting dilakukan? Apa risiko jika mitigasi tidak diverifikasi?

4. Bagaimana cara menyampaikan temuan keamanan kepada rektor universitas yang tidak memiliki latar belakang teknis? Apa yang harus ditekankan?

---

## Referensi
- PTES (Penetration Testing Execution Standard) — Reporting
- OWASP Vulnerability Disclosure Cheat Sheet
- ModSecurity Reference Manual: https://github.com/owasp-modsecurity/ModSecurity
- fail2ban Documentation: https://www.fail2ban.org/wiki/index.php/Main_Page
- OJS PKP Security Advisories: https://pkp.sfu.ca/security-advisories/
- NIST SP 800-115: Technical Guide to Information Security Testing
