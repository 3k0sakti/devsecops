# Pertemuan 1 — Kickoff Case 1: Vulnerability OJS

## Tujuan Pembelajaran
Setelah pertemuan ini, mahasiswa mampu:
1. Mendefinisikan ruang lingkup (scope) vulnerability assessment secara profesional
2. Membentuk tim penguji dengan peran yang jelas
3. Menyiapkan lingkungan VPS dan menginstal OJS versi vulnerable
4. Memahami Rules of Engagement (RoE) dalam pengujian keamanan

---

## 1. Latar Belakang

**Open Journal Systems (OJS)** adalah platform manajemen dan penerbitan jurnal ilmiah open-source yang dikembangkan oleh Public Knowledge Project (PKP). OJS banyak digunakan oleh institusi akademik di seluruh dunia, termasuk perguruan tinggi di Indonesia.

Karena popularitasnya, OJS menjadi target menarik bagi penyerang. Beberapa versi OJS lama memiliki kerentanan kritis yang terdokumentasi di database CVE, antara lain:

| CVE | Versi Terdampak | Jenis Kerentanan | Severity |
|---|---|---|---|
| CVE-2020-28112 | < 3.2.1-2 | Stored XSS | High |
| CVE-2021-27188 | ≤ 3.3.0-6 | Server-Side Request Forgery (SSRF) | High |
| CVE-2022-24822 | ≤ 3.3.0-8 | Open Redirect + XSS | Medium |
| CVE-2023-28131 | < 3.3.0-12 | XSS via Editor | High |
| CVE-2020-12432 | 2.x | SQL Injection | Critical |

> **Catatan Etika:** Seluruh pengujian dilakukan di lingkungan yang telah disetujui (VPS milik kelas). Dilarang keras menguji sistem di luar scope yang ditetapkan.

---

## 2. Pembentukan Tim

Setiap kelompok terdiri dari **4–5 mahasiswa** dengan pembagian peran sebagai berikut:

| Peran | Jumlah | Tanggung Jawab |
|---|---|---|
| **Project Lead / Scrum Master** | 1 | Koordinasi jadwal, komunikasi, final review laporan |
| **Security Engineer (DAST)** | 1–2 | Menjalankan scanning aktif (ZAP, Nikto, SQLMap) |
| **Security Engineer (SAST)** | 1 | Analisis source code statis (Semgrep, phpcs) |
| **Documentation & Reporting** | 1 | Menulis laporan, risk register, rekomendasi mitigasi |

### Template Daftar Tim

```
Nama Tim       : ___________________________________
Kelas          : ___________________________________

| No | Nama | NIM | Peran |
|----|------|-----|-------|
| 1  |      |     |       |
| 2  |      |     |       |
| 3  |      |     |       |
| 4  |      |     |       |
| 5  |      |     |       |

Repositori Tim (GitHub) : ___________________________
```

---

## 3. Scope & Rules of Engagement (RoE)

### 3.1 In-Scope

- Aplikasi OJS yang berjalan di VPS yang diberikan dosen
- URL target: `http://<IP-VPS-TIM>/`
- Port yang diizinkan untuk diuji: 80, 443, 8080
- Semua fitur OJS yang dapat diakses via browser (sebagai guest, author, reviewer, editor)

### 3.2 Out-of-Scope

- Infrastruktur jaringan kampus
- VPS tim lain
- Server PKP (upstream OJS)
- Pengujian DoS / DDoS (dilarang keras)
- Eksploitasi di luar kerentanan yang ditemukan (pivoting ke server lain)

### 3.3 Rules of Engagement

| Aturan | Keterangan |
|---|---|
| Waktu pengujian | selama waktu penugasan |
| Metode | Non-destructive (jangan hapus/ubah data produksi) |
| Dokumentasi | Semua aktivitas harus dicatat (log, screenshot) |
| Pelaporan | Temuan kritis wajib dilaporkan ke dosen dalam 24 jam |
| Etika | Tidak membagikan akses VPS ke pihak luar |

---

## 4. Persiapan Lingkungan VPS

### 4.1 Spesifikasi VPS yang Diberikan

