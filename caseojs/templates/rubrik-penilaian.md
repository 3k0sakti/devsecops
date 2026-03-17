# Rubrik Penilaian — Case 1: Vulnerability Assessment OJS

## Informasi Umum

| Field | Detail |
|---|---|
| **Mata Kuliah** | DevSecOps |
| **Total Bobot Case 1** | 100 poin (kelompok) + 20 poin (refleksi individu) |
| **Mode Pengerjaan** | Kelompok (4–5 mahasiswa) + Penilaian Individu |

### Komposisi Nilai Akhir Individu

```
Nilai Akhir Individu = (Nilai Kelompok × 80%) + (Nilai Refleksi × 20%)
```

| Komponen | Bobot | Keterangan |
|---|---|---|
| Nilai Kelompok | 80% | Sama untuk semua anggota tim |
| Nilai Refleksi (Peer Assessment) | 20% | Berbeda per individu — dinilai oleh rekan tim |

---

## Komponen Penilaian

### Pertemuan 1 — Kickoff & Setup (15 poin)

| Kriteria | Bobot | Skor Maks | Deskripsi |
|---|---|---|---|
| Dokumen scope & RoE | 5 | 5 | Lengkap, jelas, tertanda tangan semua anggota |
| Pembagian peran tim | 3 | 3 | Semua peran terisi, deskripsi tugas jelas |
| OJS berhasil diinstal di VPS | 7 | 7 | Screenshot versi OJS terlihat jelas, URL aktif |
| **Subtotal** | **15** | | |

**Skala penilaian setiap kriteria:**
- `5/5` atau `100%` → Sempurna, tidak ada kekurangan
- `3–4 / 5` atau `60–80%` → Baik, ada kekurangan minor
- `1–2 / 5` atau `20–40%` → Kurang, ada kekurangan signifikan
- `0 / 5` atau `0%` → Tidak dikerjakan atau tidak relevan

---

### Pertemuan 2 — Attack Surface Mapping (20 poin)

| Kriteria | Bobot | Deskripsi |
|---|---|---|
| Kelengkapan entry point table | 8 | Minimal 15 entry point teridentifikasi dengan benar |
| Kualitas attack surface diagram | 5 | Visual jelas, komponen lengkap, trust boundary ditandai |
| Data flow diagram | 4 | Minimal 2 DFD untuk alur kritis (login + submission) |
| Threat model STRIDE | 3 | Minimal 5 dari 6 kategori STRIDE diisi dengan contoh nyata |
| **Subtotal** | **20** | |

---

### Pertemuan 3 — Scanning SAST/DAST (25 poin)

| Kriteria | Bobot | Deskripsi |
|---|---|---|
| Nikto — output tersimpan & lengkap | 4 | Output ada, minimal 10 temuan dari Nikto |
| ZAP — scan unauthenticated | 5 | Report ZAP HTML tersedia, minimal 5 alert |
| ZAP — scan authenticated (min. 1 role) | 4 | Konfigurasi autentikasi berhasil |
| SQLMap — minimal 2 parameter | 4 | Output tersimpan, parameter yang diuji terdokumentasi |
| Semgrep SAST — output tersimpan | 4 | JSON output ada, minimal 5 temuan relevan |
| Kualitas dokumentasi temuan raw | 4 | Template diisi lengkap, screenshot ada |
| **Subtotal** | **25** | |

---

### Pertemuan 4 — Analisis OWASP & Risk Scoring (20 poin)

| Kriteria | Bobot | Deskripsi |
|---|---|---|
| Pemetaan OWASP Top 10 | 4 | Semua temuan terpetakan ke kategori yang tepat |
| Kalkulasi CVSS (min. 5 temuan) | 6 | Vector string benar, skor dihitung dengan tepat |
| Risk Register — kelengkapan | 5 | Semua kolom terisi, minimal 8 temuan |
| Risk Matrix visual | 3 | Visual jelas, posisi temuan akurat |
| Analisis narasi OWASP | 2 | Minimal 3 kategori OWASP dijelaskan secara naratif |
| **Subtotal** | **20** | |

