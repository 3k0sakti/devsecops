# Tutorial: SAST dengan Semgrep untuk PHP Vulnerable Web Application

## 📋 Daftar Isi
1. [Pendahuluan](#pendahuluan)
2. [Instalasi Semgrep](#instalasi-semgrep)
3. [Scanning Dasar](#scanning-dasar)
4. [Scanning dengan Rules Spesifik](#scanning-dengan-rules-spesifik)
5. [Analisis Hasil](#analisis-hasil)
6. [Generate Report](#generate-report)
7. [Integrasi CI/CD](#integrasi-cicd)

---

## 🎯 Pendahuluan

**SAST (Static Application Security Testing)** adalah metode analisis keamanan yang memeriksa source code tanpa mengeksekusinya. Semgrep adalah tool SAST yang:

- **Fast**: Analisis dalam hitungan detik
- **Open Source**: Gratis dan customizable
- **Multi-language**: Mendukung 30+ bahasa
- **CI/CD Ready**: Mudah diintegrasikan

### Kenapa Semgrep?
- ✅ Tidak perlu kompilasi
- ✅ Rules mudah dipahami (YAML)
- ✅ False positive rendah
- ✅ Komunitas aktif

---

## 🔧 Instalasi Semgrep

### Metode 1: Menggunakan pip (Recommended)
```bash
pip install semgrep
```

### Metode 2: Menggunakan Homebrew (macOS)
```bash
brew install semgrep
```

### Metode 3: Menggunakan Docker
```bash
docker pull returntocorp/semgrep
```

### Verifikasi Instalasi
```bash
semgrep --version
```

---

## 🚀 Scanning Dasar

### 1. Quick Scan dengan Auto Rules
Semgrep otomatis memilih rules berdasarkan bahasa yang terdeteksi:

```bash
semgrep --config=auto src/
```

### 2. Scan dengan Registry Rules
Menggunakan rules dari Semgrep Registry:

```bash
# Scan untuk PHP security issues
semgrep --config=p/php src/

# Scan untuk OWASP Top 10
semgrep --config=p/owasp-top-ten src/

# Scan untuk SQL Injection
semgrep --config=p/sql-injection src/

# Scan untuk XSS
semgrep --config=p/xss src/
```

### 3. Scan Spesifik File
```bash
semgrep --config=auto src/login.php
semgrep --config=auto src/config.php
```

---

## 🎯 Scanning dengan Rules Spesifik

### Custom Rules untuk PHP Vulnerabilities

Buat file `.semgrep/rules.yml`:

```yaml
rules:
  - id: php-sql-injection-raw-query
    message: "Possible SQL injection: user-controlled input used in SQL execution without prepared statements."
    languages: [php]
    severity: ERROR
    patterns:
      - pattern-either:
          - pattern: mysqli_query($CONN, $SQL)
          - pattern: mysql_query($SQL)
          - pattern: $DB->query($SQL)
          - pattern: $PDO->query($SQL)
      - pattern-inside: |
          $SQL = $ANY
      - metavariable-regex:
          metavariable: $ANY
          regex: "\\$_(GET|POST|REQUEST|COOKIE|FILES|SERVER)"
    metadata:
      cwe: "CWE-89"

  - id: php-sql-injection-direct-user
    message: "Direct use of superglobals in SQL string — possible injection."
    languages: [php]
    severity: ERROR
    pattern-either:
      - pattern: mysqli_query(..., "..." . $_GET[$X] . "...")
      - pattern: mysqli_query(..., "..." . $_POST[$X] . "...")
      - pattern: $DB->query("..." . $_REQUEST[$X] . "...")
    metadata:
      cwe: "CWE-89"

  - id: php-hardcoded-credentials
    message: "Hardcoded credential or secret detected — consider using environment variables or secret manager."
    languages: [php]
    severity: WARNING
    patterns:
      - pattern-either:
          - pattern: define("DB_PASS", $VAL)
          - pattern: define('DB_USER', $VAL)
          - pattern: $config['password'] = $VAL
          - pattern: $dbPassword = $VAL
          - pattern: $creds = ['password' => $VAL]
      - metavariable-regex:
          metavariable: $VAL
          regex: "^['\"][^'\"]{3,200}['\"]$"
    metadata:
      cwe: "CWE-798"

  - id: php-weak-session-settings
    message: "Insecure session cookie/config detected (cookie_httponly=false or cookie_secure=false)."
    languages: [php]
    severity: ERROR
    pattern-either:
      - pattern: ini_set('session.cookie_httponly', 0)
      - pattern: ini_set("session.cookie_secure", 0)
      - pattern: session_set_cookie_params(..., ['httponly' => false])
      - pattern: session_set_cookie_params(..., ['secure' => false])
    metadata:
      cwe: "CWE-614"

  - id: php-session-missing-secure-options
    message: "Session started without secure cookie flags — check cookie_secure, httponly, samesite."
    languages: [php]
    severity: WARNING
    patterns:
      - pattern: session_start()
      - pattern-not: ini_set('session.cookie_secure', 1)
      - pattern-not: ini_set('session.cookie_httponly', 1)
      - pattern-not: ini_set('session.cookie_samesite', "Strict")
    metadata:
      cwe: "CWE-614"
```

Jalankan dengan custom rules:
```bash
semgrep --config=.semgrep/rules.yml src/
```

---

## 📊 Generate Report

### 1. Output ke JSON
```bash
semgrep --config=auto src/ --json > semgrep-report.json
```

### 2. Output ke SARIF (untuk GitHub/GitLab)
```bash
semgrep --config=auto src/ --sarif > semgrep-report.sarif
```

### 3. Output ke HTML (lebih readable)
```bash
semgrep --config=auto src/ --json | \
  python3 -c "import json, sys; \
  data = json.load(sys.stdin); \
  print('<html><body><h1>Semgrep Report</h1><pre>', json.dumps(data, indent=2), '</pre></body></html>')" \
  > semgrep-report.html
```

### 4. Output dengan Severity Filtering
```bash
# Hanya tampilkan ERROR dan WARNING
semgrep --config=auto src/ --severity ERROR --severity WARNING
```

---

## 📈 Analisis Hasil

### Format Output

Semgrep menampilkan:
- **File path**: Lokasi file yang vulnerable
- **Line number**: Baris kode yang bermasalah
- **Rule ID**: ID rule yang triggered
- **Severity**: ERROR, WARNING, atau INFO
- **Message**: Deskripsi vulnerability
- **Code snippet**: Potongan kode vulnerable

### Contoh Output:
```
findings:
  - check_id: php.lang.security.sql-injection
    path: src/login.php
    start:
      line: 32
      col: 9
    end:
      line: 32
      col: 85
    severity: ERROR
    message: SQL injection detected
```

---

## 🔍 Scanning Complete dengan Berbagai Konfigurasi

### Command Lengkap untuk Analisis Mendalam

```bash
# 1. Scan full dengan semua rules PHP
semgrep --config=p/php \
        --config=p/owasp-top-ten \
        --config=p/security-audit \
        src/ \
        --json \
        --output=full-scan-report.json

# 2. Scan dengan metrics
semgrep --config=auto src/ --metrics

# 3. Scan dengan verbose output
semgrep --config=auto src/ --verbose

# 4. Scan dan exclude false positives
semgrep --config=auto src/ --exclude="*.min.php" --exclude="vendor/"
```

---

## 🔄 Integrasi CI/CD (tidak perlu dikerjakan)

### GitHub Actions
Buat file `.github/workflows/semgrep.yml`:

```yaml
name: Semgrep Security Scan

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  semgrep:
    name: SAST Scan
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Run Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/php
            p/owasp-top-ten
            p/security-audit
```

### GitLab CI
Tambahkan ke `.gitlab-ci.yml`:

```yaml
semgrep:
  image: returntocorp/semgrep
  script:
    - semgrep --config=auto src/ --json > semgrep-report.json
  artifacts:
    reports:
      sast: semgrep-report.json
```

### Docker Command
```bash
docker run --rm -v "${PWD}:/src" returntocorp/semgrep semgrep --config=auto /src
```

---

## 🎓 Best Practices

### 1. **Scan Secara Regular**
- Jalankan di setiap commit
- Integrasikan di CI/CD pipeline
- Schedule daily/weekly scans

### 2. **Progressive Remediation**
- Fix ERROR severity first
- Kemudian WARNING
- Review INFO findings

### 3. **Custom Rules**
- Buat rules spesifik untuk coding standards tim
- Document false positives
- Share rules dengan tim

### 4. **Baseline**
- Set baseline untuk legacy code
- Track progress perbaikan
- Focus on new code first

### 5. **Developer Training**
- Review findings bersama tim
- Understand why vulnerability matters
- Learn secure coding patterns

---

## 📝 Checklist SAST Implementation

- [ ] Install Semgrep
- [ ] Run initial scan
- [ ] Review all findings
- [ ] Create remediation plan
- [ ] Fix critical issues
- [ ] Setup CI/CD integration
- [ ] Configure notifications
- [ ] Train development team
- [ ] Document false positives
- [ ] Create custom rules
- [ ] Monitor trends over time

---

## 🔗 Resources

- **Semgrep Docs**: https://semgrep.dev/docs/
- **Semgrep Registry**: https://semgrep.dev/r
- **Playground**: https://semgrep.dev/playground
- **Community**: https://go.semgrep.dev/slack

---

## 📞 Support

Untuk pertanyaan atau issues:
- GitHub Issues: https://github.com/returntocorp/semgrep/issues
- Slack Community: https://go.semgrep.dev/slack
- Documentation: https://semgrep.dev/docs/

---

**Last Updated**: October 2025
**Author**: DevSecOps Tutorial
**License**: MIT