```
OS       : Ubuntu 22.04 LTS
CPU      : 2 vCPU
RAM      : 2 GB
Storage  : 20 GB SSD
Web      : Apache 2.4 / Nginx
Database : MySQL 8.0 / MariaDB 10.6
PHP      : PHP 7.4 (sengaja versi lama untuk reproduksi CVE)
```

### 4.2 Instalasi OJS Versi Vulnerable

> Target versi: **OJS 3.3.0-8** (mengandung CVE-2021-27188 dan CVE-2022-24822)

**Langkah 1 — Akses VPS via SSH**
```bash
ssh ubuntu@<IP-VPS>
```

**Langkah 2 — Instalasi dependensi**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y apache2 mysql-server php7.4 php7.4-cli \
  php7.4-common php7.4-mysql php7.4-xml php7.4-mbstring \
  php7.4-gd php7.4-curl php7.4-zip php7.4-intl unzip wget
```

**Langkah 3 — Unduh OJS versi vulnerable**
```bash
cd /var/www/html
sudo wget https://pkp.sfu.ca/ojs/download/ojs-3.3.0-8.tar.gz
sudo tar -xzf ojs-3.3.0-8.tar.gz
sudo mv ojs-3.3.0-8 ojs
sudo chown -R www-data:www-data ojs
sudo chmod -R 755 ojs
```

**Langkah 4 — Buat database MySQL**
```bash
sudo mysql -u root -e "
  CREATE DATABASE ojs_db CHARACTER SET utf8mb3;
  CREATE USER 'ojs_user'@'localhost' IDENTIFIED BY 'P@ssw0rd_OJS!';
  GRANT ALL PRIVILEGES ON ojs_db.* TO 'ojs_user'@'localhost';
  FLUSH PRIVILEGES;
"
```

**Langkah 5 — Konfigurasi Apache Virtual Host**
```bash
sudo nano /etc/apache2/sites-available/ojs.conf
```

Isi file konfigurasi:
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/ojs
    ServerName <IP-VPS>

    <Directory /var/www/html/ojs>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/ojs_error.log
    CustomLog ${APACHE_LOG_DIR}/ojs_access.log combined
</VirtualHost>
```

```bash
sudo a2ensite ojs.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
```

**Langkah 6 — Selesaikan instalasi via browser**
```
Buka browser → http://<IP-VPS>/ojs/index.php/index/install
Ikuti wizard instalasi dengan data berikut:
  - Database Host    : localhost
  - Database Name    : ojs_db
  - Database User    : ojs_user
  - Database Password: P@ssw0rd_OJS!
  - Admin Username   : admin
  - Admin Password   : Admin@123 (catat dengan baik!)
```

**Langkah 7 — Verifikasi instalasi**
```bash
# Cek versi OJS lewat terminal
curl -s http://localhost/ojs/index.php/index | grep -i "ojs\|version"

# Cek log Apache
sudo tail -f /var/log/apache2/ojs_access.log
```

---

## 5. Deliverable Pertemuan 1

| No | Deliverable | Format | Dikumpulkan Via |
|---|---|---|---|
| 1 | Dokumen Scope & RoE (ditandatangani tim) | PDF | LMS / GitHub |
| 2 | Daftar anggota tim + peran | MD / PDF | GitHub repo tim |
| 3 | Screenshot OJS berhasil diinstal (menampilkan versi) | PNG | GitHub repo tim |
| 4 | URL VPS aktif | Plain text | LMS |

---

## 6. Pertanyaan Diskusi

1. Mengapa penting mendefinisikan scope sebelum melakukan penetration testing?
2. Apa risiko yang mungkin terjadi jika pengujian dilakukan tanpa Rules of Engagement?
3. Jelaskan perbedaan antara **vulnerability assessment** dan **penetration testing**!
4. Mengapa OJS versi lama masih banyak dipakai di institusi akademik? Apa implikasinya terhadap keamanan?

---

## Referensi
- OWASP Testing Guide v4.2 — Chapter: Test Management
- PTES (Penetration Testing Execution Standard) — Pre-engagement Interactions
- PKP OJS Security Advisories: https://pkp.sfu.ca/ojs/