---

### Pertemuan 5 — Laporan & Presentasi (20 poin)

| Kriteria | Bobot | Deskripsi |
|---|---|---|
| Kualitas Executive Summary | 4 | Dapat dipahami non-teknis, mencakup risiko utama |
| Kelengkapan laporan (format template) | 4 | Semua bagian template diisi lengkap |
| Kualitas rekomendasi mitigasi | 5 | Konkret, actionable, dibagi per timeframe |
| Kualitas presentasi & komunikasi | 4 | Jelas, terstruktur, menjawab pertanyaan dengan baik |
| Repository GitHub tersusun rapi | 3 | Folder terstruktur, README tersedia, file raw tools ada |
| **Subtotal** | **20** | |

---

## Penilaian Refleksi — Peer Assessment (20 poin individu)

Setiap mahasiswa **menilai rekan satu timnya** (bukan diri sendiri) menggunakan form refleksi.
Pengisian dilakukan secara **anonim** dan dikumpulkan **paling lambat H+1 setelah presentasi**.

### Mekanisme

1. Setiap mahasiswa mengisi **satu form per rekan tim** (misal: tim 5 orang → masing-masing mengisi 4 form)
2. Form dikumpulkan ke dosen secara rahasia (bukan ke repo GitHub)
3. Dosen menghitung rata-rata skor yang diterima masing-masing mahasiswa
4. Hasil peer assessment **tidak dibuka ke sesama anggota tim**

### Aspek Penilaian Refleksi

| Aspek | Bobot | Deskripsi |
|---|---|---|
| **Kontribusi Teknis** | 8 | Seberapa aktif rekan berkontribusi pada scanning, analisis, dan kode |
| **Kolaborasi & Komunikasi** | 5 | Responsif, membantu rekan, aktif diskusi |
| **Kualitas Deliverable Pribadi** | 4 | Bagian yang dikerjakan rekan memenuhi standar kualitas |
| **Kehadiran & Komitmen** | 3 | Hadir di semua pertemuan, memenuhi deadline internal tim |
| **Total** | **20** | |

### Skala Penilaian Tiap Aspek

| Skor | Deskripsi |
|---|---|
| 4 (Sangat Baik) | Selalu berkontribusi, melampaui ekspektasi |
| 3 (Baik) | Berkontribusi konsisten sesuai peran |
| 2 (Cukup) | Berkontribusi namun perlu diingatkan / tidak konsisten |
| 1 (Kurang) | Jarang berkontribusi, sering tidak hadir atau tidak responsif |
| 0 (Tidak Berkontribusi) | Tidak ada kontribusi yang dapat diidentifikasi |

> **Catatan Integritas:** Penilaian refleksi yang terbukti tidak jujur (misal: semua rekan diberi nilai maksimal tanpa dasar, atau sebaliknya diberi nilai 0 karena konflik personal) dapat dibatalkan oleh dosen. Isilah berdasarkan observasi nyata selama pengerjaan proyek.

---

## Bonus Poin (+10 poin)

| Bonus | Poin | Kriteria |
|---|---|---|
| Demo PoC live / video | +5 | Eksploitasi berhasil didemonstrasikan secara nyata |
| Patch verification | +3 | Bukti sebelum & sesudah mitigasi yang valid |
| Custom Semgrep rule | +2 | Minimal 1 custom rule yang relevan dengan celah OJS |

---

## Rubrik Nilai Akhir

### Nilai Kelompok

