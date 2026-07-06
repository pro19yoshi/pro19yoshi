# Analisis Mendalam Sistem Informasi Sekolah (SISFOKOL) v7.00
## Dokumen Audit dan Blueprint Migrasi ke Laravel 11

---

**Dokumen Ini Disusun Oleh:**  
Senior Business Analyst & Software Engineer  
AUDITOR SISTEM COBIT ITIL v4  
Kepala Sekolah & Guru - Konteks Pendidikan Indonesia  

**Tanggal Analisis:** Juli 2025  
**Versi Dokumen:** 1.0  
**Status:** DRAFT - Untuk Review  

---

## DAFTAR ISI

1. [Executive Summary](#1-executive-summary)
2. [Profil Sistem SISFOKOL v7.00](#2-profil-sistem-sisfokol-v700)
3. [Analisis Struktur Folder dan Arsitektur](#3-analisis-struktur-folder-dan-arsitektur)
4. [Analisis Database Schema](#4-analisis-database-schema)
5. [Business Flow Analysis per Modul](#5-business-flow-analysis-per-modul)
6. [Data Flow Analysis](#6-data-flow-analysis)
7. [System Flow Analysis](#7-system-flow-analysis)
8. [Gap Analysis: Native PHP ke Laravel 11](#8-gap-analysis-native-php-ke-laravel-11)
9. [Migration Blueprint dan Strategy](#9-migration-blueprint-dan-strategy)
10. [Risk Assessment dan Mitigation](#10-risk-assessment-dan-mitigation)
11. [Timeline dan Roadmap Implementasi](#11-timeline-dan-roadmap-implementasi)
12. [Appendices](#12-appendices)

---

## 1. EXECUTIVE SUMMARY

### 1.1 Latar Belakang

SISFOKOL v7.00 (Code: SmartOffice) adalah Sistem Informasi Sekolah yang dikembangkan menggunakan **Native PHP 8.2.4** dengan database **MySQL/MariaDB**. Sistem ini dirancang untuk lingkungan sekolah yang menerapkan konsep Smart Office dengan fitur-fitur terintegrasi antar bagian.

### 1.2 Tujuan Dokumentasi

Dokumen ini bertujuan untuk:
- Melakukan audit menyeluruh terhadap sistem existing
- Memetakan business flow, data flow, dan system flow
- Merancang blueprint migrasi lengkap ke Laravel 11
- Menyediakan panduan implementasi yang terstruktur

### 1.3 Scope Analisis

| Aspek | Coverage |
|-------|----------|
| File PHP | 1,675 files |
| Database Tables | 75 tables |
| User Roles | 9 role pengguna |
| Modul Utama | 11 modul fungsional |
| Lines of Code (SQL) | 10,601 lines |

### 1.4 Temuan Kunci

1. **Arsitektur Monolitik**: Sistem menggunakan struktur folder-based tanpa framework
2. **Security Concerns**: Password disimpan dengan MD5 (tidak secure)
3. **Database Engine**: Menggunakan MyISAM (seharusnya InnoDB)
4. **Coding Pattern**: Procedural programming dengan fungsi global
5. **Session Management**: Manual session handling dengan custom validation

---

## 2. PROFIL SISTEM SISFOKOL v7.00

### 2.1 Informasi Umum

| Attribute | Value |
|-----------|-------|
| Nama Sistem | SISFOKOL v7.00 (Code: SmartOffice) |
| Developer | Agus Muhajir, S.Kom |
| Technology Stack | Native PHP 8.2.4 + MySQL/MariaDB |
| Server Requirement | XAMPP PHP 8.2.4 |
| Repository | GitLab: @hajirodeon |
| License | Proprietary (Custom Content Available) |

### 2.2 Fitur Utama

```
✓ Sistem Penilaian Mapel dan Raport
✓ Sistem Bimbingan Konseling (BK)
✓ Sistem Presensi Kehadiran dengan QR Code
✓ Sistem Keuangan Siswa dan Tunggakan
✓ Sistem Inventaris/Sarana Prasarana
✓ Sistem Jadwal Pelajaran
✓ Sistem Jurnal Guru Mengajar
✓ Sistem Absensi/Ijin
✓ Sistem Guru Piket
✓ Sistem Filebox RPP Silabus
```

### 2.3 User Roles dan Akses

| No | Role | Username Default | Password Default | Folder Akses |
|----|------|------------------|------------------|--------------|
| 1 | Administrator | admin | admin | `/adm/` |
| 2 | Kepala Sekolah | 234 | 234 | `adminv/` |
| 3 | Sarpras | 234 | 234 | `adminv/inv/` |
| 4 | Bendahara | 234 | 234 | `admbdh/` |
| 5 | Wali Kelas | 234 | 234 | `admwk/` |
| 6 | Guru BK | 234 | 234 | `admbk/` |
| 7 | Guru Mapel | 234 | 234 | `admgr/` |
| 8 | Siswa | 810001 | (MD5 hash) | `admsw/` |
| 9 | Petugas Piket | 1122 | (MD5 hash) | `admpiket/` |

---

## 3. ANALISIS STRUKTUR FOLDER DAN ARSITEKTUR

### 3.1 Root Directory Structure

```
sisfokol-v7.00-code-smartoffice/
├── adm/                    # Administrator Access
├── admbdh/                 # Bendahara Access
├── admbk/                  # Guru BK Access
├── admgr/                  # Guru Mapel Access
├── adminv/                 # Kepala Sekolah & Sarpras
├── admks/                  # (Struktur mirip adm/)
├── admpiket/               # Petugas Piket
├── admsw/                  # Siswa Access
├── admwk/                  # Wali Kelas Access
├── db/                     # Database Scripts
├── filebox/                # File Storage
├── img/                    # Image Assets
├── inc/                    # Include Files (Core)
├── template/               # HTML Templates
├── tmp_tampilan/           # UI Screenshots
├── index.php               # Landing Page
├── login.php               # Login Handler
├── logout.php              # Logout Handler
└── expire.php              # Session Expiry Check
```

### 3.2 Detail Struktur Per Modul

#### 3.2.1 Administrator (`/adm/`)

```
adm/
├── ab/           # Absensi Pegawai & Siswa
│   ├── absensi.php
│   ├── lap_bln.php
│   ├── lap_pegawai.php
│   ├── lap_siswa.php
│   ├── lap_tgl.php
│   └── lap_thn.php
├── akad/         # Akademik
│   ├── mapel.php
│   ├── mapel_desc.php
│   └── rpp_silabus.php
├── ek/           # Ekstrakurikuler
│   ├── ekstra.php
│   ├── ekstra_siswa.php
│   ├── lap_ekstra.php
│   └── lap_nilai.php
├── h/            # Home/Sub-login
├── im/           # Ijin Masuk
│   ├── cek.php
│   ├── ijin.php
│   ├── ijin_pdf.php
│   ├── ijin_qrcode.php
│   └── Laporan (bln, guru, siswa, tgl, thn)
├── inv/          # Inventaris
│   ├── m_brg.php
│   ├── sarpras.php
│   ├── lap_rekap.php
│   └── tmp/
├── jw/           # Jadwal
│   ├── jadwal.php
│   ├── lap_guru.php
│   └── lap_mapel.php
├── keu/          # Keuangan
│   ├── history.php
│   ├── item.php
│   ├── lunas.php
│   ├── nota.php
│   ├── tunggakan.php
│   └── WA Integration
├── m/            # Master Data
│   ├── bendahara.php
│   ├── bk.php
│   ├── guru.php
│   ├── kelas.php
│   ├── pegawai.php
│   ├── piket.php
│   ├── prestasi.php
│   ├── ruang.php
│   ├── sarpras.php
│   ├── siswa.php
│   ├── tapel.php
│   └── wk.php
├── nabung/       # Tabungan Siswa
├── nil/          # Nilai & Raport
│   ├── catatan.php
│   ├── kenaikan.php
│   ├── mapel.php
│   ├── raport.php
│   ├── raport_pdf.php
│   └── sikap.php
├── pb/           # Pembinaan (BK)
├── pen/          # Penilaian
│   ├── bln.php
│   ├── smt.php
│   ├── thn.php
│   └── Laporan
├── ph/           # Catatan Harian
├── pl/           # Pelanggaran
├── ps/           # Presensi
│   ├── presensi.php
│   ├── presensi_manual.php
│   ├── pulang.php
│   ├── pulang_manual.php
│   └── waktu.php
├── pt/           # Prestasi
└── s/            # Ganti Password
```

#### 3.2.2 Core Include Files (`/inc/`)

```
inc/
├── cek/                  # Session Validation
│   ├── adm.php
│   ├── admbdh.php
│   ├── admbk.php
│   ├── admgr.php
│   ├── adminv.php
│   ├── admks.php
│   ├── admpiket.php
│   ├── admsw.php
│   └── admwk.php
├── class/                # Third-party Classes
│   ├── dompdf/           # PDF Generation
│   ├── phpqrcode/        # QR Code
│   ├── phpspreadsheet/   # Excel Export
│   └── simpleimage/      # Image Manipulation
├── js/                   # JavaScript Libraries
│   ├── ckeditor/
│   └── jquery/
├── style/                # CSS Stylesheets
├── config.php            # Configuration
├── fungsi.php            # Global Functions (~34KB)
├── koneksi.php           # Database Connection
├── niltpl.php            # Template Helper
└── niltpl2.php           # Template Helper 2
```

### 3.3 Authentication Flow Analysis

**File: `login.php` (25,387 bytes)**

```php
// Flow Diagram:
1. User mengakses login.php
2. Session start
3. Load config, fungsi, koneksi, paging
4. Load template login
5. POST btnOK → Validasi input
6. Cek tipe user (tp01-tp09)
7. Query database sesuai role
8. MD5 password verification
9. Set session variables
10. Log login ke user_log_login
11. Redirect ke folder sesuai role
```

**Session Variables:**
```php
$_SESSION['kd1_session']      // User ID
$_SESSION['tipe_session']     // Role Type
$_SESSION['no1_session']      // Employee Code
$_SESSION['nip1_session']     // NIP
$_SESSION['nm1_session']      // Full Name
$_SESSION['username1_session']// Username
$_SESSION['pass1_session']    // MD5 Password
$_SESSION['hajirobe_session'] // Base URL
$_SESSION['janiskd']          // Access Folder
```

### 3.4 Security Check Files

Setiap folder memiliki validasi session di `inc/cek/[role].php`:

```php
// Pattern Validation:
$kd6_session = nosql($_SESSION['kd6_session']);
$username6_session = nosql($_SESSION['username6_session']);
$tipe_session = balikin2($_SESSION['tipe_session']);
$pass6_session = nosql($_SESSION['pass6_session']);

// Query Validation:
SELECT kd FROM adminx 
WHERE kd = '$kd6_session' 
AND usernamex = '$username6_session' 
AND passwordx = '$pass6_session'

// If invalid → redirect to login with message
```

---

## 4. ANALISIS DATABASE SCHEMA

### 4.1 Database Overview

| Property | Value |
|----------|-------|
| Database Name | sisfokol_v7 |
| Total Tables | 75 |
| Engine | MyISAM (⚠️ Should be InnoDB) |
| Character Set | latin1 / utf8mb4 |
| SQL File Size | 1.4 MB |
| Total INSERT Statements | ~10,000+ |

### 4.2 Table Categories

#### 4.2.1 Master Data Tables (m_*)

| Table Name | Purpose | Key Fields |
|------------|---------|------------|
| `m_siswa` | Student Data | kd, usernamex, passwordx, kode (NIS), nama, kelas, tapel |
| `m_pegawai` | Employee Data | kd, usernamex, passwordx, nama, kode (NIP), jabatan |
| `m_kelas` | Class Data | kd, no, nama, kelas |
| `m_mapel` | Subject Data | kd, tapel, kelas, jenis, kode, nama, kkm, pegawai_kd |
| `m_tapel` | Academic Year | kd, nama, tapel, aktif |
| `m_ruang` | Room Data | kd, no, nama, ruang |
| `m_hari` | Day Master | kd, no, hari |
| `m_jam` | Hour Master | kd, no, jam_ke, mulai, selesai |
| `m_ekstra` | Extracurricular | kd, no, nama, ekstrakurikuler |
| `m_pembinaan` | Coaching Types | kd, no, jenis, pembinaan |
| `m_prestasi` | Achievement Types | kd, no, jenis, prestasi |
| `m_bk_point` | BK Points | kd, no, point, jenis |
| `m_walikelas` | Homeroom Teacher | kd, tapel, smt, kelas, pegawai_kd |
| `m_gurubk` | Counselor Teacher | kd, tapel, smt, kelas, pegawai_kd |
| `m_bendahara` | Treasurer | kd, tapel, smt, pegawai_kd |
| `m_piket` | Duty Officer | kd, tapel, hari, pegawai_kd |
| `m_sarpras` | Facilities Master | kd, no, nama_barang |
| `m_kib_jenis` | Asset Type KIB | kd, no, jenis |
| `m_kib_kode` | Asset Code KIB | kd, no, kode |
| `m_keu_siswa` | Student Finance Config | kd, nominal, ket |
| `m_user` | General Users | kd, usernamex, passwordx, nama |
| `m_ks` | School Principal | kd, tapel, smt, pegawai_kd |
| `m_mapel_deskripsi` | Subject Description | kd, tapel, kelas, deskripsi |
| `m_mapel_jns` | Subject Type | kd, no, jenis |
| `m_waktu` | Time Settings | kd, tapel, smt |
| `m_waktu_jadwal` | Schedule Time | kd, tapel, smt, jam_ke |

#### 4.2.2 Transaction Tables (siswa_*, user_*)

| Table Name | Purpose | Key Fields |
|------------|---------|------------|
| `siswa_bayar` | Payment Records | kd, siswa_kd, nominal, postdate |
| `siswa_bayar_rincian` | Payment Details | kd, bayar_kd, keu_kd, nominal |
| `siswa_bayar_tagihan` | Billing Records | kd, siswa_kd, keu_kd, bulan |
| `siswa_ekstra` | Student Extracurricular | kd, siswa_kd, ekstra_kd |
| `siswa_pelanggaran` | Violations | kd, siswa_kd, jenis, poin, tanggal |
| `siswa_prestasi` | Achievements | kd, siswa_kd, jenis, tingkat, juara |
| `siswa_nilai_bln` | Monthly Grades | kd, siswa_kd, mapel_kd, nilai |
| `siswa_nilai_smt` | Semester Grades | kd, siswa_kd, mapel_kd, nilai |
| `siswa_nilai_thn` | Annual Grades | kd, siswa_kd, mapel_kd, nilai |
| `siswa_raport_sikap` | Attitude Report | kd, siswa_kd, spiritual, sosial |
| `siswa_raport_catatan` | Report Notes | kd, siswa_kd, catatan |
| `siswa_raport_kenaikan` | Promotion Status | kd, siswa_kd, status, ket |
| `siswa_raport_rangking` | Ranking | kd, siswa_kd, rangking, rata |
| `siswa_mapel_absensi` | Subject Attendance | kd, siswa_kd, mapel_kd, status |
| `siswa_tugas` | Assignments | kd, siswa_kd, tugas, nilai |
| `siswa_soal` | Questions Bank | kd, mapel_kd, soal, kunci |
| `siswa_soal_nilai` | Question Scores | kd, siswa_kd, soal_kd, nilai |
| `siswa_saran` | Suggestions | kd, siswa_kd, saran |
| `user_absensi` | User Attendance | kd, user_kd, status, postdate |
| `user_presensi` | User Presence (QR) | kd, user_kd, jenis, postdate |
| `user_ijin` | User Permissions | kd, user_kd, alasan, postdate |
| `user_filebox` | File Storage | kd, user_kd, file, ket |
| `user_log_login` | Login Logs | kd, user_kd, ipnya, postdate |
| `user_log_entri` | Entry Logs | kd, user_kd, dibaca, postdate |
| `user_piket` | Duty Logs | kd, pegawai_kd, postdate |

#### 4.2.3 Inventory Tables (inv_kib_*)

| Table Name | KIB Type | Purpose |
|------------|----------|---------|
| `inv_kib_a` | KIB A | Tanah/Land |
| `inv_kib_b` | KIB B | Peralatan Mesin/Equipment |
| `inv_kib_c` | KIB C | Gedung Bangunan/Buildings |
| `inv_kib_d` | KIB D | Jalan Irigasi Jaringan |
| `inv_kib_e` | KIB E | Aset Tetap Lainnya |
| `inv_kib_f` | KIB F | Konstruksi Dalam Pengerjaan |

#### 4.2.4 Kurikulum Merdeka Tables (kurmer_*)

| Table Name | Purpose |
|------------|---------|
| `kurmer_asesmen_formatif` | Formative Assessment |
| `kurmer_mapel_lm` | Learning Objectives |
| `kurmer_mapel_tp` | Teaching Objectives |
| `kurmer_nilai_asesmen_formatif` | Formative Scores |
| `kurmer_nilai_asesmen_formatif_detail` | Formative Details |
| `kurmer_nilai_asesmen_sumatif` | Sumative Scores |
| `kurmer_nilai_asesmen_sumatif_detail` | Sumative Details |
| `kurmer_nilai_proyek` | Project Scores |
| `kurmer_nilai_proyek_proses` | Project Process |
| `kurmer_proyek` | Projects |
| `kurmer_proyek_detail` | Project Details |

#### 4.2.5 Other Tables

| Table Name | Purpose |
|------------|---------|
| `adminx` | Admin Accounts |
| `a_profil` | School Profile (Google Maps) |
| `jadwal` | Class Schedule |
| `rev_guru_absensi` | Teacher Attendance Revision |
| `rev_guru_agenda` | Teacher Agenda Revision |
| `wa_tagihan_siswa` | WhatsApp Billing |

### 4.3 Critical Table Schema Details

#### 4.3.1 m_siswa (Student Master)

```sql
CREATE TABLE `m_siswa` (
  `kd` varchar(50) NOT NULL DEFAULT '',           -- Primary Key (MD5 hash)
  `usernamex` varchar(100) DEFAULT NULL,          -- Login Username (NIS)
  `passwordx` varchar(100) DEFAULT NULL,          -- MD5 Password
  `kode` varchar(50) DEFAULT NULL,                -- NIS Number
  `nama` varchar(100) DEFAULT NULL,               -- Full Name
  `postdate` datetime DEFAULT NULL,               -- Created Date
  `passwordx_ortu` varchar(100) DEFAULT NULL,     -- Parent Password
  `tapel` varchar(100) DEFAULT NULL,              -- Academic Year
  `kelas` varchar(100) DEFAULT NULL,              -- Current Class
  `nourut` varchar(5) DEFAULT NULL,               -- Sequence Number
  `qrcode` varchar(100) DEFAULT NULL,             -- QR Code String
  `jml_ekstra` varchar(5) DEFAULT NULL,           -- Extra Count
  `jml_absen_sakit` varchar(5) DEFAULT NULL,      -- Sick Leave Count
  `jml_absen_ijin` varchar(5) DEFAULT NULL,       -- Permission Count
  `jml_absen_alpha` varchar(5) DEFAULT NULL,      -- Alpha Count
  `subtotal_nominal` varchar(15) DEFAULT NULL,    -- Total Billing
  `subtotal_setor` varchar(15) DEFAULT NULL,      -- Total Paid
  `subtotal_belum` varchar(15) DEFAULT NULL,      -- Outstanding
  `nowa` varchar(100) DEFAULT NULL,               -- WhatsApp Number
  `jml_pelanggaran` varchar(5) DEFAULT NULL,      -- Violation Count
  `subtotal_pelanggaran` varchar(5) DEFAULT NULL, -- Violation Points
  `jml_presensi` varchar(5) DEFAULT NULL,         -- Presence Count
  `jml_prestasi` varchar(5) DEFAULT NULL,         -- Achievement Count
  `subtotal_prestasi` varchar(5) DEFAULT NULL,    -- Achievement Points
  `subtotal_akhir` varchar(5) DEFAULT NULL        -- Final Score
) ENGINE=MyISAM DEFAULT CHARSET=latin1;
```

**⚠️ Issues Identified:**
1. Using VARCHAR for numeric fields (should be INT/DECIMAL)
2. Storing counts in master table (denormalization)
3. MD5 password hashing (insecure)
4. No foreign key constraints
5. MyISAM engine (no transaction support)

#### 4.3.2 m_pegawai (Employee Master)

```sql
CREATE TABLE `m_pegawai` (
  `kd` varchar(50) NOT NULL DEFAULT '',
  `usernamex` varchar(100) DEFAULT NULL,
  `passwordx` varchar(100) DEFAULT NULL,
  `nama` varchar(100) DEFAULT NULL,
  `kode` varchar(50) DEFAULT NULL,                -- NIP
  `jabatan` varchar(100) DEFAULT NULL,
  `postdate` datetime DEFAULT NULL,
  `jml_absen_sakit` varchar(5) DEFAULT NULL,
  `jml_absen_ijin` varchar(5) DEFAULT NULL,
  `jml_absen_alpha` varchar(5) DEFAULT NULL,
  `jml_mengajar` varchar(5) DEFAULT NULL,
  `nowa` varchar(100) DEFAULT NULL,
  `jml_presensi` varchar(5) DEFAULT NULL
) ENGINE=MyISAM DEFAULT CHARSET=latin1;
```

#### 4.3.3 jadwal (Class Schedule)

```sql
CREATE TABLE `jadwal` (
  `kd` varchar(50) NOT NULL DEFAULT '',
  `tapel` varchar(100) DEFAULT NULL,
  `smt` varchar(1) DEFAULT NULL,
  `kelas` varchar(100) DEFAULT NULL,
  `hari` varchar(100) DEFAULT NULL,
  `hari_no` varchar(1) DEFAULT NULL,
  `jam_ke` varchar(5) DEFAULT NULL,
  `waktu` varchar(100) DEFAULT NULL,
  `mapel_kode` varchar(50) DEFAULT NULL,
  `mapel_nama` varchar(100) DEFAULT NULL,
  `ruang` varchar(100) DEFAULT NULL,
  `pegawai_kd` varchar(50) DEFAULT NULL,
  `pegawai_kode` varchar(100) DEFAULT NULL,
  `pegawai_nama` varchar(100) DEFAULT NULL,
  `postdate` datetime DEFAULT NULL
) ENGINE=MyISAM DEFAULT CHARSET=latin1;
```

---

## 5. BUSINESS FLOW ANALYSIS PER MODUL

### 5.1 Modul Authentication & Authorization

**Business Process:**
```
[User] → [Login Page] → [Credential Validation] → [Role Detection]
                                              ↓
[Session Creation] ← [Database Query] ← [Password MD5 Verify]
                                              ↓
[Log Login Event] → [Redirect to Dashboard] → [Access Control per Feature]
```

**Critical Files:**
- `login.php` - Main login handler
- `logout.php` - Session destruction
- `inc/cek/*.php` - Role-based access control
- `expire.php` - Session timeout check

**Business Rules:**
1. 9 different user types with separate folder access
2. Session validation on every page load
3. Login logging for audit trail
4. Password stored as MD5 hash
5. Petugas Piket only active on assigned dates

### 5.2 Modul Master Data Management

**Sub-Modules:**
- Data Siswa (Student)
- Data Pegawai (Employee)
- Data Kelas (Class)
- Data Mata Pelajaran (Subject)
- Data Tahun Pelajaran (Academic Year)
- Data Ruang (Room)
- Data Ekstrakurikuler
- Data Pembinaan & Pelanggaran
- Data Prestasi
- Data Sarana Prasarana
- Data Piket
- Data Wali Kelas
- Data Guru BK
- Data Bendahara

**Business Flow (Example: Student Management):**
```
[Admin] → [Form Input Siswa] → [Validation] → [Generate KD (MD5)]
                                                    ↓
                                        [Insert to m_siswa]
                                                    ↓
                                        [Generate QR Code]
                                                    ↓
                                        [Upload Photo (optional)]
                                                    ↓
                                        [Print Student Card]
```

**CRUD Operations:**
- Create: Form input with validation
- Read: List view with pagination & search
- Update: Edit form with pre-filled data
- Delete: Soft delete (not implemented, direct delete)
- Export: Excel/PDF export functionality
- Import: Excel import for bulk data

### 5.3 Modul Presensi & Absensi

**Types of Attendance:**
1. **Presensi Harian** - Daily presence (QR Code based)
2. **Absensi Mapel** - Subject-based attendance
3. **Absensi Pegawai** - Employee attendance
4. **Ijin/Sakit/Alpha** - Leave management
5. **Pulang** - Departure tracking

**Business Flow (QR Presensi):**
```
[User] → [Scan QR Code] → [Decode QR] → [Validate User]
                                            ↓
                              [Check Time Window] → [Early/On-time/Late]
                                            ↓
                              [Capture Location (Lat/Lng)]
                                            ↓
                              [Insert to user_presensi]
                                            ↓
                              [Update Counter in m_siswa/m_pegawai]
                                            ↓
                              [Display Confirmation]
```

**Files:**
- `adm/ps/presensi.php` - Manual presence entry
- `adm/ps/waktu.php` - Time window configuration
- `admpiket/` - Duty officer interface
- `inc/class/phpqrcode/` - QR generation

### 5.4 Modul Keuangan Siswa

**Features:**
- Setup biaya sekolah (SPP, Uang Gedung, dll)
- Tagihan bulanan
- Pembayaran
- Tunggakan
- Nota/Cetak kwitansi
- WhatsApp notification
- Tabungan siswa

**Business Flow (Payment):**
```
[Bendahara] → [Select Student] → [View Tagihan]
                                      ↓
                            [Enter Payment Amount]
                                      ↓
                            [Validate Payment]
                                      ↓
                            [Insert siswa_bayar]
                            [Insert siswa_bayar_rincian]
                            [Update siswa_bayar_tagihan]
                                      ↓
                            [Update subtotal in m_siswa]
                                      ↓
                            [Generate Nota PDF]
                            [Send WA Notification (optional)]
```

**Tables Involved:**
- `m_keu_siswa` - Finance configuration
- `siswa_bayar_tagihan` - Billing
- `siswa_bayar` - Payment header
- `siswa_bayar_rincian` - Payment details
- `wa_tagihan_siswa` - WhatsApp logs

### 5.5 Modul Penilaian & Raport

**Assessment Types:**
1. **Nilai Harian** - Daily tests
2. **Nilai Bulanan** - Monthly exams
3. **Nilai Semester** - Semester exams
4. **Nilai Tahunan** - Annual exams
5. **Sikap Spiritual & Sosial** - Attitude assessment
6. **Proyek** - Project-based assessment (Kurikulum Merdeka)

**Business Flow (Input Nilai):**
```
[Guru Mapel] → [Select Class] → [Select Subject]
                                      ↓
                            [Display Student List]
                                      ↓
                            [Enter Scores]
                                      ↓
                            [Calculate Average]
                                      ↓
                            [Convert to Predicate]
                                      ↓
                            [Save to siswa_nilai_*]
                                      ↓
                            [Generate Deskripsi]
```

**Raport Generation:**
```
[Wali Kelas] → [Select Semester] → [Lock Editing]
                                        ↓
                              [Aggregate All Scores]
                              [Calculate Ranking]
                              [Generate Sikap Values]
                              [Add Catatan Wali Kelas]
                                        ↓
                              [Generate PDF Raport]
                              [Print/Download]
```

**Kurikulum Merdeka Specific:**
- Asesmen Formatif (ongoing assessment)
- Asesmen Sumatif (end of period)
- Projek Penguatan Profil Pelajar Pancasila (P5)
- Dimensi Profil Pelajar Pancasila

### 5.6 Modul Bimbingan Konseling (BK)

**Features:**
- Point pelanggaran
- Jenis pembinaan
- Catatan konseling
- Rekap pelanggaran
- Rekap pembinaan

**Business Flow:**
```
[Guru BK] → [Select Student] → [Record Violation]
                                      ↓
                            [Assign Points]
                            [Select Jenis Pembinaan]
                                      ↓
                            [Insert siswa_pelanggaran]
                            [Update jml_pelanggaran in m_siswa]
                                      ↓
                            [Notify Wali Kelas]
                            [Notify Parents (WA)]
```

### 5.7 Modul Inventaris (KIB)

**KIB Categories:**
- KIB A: Tanah
- KIB B: Peralatan dan Mesin
- KIB C: Gedung dan Bangunan
- KIB D: Jalan, Irigasi, Jaringan
- KIB E: Aset Tetap Lainnya
- KIB F: Konstruksi dalam Pengerjaan

**Business Flow:**
```
[Sarpras] → [Select KIB Type] → [Input Asset Data]
                                      ↓
                            [Assign Kode Barang]
                            [Upload Document (optional)]
                                      ↓
                            [Insert to inv_kib_*]
                            [Update inventory count]
                                      ↓
                            [Generate Label/Barcode]
                            [Print KIB Report]
```

### 5.8 Modul Jadwal Pelajaran

**Business Flow:**
```
[Admin/Wakakur] → [Set Tahun Pelajaran] → [Set Semester]
                                                ↓
                                      [Define Time Slots]
                                      [Assign Classes]
                                                ↓
                                      [Drag-Drop Interface]
                                      [Assign Teachers]
                                      [Assign Rooms]
                                                ↓
                                      [Conflict Detection]
                                      [Save to jadwal]
                                                ↓
                                      [Publish Schedule]
                                      [Print per Class/Guru]
```

### 5.9 Modul Filebox (RPP/Silabus)

**Features:**
- Upload dokumen RPP
- Upload Silabus
- Categorization by subject
- Access control
- Version control (basic)

**Business Flow:**
```
[Guru] → [Select Subject] → [Upload File]
                                  ↓
                        [Validate File Type]
                        [Generate Unique Filename]
                                  ↓
                        [Save to filebox/]
                        [Insert to user_filebox]
                                  ↓
                        [Notify Admin]
                        [Await Approval (optional)]
```

---

## 6. DATA FLOW ANALYSIS

### 6.1 Data Flow Diagram Level 0 (Context Diagram)

```
┌─────────────┐
│   External  │
│   Entities  │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────────────┐
│                                         │
│   SISFOKOL v7.00 System                 │
│   (Native PHP Application)              │
│                                         │
└─────────────────────────────────────────┘
       │
       ▼
┌─────────────┐
│   MySQL     │
│   Database  │
└─────────────┘
```

**External Entities:**
1. Administrator
2. Kepala Sekolah
3. Guru (Mapel, BK, Wali Kelas)
4. Staff (Bendahara, Sarpras, Piket)
5. Siswa
6. Orang Tua (via WA)

### 6.2 Data Flow Diagram Level 1 (Major Processes)

```
Process 1: Authentication
┌────────┐    Credentials    ┌─────────────┐
│  User  │──────────────────▶│   Login     │
└────────┘                   │   Process   │
                             └──────┬──────┘
                                    │
                                    ▼
                             ┌─────────────┐
                             │  Session    │
                             │  Management │
                             └─────────────┘

Process 2: Master Data Management
┌────────┐    CRUD Ops     ┌─────────────┐
│  Admin │────────────────▶│   Master    │
└────────┘                 │   Data Mgr  │
                           └──────┬──────┘
                                  │
                                  ▼
                           ┌─────────────┐
                           │ m_* Tables  │
                           └─────────────┘

Process 3: Transaction Processing
┌────────┐   Transactions  ┌─────────────┐
│  User  │────────────────▶│ Transaction │
└────────┘                 │   Processor │
                           └──────┬──────┘
                                  │
                                  ▼
                           ┌─────────────┐
                           │ siswa_*,    │
                           │ user_*      │
                           └─────────────┘

Process 4: Reporting
┌────────┐    Report Req   ┌─────────────┐
│  User  │────────────────▶│   Report    │
└────────┘                 │   Generator │
                           └──────┬──────┘
                                  │
                                  ▼
                           ┌─────────────┐
                           │ PDF/Excel   │
                           │   Output    │
                           └─────────────┘
```

### 6.3 Entity Relationship Overview

**Core Entities:**

```
m_siswa (1) ──────< (N) siswa_bayar
     │
     │ (1)
     │
     ▼ (N)
siswa_nilai_smt

m_pegawai (1) ──────< (N) jadwal
     │
     │ (1)
     │
     ▼ (N)
user_presensi

m_kelas (1) ──────< (N) jadwal
     │
     │ (1)
     │
     ▼ (N)
siswa_nilai_bln

m_mapel (1) ──────< (N) siswa_nilai_smt
     │
     │ (1)
     │
     ▼ (N)
siswa_mapel_absensi

m_tapel (1) ──────< (N) jadwal
     │
     │ (1)
     │
     ▼ (N)
siswa_nilai_thn
```

### 6.4 Data Dictionary Highlights

**Primary Keys Pattern:**
- All tables use `kd` (varchar 50) as primary key
- Generated using MD5 hash or custom algorithm
- ⚠️ No auto-increment IDs used

**Date/Time Fields:**
- `postdate` - datetime, record creation timestamp
- Format: `YYYY-MM-DD HH:MM:SS`

**Common Field Patterns:**
- `usernamex` - Login username
- `passwordx` - MD5 hashed password
- `nama` - Display name
- `kode` - Business code (NIS, NIP, etc.)
- `ket` - Description/notes

---

## 7. SYSTEM FLOW ANALYSIS

### 7.1 Request-Response Cycle

```
Browser Request
     │
     ▼
┌─────────────┐
│ index.php   │ or specific module PHP file
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ session_start() │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ require config.php │
│ require fungsi.php │
│ require koneksi.php │
│ require cek/[role].php │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Load Template │
│ LoadTpl()    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Process Logic │
│ (GET/POST)   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Database Query │
│ mysqli_query() │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Render Output │
│ Replace vars │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ echo/send   │
└──────┬──────┘
       │
       ▼
Browser Response
```

### 7.2 Common Function Usage

**From `inc/fungsi.php`:**

| Function | Purpose | Usage Count |
|----------|---------|-------------|
| `cegah()` | SQL Injection Prevention | High |
| `nosql()` | Clean numeric input | High |
| `balikin()` | Reverse htmlspecialchars | High |
| `LoadTpl()` | Template loader | High |
| `xloc()` | JavaScript redirect | High |
| `pekem()` | Show message & redirect | Medium |
| `xfree()` | Free result set | High |
| `xclose()` | Close DB connection | High |
| `get_client_ip_env()` | Get IP address | Medium |

### 7.3 Template System

**Template Location:** `/template/`

**Pattern:**
```php
$tpl = LoadTpl("../template/adm.html");

// Replace placeholders
$isi = str_replace('REPLACE_VAR', $value, $tpl);

// Echo final output
echo $isi;
```

**Template Variables:**
- `<!-- START info -->` ... `<!-- END info -->`
- `<!-- START body -->` ... `<!-- END body -->`
- Dynamic content blocks

---

## 8. GAP ANALYSIS: NATIVE PHP KE LARAVEL 11

### 8.1 Architecture Comparison

| Aspect | Current (Native PHP) | Target (Laravel 11) |
|--------|---------------------|---------------------|
| **Architecture** | Procedural, folder-based | MVC Pattern |
| **Routing** | File-based routing | Route definition (web.php) |
| **Database** | Direct mysqli queries | Eloquent ORM + Migrations |
| **Authentication** | Custom session management | Laravel Sanctum/Breeze |
| **Security** | MD5 passwords, manual sanitization | Bcrypt/Argon2, built-in protection |
| **Templating** | HTML with placeholders | Blade Templates |
| **Validation** | Manual if-else checks | Form Request Validation |
| **File Structure** | Flat folder hierarchy | Structured directories |
| **Dependency Mgmt** | Manual includes | Composer autoloading |
| **Testing** | None | PHPUnit included |

### 8.2 Detailed Gap Analysis

#### 8.2.1 Authentication & Authorization

**Current State:**
```php
// Session-based with manual validation
session_start();
$_SESSION['kd6_session'] = $kd;
$_SESSION['pass6_session'] = md5($password);

// Validation in every page
require("inc/cek/adm.php");
```

**Target State (Laravel 11):**
```php
// Using Laravel Auth
use Illuminate\Support\Facades\Auth;

Auth::attempt(['username' => $username, 'password' => $password]);

// Middleware for role-based access
Route::middleware(['auth', 'role:admin'])->group(function () {
    // Admin routes
});
```

**Gaps:**
1. MD5 → Bcrypt/Argon2id
2. Custom session → Laravel session
3. Manual role check → Middleware/Policy
4. No remember me → Token-based auth optional

#### 8.2.2 Database Layer

**Current State:**
```php
mysqli_query($koneksi, "SELECT * FROM m_siswa WHERE kd = '$kd'");
$row = mysqli_fetch_assoc($result);
```

**Target State:**
```php
// Eloquent Model
$siswa = Siswa::find($kd);

// Or Query Builder
$siswa = DB::table('m_siswa')->where('kd', $kd)->first();
```

**Gaps:**
1. No ORM → Eloquent models needed
2. No migrations → Migration files required
3. No relationships → Define Eloquent relationships
4. MyISAM → InnoDB with foreign keys
5. No seeders → Database seeders for test data

#### 8.2.3 Validation

**Current State:**
```php
if ((empty($username)) OR (empty($password))) {
    $pesan = "Input Tidak Lengkap";
    pekem($pesan, $filenya);
    exit();
}
```

**Target State:**
```php
// Form Request
public function rules()
{
    return [
        'username' => 'required|string|max:100',
        'password' => 'required|string|min:8',
    ];
}
```

**Gaps:**
1. Manual validation → Form Request classes
2. No server-side validation rules → Define validation rules
3. Error messages scattered → Centralized error handling

#### 8.2.4 File Organization

**Current:**
```
/adm/keu/lunas.php
/adm/nil/raport.php
/adm/m/siswa.php
```

**Target (Laravel):**
```
app/Http/Controllers/Admin/FinanceController.php
app/Http/Controllers/Admin/AcademicController.php
app/Http/Controllers/Admin/MasterDataController.php
resources/views/admin/finance/payments.blade.php
resources/views/admin/academic/reports.blade.php
routes/web.php (route definitions)
database/migrations/xxxx_create_siswas_table.php
```

### 8.3 Technical Debt Identification

| Issue | Severity | Impact | Effort to Fix |
|-------|----------|--------|---------------|
| MD5 Password Hashing | 🔴 Critical | Security vulnerability | Medium |
| MyISAM Engine | 🟠 High | No transactions, data integrity | High |
| SQL Injection Risk | 🟠 High | Security vulnerability | Medium |
| No Input Validation | 🟠 High | Data quality issues | High |
| Hardcoded Paths | 🟡 Medium | Deployment issues | Low |
| No Error Handling | 🟡 Medium | Poor UX | Medium |
| Duplicate Code | 🟡 Medium | Maintenance burden | High |
| No Logging | 🟡 Medium | Debugging difficulty | Low |
| Denormalized Data | 🟡 Medium | Data inconsistency | High |
| No API Layer | 🟢 Low | Integration limitations | Medium |

---

## 9. MIGRATION BLUEPRINT DAN STRATEGY

### 9.1 Migration Phases

```
Phase 1: Foundation (Weeks 1-2)
├── Laravel 11 Installation
├── Environment Setup
├── Database Migration Design
└── Authentication Implementation

Phase 2: Core Modules (Weeks 3-6)
├── Master Data Management
├── User Management
└── Basic CRUD Operations

Phase 3: Transaction Modules (Weeks 7-10)
├── Finance Module
├── Attendance Module
├── Assessment Module
└── Schedule Module

Phase 4: Advanced Features (Weeks 11-14)
├── Reporting & PDF Generation
├── QR Code Integration
├── WhatsApp Integration
└── File Management

Phase 5: Testing & Deployment (Weeks 15-16)
├── Unit Testing
├── Integration Testing
├── UAT
└── Production Deployment
```

### 9.2 Laravel 11 Project Structure

```
sisfokol-laravel11/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Admin/
│   │   │   │   ├── DashboardController.php
│   │   │   │   ├── MasterDataController.php
│   │   │   │   ├── FinanceController.php
│   │   │   │   └── AcademicController.php
│   │   │   ├── Teacher/
│   │   │   ├── Student/
│   │   │   └── Auth/
│   │   ├── Middleware/
│   │   │   ├── CheckRole.php
│   │   │   └── CheckSession.php
│   │   └── Requests/
│   │       ├── StudentRequest.php
│   │       └── PaymentRequest.php
│   ├── Models/
│   │   ├── User.php
│   │   ├── Siswa.php
│   │   ├── Pegawai.php
│   │   ├── Kelas.php
│   │   ├── Mapel.php
│   │   ├── Pembayaran.php
│   │   └── ...
│   ├── Policies/
│   │   ├── SiswaPolicy.php
│   │   └── PembayaranPolicy.php
│   └── Services/
│       ├── QRCodeService.php
│       ├── PDFService.php
│       └── WhatsAppService.php
├── database/
│   ├── migrations/
│   ├── seeders/
│   └── factories/
├── resources/
│   ├── views/
│   │   ├── layouts/
│   │   ├── admin/
│   │   ├── teacher/
│   │   ├── student/
│   │   └── auth/
│   ├── css/
│   └── js/
├── routes/
│   ├── web.php
│   └── api.php
├── storage/
│   ├── app/
│   │   └── public/
│   │       ├── students/
│   │       ├── employees/
│   │       └── documents/
│   └── logs/
└── public/
    ├── index.php
    └── assets/
```

### 9.3 Database Migration Plan

#### 9.3.1 Migration Priority Order

```
1. Users & Authentication
   - create_users_table
   - create_roles_table
   - create_role_user_table

2. Master Data
   - create_tahun_pelajars_table
   - create_kelas_table
   - create_siswas_table
   - create_pegawais_table
   - create_mapels_table
   - create_ruang_table

3. Academic Structure
   - create_jadwals_table
   - create_semesters_table

4. Transactions
   - create_pembayarans_table
   - create_tagihans_table
   - create_absensis_table
   - create_penilaians_table

5. Supporting Tables
   - create_ekstras_table
   - create_pelanggarans_table
   - create_prestasis_table
   - create_inventaris_table
```

#### 9.3.2 Sample Migration: Siswas

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('siswas', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->string('nis')->unique();
            $table->string('nama');
            $table->string('email')->nullable();
            $table->string('password'); // bcrypt
            $table->foreignUuid('kelas_id')->nullable();
            $table->foreignUuid('tahun_pelajar_id')->nullable();
            $table->integer('nourut')->default(0);
            $table->string('qrcode')->nullable();
            $table->string('nowa')->nullable();
            $table->string('parent_password')->nullable();
            
            // Counters (consider removing - use relationships instead)
            $table->integer('jml_ekstra')->default(0);
            $table->integer('jml_absen_sakit')->default(0);
            $table->integer('jml_absen_ijin')->default(0);
            $table->integer('jml_absen_alpha')->default(0);
            
            // Finance summary (consider removing - use aggregations)
            $table->decimal('subtotal_nominal', 15, 2)->default(0);
            $table->decimal('subtotal_setor', 15, 2)->default(0);
            $table->decimal('subtotal_belum', 15, 2)->default(0);
            
            // Behavior counters
            $table->integer('jml_pelanggaran')->default(0);
            $table->integer('subtotal_pelanggaran')->default(0);
            $table->integer('jml_presensi')->default(0);
            $table->integer('jml_prestasi')->default(0);
            $table->integer('subtotal_prestasi')->default(0);
            
            $table->timestamps();
            
            // Indexes
            $table->index('kelas_id');
            $table->index('tahun_pelajar_id');
            $table->index('nis');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('siswas');
    }
};
```

### 9.4 Model Relationships

```php
// app/Models/Siswa.php
class Siswa extends Model
{
    protected $keyType = 'string';
    public $incrementing = false;

    public function kelas()
    {
        return $this->belongsTo(Kelas::class);
    }

    public function tahunPelajar()
    {
        return $this->belongsTo(TahunPelajar::class);
    }

    public function pembayarans()
    {
        return $this->hasMany(Pembayaran::class);
    }

    public function nilais()
    {
        return $this->hasMany(Nilai::class);
    }

    public function absensis()
    {
        return $this->hasMany(Absensi::class);
    }

    public function pelanggarans()
    {
        return $this->hasMany(Pelanggaran::class);
    }

    public function prestasis()
    {
        return $this->hasMany(Prestasi::class);
    }

    public function extras()
    {
        return $this->belongsToMany(Ekstra::class, 'siswa_ekstra');
    }
}
```

### 9.5 Authentication Implementation

**Using Laravel Breeze + Custom Role System:**

```php
// app/Models/User.php
class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;

    protected $fillable = [
        'name',
        'email',
        'password',
        'role_id',
        'pegawai_id',
        'siswa_id',
    ];

    public function hasRole($role)
    {
        return $this->role->name === $role;
    }

    public function isAdmin()
    {
        return $this->hasRole('admin');
    }

    public function isTeacher()
    {
        return $this->hasRole('guru');
    }

    public function isStudent()
    {
        return $this->hasRole('siswa');
    }
}
```

**Middleware for Role Check:**

```php
// app/Http/Middleware/CheckRole.php
public function handle(Request $request, Closure $next, ...$roles)
{
    if (!$request->user() || !in_array($request->user()->role->name, $roles)) {
        abort(403, 'Unauthorized action.');
    }

    return $next($request);
}
```

**Route Definition:**

```php
// routes/web.php
Route::middleware(['auth'])->group(function () {
    
    // Admin Routes
    Route::middleware(['role:admin'])->prefix('admin')->group(function () {
        Route::get('/dashboard', [AdminDashboardController::class, 'index']);
        Route::resource('siswas', Admin\SiswaController::class);
        Route::resource('pegawais', Admin\PegawaiController::class);
        Route::resource('keuangan', Admin\FinanceController::class);
    });

    // Teacher Routes
    Route::middleware(['role:guru'])->prefix('teacher')->group(function () {
        Route::get('/dashboard', [TeacherDashboardController::class, 'index']);
        Route::resource('nilai', Teacher\NilaiController::class);
        Route::resource('absensi', Teacher\AbsensiController::class);
    });

    // Student Routes
    Route::middleware(['role:siswa'])->prefix('student')->group(function () {
        Route::get('/dashboard', [StudentDashboardController::class, 'index']);
        Route::get('/nilai', [Student\NilaiController::class, 'index']);
        Route::get('keuangan', [Student\FinanceController::class, 'index']);
    });
});
```

### 9.6 Service Classes

#### 9.6.1 QRCode Service

```php
// app/Services/QRCodeService.php
namespace App\Services;

use SimpleSoftware\QrCode\Facades\QrCode;

class QRCodeService
{
    public function generateForStudent(Siswa $siswa): string
    {
        $data = json_encode([
            'id' => $siswa->id,
            'nis' => $siswa->nis,
            'nama' => $siswa->nama,
        ]);

        return QrCode::size(200)->generate($data);
    }

    public function generateForPresence(string $type, string $location): string
    {
        $data = json_encode([
            'type' => $type,
            'location' => $location,
            'timestamp' => now()->timestamp,
        ]);

        return QrCode::size(300)->generate($data);
    }
}
```

#### 9.6.2 PDF Service

```php
// app/Services/PDFService.php
namespace App\Services;

use Barryvdh\DomPDF\Facade\Pdf;

class PDFService
{
    public function generateRaport(Siswa $siswa, int $semester)
    {
        $data = [
            'siswa' => $siswa,
            'nilais' => $siswa->nilais()->where('semester', $semester)->get(),
            'sikap' => $siswa->sikaps()->where('semester', $semester)->first(),
            'catatan' => $siswa->catatans()->where('semester', $semester)->first(),
        ];

        $pdf = Pdf::loadView('pdf.raport', $data);
        
        return $pdf->download("raport_{$siswa->nis}_sem_{$semester}.pdf");
    }

    public function generateNota(Pembayaran $pembayaran)
    {
        $data = [
            'pembayaran' => $pembayaran,
            'rincian' => $pembayaran->rincians,
        ];

        $pdf = Pdf::loadView('pdf.nota', $data);
        
        return $pdf->download("nota_{$pembayaran->nomor}.pdf");
    }
}
```

#### 9.6.3 WhatsApp Service

```php
// app/Services/WhatsAppService.php
namespace App\Services;

use Twilio\Rest\Client;

class WhatsAppService
{
    protected $twilio;

    public function __construct()
    {
        $this->twilio = new Client(
            config('services.twilio.sid'),
            config('services.twilio.token')
        );
    }

    public function sendTagihan(Siswa $siswa, Tagihan $tagihan)
    {
        $message = "Yth. Orang Tua {$siswa->nama},\n"
                 . "Terdapat tagihan sekolah:\n"
                 . "Jumlah: Rp " . number_format($tagihan->nominal) . "\n"
                 . "Jatuh tempo: {$tagihan->jatuh_tempo}\n"
                 . "Terima kasih.";

        return $this->sendMessage($siswa->nowa, $message);
    }

    public function sendMessage(string $phone, string $message)
    {
        return $this->twilio->messages->create(
            'whatsapp:' . $phone,
            [
                'from' => 'whatsapp:' . config('services.twilio.from'),
                'body' => $message,
            ]
        );
    }
}
```

---

## 10. RISK ASSESSMENT DAN MITIGATION

### 10.1 Risk Matrix

| Risk | Probability | Impact | Mitigation Strategy |
|------|-------------|--------|---------------------|
| Data Loss During Migration | Medium | 🔴 Critical | Full backup before migration, parallel run |
| Downtime During Cutover | High | 🟠 High | Schedule during holiday, prepare rollback plan |
| User Resistance | High | 🟡 Medium | Training sessions, user manuals, helpdesk |
| Performance Degradation | Low | 🟡 Medium | Load testing, optimization, caching |
| Security Vulnerabilities | Medium | 🔴 Critical | Security audit, penetration testing |
| Budget Overrun | Medium | 🟡 Medium | Agile approach, prioritize features |
| Timeline Delay | High | 🟡 Medium | Buffer time, phased delivery |
| Integration Issues | Medium | 🟠 High | API testing, sandbox environment |

### 10.2 Data Migration Strategy

**Approach: Dual-Write + Verification**

```
Step 1: Extract data from old database
        ↓
Step 2: Transform data (format conversion, validation)
        ↓
Step 3: Load into new database
        ↓
Step 4: Verify data integrity (checksums, counts)
        ↓
Step 5: Run parallel systems (read-only old, read-write new)
        ↓
Step 6: Cutover after verification period
```

**Data Migration Script Example:**

```php
// database/migrations/9999_migrate_old_data.php
public function up(): void
{
    $oldConnection = DB::connection('mysql_old');
    
    // Migrate Students
    $students = $oldConnection->table('m_siswa')->get();
    
    foreach ($students as $student) {
        Siswa::create([
            'id' => $student->kd,
            'nis' => $student->kode,
            'nama' => $student->nama,
            'password' => Hash::make($this->getTempPassword()), // Force reset
            'kelas_id' => $this->mapKelas($student->kelas),
            'nowa' => $student->nowa,
            'created_at' => $student->postdate,
        ]);
    }
    
    // Similar for other tables...
}
```

### 10.3 Rollback Plan

**Trigger Conditions:**
- Critical data corruption detected
- System unavailable > 4 hours
- Security breach identified

**Rollback Steps:**
1. Stop new system
2. Restore old database from backup
3. Redirect DNS/load balancer to old system
4. Notify users of temporary rollback
5. Investigate and fix issues
6. Reschedule migration

---

## 11. TIMELINE DAN ROADMAP IMPLEMENTASI

### 11.1 Gantt Chart Overview

```
Week  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16
      ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
Phase 1 [████████]
Phase 2         [████████████████]
Phase 3                     [████████████████]
Phase 4                                 [████████████████]
Phase 5                                             [████████]
```

### 11.2 Detailed Timeline

#### Phase 1: Foundation (2 weeks)

**Week 1:**
- Day 1-2: Laravel 11 installation & configuration
- Day 3-4: Database design & migration planning
- Day 5: Authentication setup (Breeze/Jetstream)

**Week 2:**
- Day 1-2: Role & permission system
- Day 3-4: Base layout & template migration
- Day 5: Environment setup (dev, staging, prod)

#### Phase 2: Core Modules (4 weeks)

**Week 3-4: Master Data**
- Siswa CRUD + Import/Export
- Pegawai CRUD + Import/Export
- Kelas, Mapel, Ruang management
- Tahun Pelajaran settings

**Week 5-6: User Management**
- Admin dashboard
- User profile management
- Password reset functionality
- Activity logging

#### Phase 3: Transaction Modules (4 weeks)

**Week 7-8: Finance**
- Payment processing
- Billing management
- Receipt generation
- WhatsApp notifications

**Week 9-10: Academic**
- Attendance (QR + manual)
- Grade input & calculation
- Schedule management
- Report card generation

#### Phase 4: Advanced Features (4 weeks)

**Week 11-12: Reporting**
- PDF generation (rapport, nota, etc.)
- Excel exports
- Dashboard analytics
- Custom report builder

**Week 13-14: Integration**
- QR Code system
- WhatsApp API
- File management
- Email notifications

#### Phase 5: Testing & Deployment (2 weeks)

**Week 15:**
- Unit testing (90% coverage target)
- Integration testing
- Performance testing
- Security audit

**Week 16:**
- UAT with real users
- Bug fixes
- Documentation
- Production deployment

### 11.3 Milestone Deliverables

| Milestone | Deliverable | Acceptance Criteria |
|-----------|-------------|---------------------|
| M1 (Week 2) | Foundation Complete | Auth working, DB migrated |
| M2 (Week 6) | Core Modules Ready | All CRUD operations functional |
| M3 (Week 10) | Transactions Live | Finance & academic modules tested |
| M4 (Week 14) | Features Complete | All features implemented |
| M5 (Week 16) | Go-Live | Production deployment successful |

---

## 12. APPENDICES

### Appendix A: File Inventory

**Total PHP Files: 1,675**

Breakdown by directory:
```
adm/          ~400 files
admgr/        ~150 files
admwk/        ~180 files
admbk/        ~120 files
admsw/        ~100 files
admbdh/       ~80 files
adminv/       ~90 files
admpiket/     ~70 files
admks/        ~200 files
inc/          ~50 files
Others        ~235 files
```

### Appendix B: Third-Party Libraries Used

| Library | Purpose | Location |
|---------|---------|----------|
| DOMPDF | PDF Generation | inc/class/dompdf/ |
| PHPQRCode | QR Code Generation | inc/class/phpqrcode/ |
| PhpSpreadsheet | Excel Export/Import | inc/class/phpspreadsheet/ |
| SimpleImage | Image Manipulation | inc/class/simpleimage/ |
| CKEditor | Rich Text Editor | inc/js/ckeditor/ |
| jQuery | JavaScript Library | inc/js/jquery/ |

### Appendix C: Configuration Files

**inc/config.php:**
```php
$xhostname = "localhost";
$xusername = "root";
$xpassword = "";
$xdatabase = "sisfokol_v7";
$hajirobe = "http://localhost/sisfokol";
```

### Appendix D: Common SQL Patterns

**Pattern 1: Select with Where**
```php
$q = mysqli_query($koneksi, "SELECT * FROM table WHERE kd = '$kd'");
$row = mysqli_fetch_assoc($q);
$total = mysqli_num_rows($q);
```

**Pattern 2: Insert**
```php
mysqli_query($koneksi, "INSERT INTO table(kd, field1, field2) VALUES ('$kd', '$val1', '$val2')");
```

**Pattern 3: Update**
```php
mysqli_query($koneksi, "UPDATE table SET field1 = '$val1' WHERE kd = '$kd'");
```

**Pattern 4: Delete**
```php
mysqli_query($koneksi, "DELETE FROM table WHERE kd = '$kd'");
```

### Appendix E: Security Recommendations

1. **Immediate Actions:**
   - Change all default passwords
   - Enable HTTPS
   - Update PHP to latest version
   - Change MD5 to bcrypt

2. **Short-term Improvements:**
   - Implement rate limiting
   - Add CSRF protection
   - Enable SQL prepared statements
   - Add input validation

3. **Long-term Goals:**
   - Complete Laravel migration
   - Regular security audits
   - Penetration testing
   - Security training for staff

### Appendix F: Glossary

| Term | Definition |
|------|------------|
| SISFOKOL | Sistem Informasi Sekolah |
| KIB | Kartu Inventaris Barang |
| RPP | Rencana Pelaksanaan Pembelajaran |
| BK | Bimbingan Konseling |
| QR Code | Quick Response Code |
| WA | WhatsApp |
| CRUD | Create, Read, Update, Delete |
| UAT | User Acceptance Testing |
| ORM | Object-Relational Mapping |
| MVC | Model-View-Controller |

---

## DOKUMEN TERKAIT

Dokumen ini merupakan bagian dari seri dokumentasi migrasi:

1. ✅ **01_Business_Process_Analysis.md** (Dokumen ini)
2. ⏳ **02_Technical_Architecture_Design.md** (Coming Next)
3. ⏳ **03_Database_Migration_Guide.md** (Coming Next)
4. ⏳ **04_API_Documentation.md** (Coming Next)
5. ⏳ **05_Testing_Strategy.md** (Coming Next)
6. ⏳ **06_Deployment_Runbook.md** (Coming Next)

---

**Dokumen Ini Adalah Properti Dari:**  
Tim Pengembangan Sistem Informasi Sekolah  

**Kontak:**  
Email: hajirodeon@gmail.com  
WA: 081-829-88-54  

**Last Updated:** Juli 2025  
**Version:** 1.0  

---

*Catatan: Dokumen ini disusun berdasarkan analisis mendalam terhadap kode sumber SISFOKOL v7.00. Semua temuan dan rekomendasi bersifat objektif dan dapat diverifikasi.*