| Range Poin | Nilai Huruf | Deskripsi |
|---|---|---|
| 90 – 100 | A | Luar biasa — analisis mendalam, laporan profesional |
| 80 – 89 | AB | Sangat baik — sedikit kekurangan pada teknis atau laporan |
| 70 – 79 | B | Baik — semua deliverable ada namun kualitas perlu ditingkatkan |
| 60 – 69 | BC | Cukup — beberapa deliverable tidak lengkap |
| 50 – 59 | C | Kurang — sebagian besar deliverable tidak memenuhi standar |
| < 50 | D / E | Tidak lulus — deliverable utama tidak dikerjakan |

### Nilai Akhir Individu (dengan Refleksi)

| Range Nilai Akhir | Nilai Huruf |
|---|---|
| 90 – 100 | A |
| 80 – 89 | AB |
| 70 – 79 | B |
| 60 – 69 | BC |
| 50 – 59 | C |
| < 50 | D / E |

**Contoh perhitungan:**
```
Nilai kelompok       = 85 poin
Nilai refleksi diterima = 16/20 poin

Nilai Akhir Individu = (85 × 80%) + (16 × 20%  × 5)
                     = 68 + 16
                     = 84  →  AB
```
> Catatan: Nilai refleksi (0–20) dikonversi ke skala 0–100 dengan dikalikan 5, kemudian dikalikan bobot 20%.

---

## Kriteria Diskualifikasi (Nilai 0 untuk kriteria terkait)

| Kondisi | Sanksi |
|---|---|
| Terbukti melakukan plagiarisme (laporan persis sama dengan tim lain) | Nilai 0 untuk laporan |
| Menguji sistem di luar scope yang ditetapkan | Nilai 0 untuk seluruh case + sanksi akademis |
| Tidak ada kontribusi nyata dari satu atau lebih anggota | Anggota yang tidak berkontribusi nilai dikurangi 50% (diperkuat data refleksi rekan) |
| Tidak hadir presentasi tanpa izin | Nilai presentasi = 0 |

---

## Panduan Dosen — Penilaian Presentasi

Gunakan panduan berikut saat menilai presentasi:

| Aspek | Excellent (4) | Good (3) | Fair (2) | Poor (1) |
|---|---|---|---|---|
| **Penguasaan Materi** | Menjawab semua pertanyaan teknis dengan tepat | Menjawab sebagian besar pertanyaan | Menjawab pertanyaan dasar saja | Tidak dapat menjawab |
| **Kejelasan Penyampaian** | Terstruktur, transisi halus, durasi tepat | Terstruktur namun ada bagian kurang jelas | Cukup jelas namun beberapa bagian membingungkan | Tidak terstruktur |
| **Kualitas Visual Slide** | Profesional, konsisten, informatif | Baik namun ada inkonsistensi | Cukup, terlalu banyak teks atau berantakan | Tidak memadai |
| **Demo / Bukti** | Demo live berhasil + penjelasan detail | Demo berhasil namun penjelasan kurang | Video saja / screenshot | Tidak ada demo |

---

## Catatan Akhir untuk Mahasiswa

> **Ingat:** Tujuan utama case ini bukan hanya "mencari bug sebanyak mungkin",
> tetapi **memahami cara kerja kerentanan, dampaknya terhadap bisnis, dan 
> cara memitigasi dengan tepat**. Laporan yang analitik dan rekomendasi yang
> konkret dinilai lebih tinggi dari sekadar daftar temuan yang panjang.
>
> Praktikkan etika keamanan siber yang baik — dokumentasikan semua aktivitas,
> jangan eksploitasi data asli, dan laporkan temuan kritis segera kepada dosen.
>
> Untuk **refleksi**: isilah dengan jujur berdasarkan observasi selama proyek.
> Penilaian peer assessment dirancang untuk mendorong kontribusi yang adil,
> bukan untuk menjatuhkan rekan. Kejujuran Anda membantu dosen mengenali
> siapa yang benar-benar belajar dan berkontribusi.

---

## Form Peer Assessment

> Gunakan form di file [peer-assessment-form.md](./peer-assessment-form.md).
> Kumpulkan ke dosen secara langsung atau via email — **jangan** di-push ke repo GitHub.
