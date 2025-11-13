# Spesifikasi Fitur — Sistem Absensi Guru (Ringkasan Detail untuk Tim Pengembang)

Berikut adalah dokumen spesifikasi fitur lengkap dan terperinci (12 modul) yang disepakati untuk proyek **Sistem Absensi Guru**. Dokumen ini ditulis supaya bisa langsung diserahkan ke tim pengembang sebagai acuan requirement sebelum implementasi.

---

## Ringkasan singkat

* Teknologi: **PHP 8+ (Native, Prosedural) + PDO**, **MySQL**, **HTML5/CSS3/Bootstrap5**, **JS (ES6+)**
* Deployment: **Apache** dalam **Docker & Docker Compose**
* Library export: **PhpSpreadsheet** (untuk .xlsx)
* Role: **Superadmin**, **Guru Piket**, **Siswa (view only)**

---

# 1. Manajemen Pengguna & Role

**Tujuan:** Kelola akun dan peran (role) pengguna sistem.

**Akses:** Superadmin (CRUD penuh). Guru Piket bisa digunakan setelah dibuat oleh superadmin. Siswa tidak perlu akun (akses publik).

**Fungsi:**

* Tabel `users` (id_user, username, password_hash, role, nama, email, aktif).
* Superadmin dapat: tambah/edit/hapus user (guru, guru piket), reset password, menonaktifkan akun.
* Pengaturan guru piket harian: superadmin menentukan siapa bertugas hari tertentu (opsional: menu pilih tanggal → pilih user sebagai piket).
* Auto-session login/logout menggunakan PHP Session.
* Password disimpan hashed (bcrypt/password_hash).

**UI Notes:**

* Halaman manajemen user: tabel, tombol Tambah/Edit/Nonaktif.
* Validation: unique username/email.

**Acceptance criteria:**

* Superadmin dapat membuat user baru, login, dan menonaktifkan user.
* Sesi login benar memisahkan role (akses redirect ke dashboard role masing-masing).

---

# 2. Absensi Kehadiran Guru di Kantor

**Tujuan:** Mencatat kehadiran guru saat masuk kantor setiap hari.

**Akses:** Guru Piket (input), Superadmin (view & rekap).

**Fungsi:**

* Tabel `absensi_guru` (id_absen, id_guru, tanggal, status ENUM[hadir, izin, sakit, tugas_luar, alpha], keterangan, id_user_input, waktu_input).
* Auto-save: setiap perubahan langsung dikirim AJAX ke backend dan disimpan.
* Boleh diubah sepanjang hari berjalan (sistem menolak edit untuk tanggal < hari ini).
* Catatan keterangan (text) optional untuk setiap entri.
* Rekap harian & filter by tanggal/guru tersedia di Superadmin.

**Business rules:**

* Hanya guru terdaftar yang bisa diabsen.
* Jika belum ada entri hari itu, default = `alpha` atau kosong sampai diisi.

**UI Notes:**

* Halaman daftar guru dengan kolom status (toggle/select), field keterangan inline.
* Indikator auto-save (spinner kecil atau toast).

**Acceptance criteria:**

* Guru Piket dapat menandai status semua guru hari ini; perubahan tersimpan otomatis.
* Superadmin bisa melihat rekap filterable.

---

# 3. Absensi Guru Mengajar (per Kelas & Jam)

**Tujuan:** Mencatat apakah guru mengajar atau tidak pada jam dan kelas sesuai jadwal.

**Akses:** Guru Piket (input), Superadmin (view & rekap), Siswa (view limited).

**Fungsi:**

* Tabel `absensi_mengajar` (id, id_jadwal_mapel, tanggal, status ENUM[mengajar, tidak_mengajar, kosong], id_guru_pengganti (nullable), keterangan, id_user_input, waktu_input).
* Basis data: referensi ke `jadwal_mapel` (lihat modul Jadwal).
* Auto-save per sel/jam di UI.
* Jika kelas non-KBM untuk tanggal/ jam tersebut → tidak muncul sebagai entri yang harus diisi (atau muncul tapi status auto = `kosong` dengan flag non-KBM).
* Mendukung pencatatan guru pengganti (id_guru_pengganti) bila guru utama absen.

**Business rules:**

* Input hanya untuk tanggal = hari ini (kecuali fitur Mode Hari Esok untuk persiapan, tetapi tidak mempengaruhi rekap hari ini sampai tanggal tersebut tiba).
* Jika guru pengganti diisi → entri menyimpan dua informasi: guru utama tetap `tidak hadir` (atau status sesuai absensi kehadiran), guru pengganti tercatat sebagai `mengajar`.

**UI Notes:**

* Tampilan jadwal hari ini per jam per kelas, checkbox/toggle per jadwal.
* Kolom catatan & dropdown untuk pilih guru pengganti.

**Acceptance criteria:**

* Guru Piket dapat menandai status mengajar per jadwal; proper handling ketika kelas non-KBM dan support guru pengganti.

---

# 4. Jadwal Mapel & Pembagian Guru

**Tujuan:** Menyimpan jadwal pelajaran lengkap (hari, jam ke, kelas, mapel, guru, ruang). Menangani kasus multi-guru untuk satu mapel di kelas berbeda.

**Akses:** Superadmin (CRUD jadwal), Guru Piket (view).

**Fungsi:**

* Tabel `mapel` (id_mapel, nama_mapel).
* Tabel `kelas` (id_kelas, nama_kelas, jurusan, tingkat).
* Tabel `guru` (id_guru, nama, nip, dsb).
* Tabel `jadwal_mapel` (id_jadwal, hari ENUM, id_kelas, id_mapel, id_guru, jam_ke_mulai INT, jam_ke_selesai INT, ruang, catatan).

  * `jam_ke_mulai` dan `jam_ke_selesai` merujuk ke nomor jam (1..N). Waktu aktual dihitung dari pengaturan jam pelajaran.
* Support: satu mapel di banyak kelas dengan guru yang berbeda (masing-masing entri unik berdasarkan kelas & jam).
* Fitur edit / delete / duplicate (copy jadwal untuk hari/kelas lain).

**Business rules:**

* Validasi bentrok: opsional (di versi awal bisa peringatan manual; di versi lanjut dapat dicek otomatis).
* Jika jadwal diubah → pengaruh ke modul absensi mengajar secara langsung.

**UI Notes:**

* Form tambah jadwal: pilih Hari, Kelas, Mapel, Guru, Jam ke mulai & selesai (multi-select jam ke), Ruang.
* Tabel jadwal per hari dengan filter Hari/Kelas/Guru.

**Acceptance criteria:**

* Superadmin dapat membuat jadwal terperinci; setiap jadwal memetakan mapel+kelas+guru+jam.

---

# 5. Pengaturan Status KBM (per Kelas per Hari/Jam)

**Tujuan:** Menandai kelas menjadi non-KBM (karena kunjungan, PKL, acara) sehingga absensi mengajar untuk kelas tersebut tidak wajib/diabaikan.

**Akses:** Superadmin & Guru Piket.

**Fungsi:**

* Tabel `status_kbm` (id, id_kelas, tanggal, jam_ke_mulai, jam_ke_selesai, alasan, id_user_input, waktu_input).
* Jika status non-KBM aktif untuk rentang jam tertentu → sistem:

  * Menyembunyikan/menandai jadwal sebagai non-KBM pada UI absensi mengajar,
  * Tidak menghitung guru sebagai `tidak_mengajar` untuk jadwal tersebut.
* Catatan alasan wajib diisi.

**UI Notes:**

* Form singkat: pilih Kelas, Tanggal, Jam ke range, alasan (dropdown + text).
* Indikator di jadwal hari itu (warna, badge NON-KBM).

**Acceptance criteria:**

* Guru Piket atau Superadmin bisa menandai kelas non-KBM; absensi mengajar menyesuaikan otomatis.

---

# 6. Pengaturan Sekolah (Durasi Jam & Upacara)

**Tujuan:** Menyimpan pengaturan jam pelajaran, jam mulai, dan opsi upacara hari Senin.

**Akses:** Superadmin (full), Guru Piket (bisa toggle Upacara untuk hari Senin).

**Fungsi:**

* Tabel `pengaturan_sekolah` (durasi_jam_menit, jam_mulai_hari, default_upacara_hari_senin (boolean), dst).
* Tabel `pengaturan_harian` (tanggal, upacara ENUM[ya,tidak], id_user_set, waktu_set) untuk override harian.
* Jika upacara aktif pada suatu Senin:

  * Sistem menandai jam ke-1 sebagai `Upacara`.
  * Opsi: jam ke-1 di-skip/dialihkan; jadwal jam-ke bergeser sesuai kebijakan.
* Durasi jam dipakai untuk menghitung waktu aktual tiap jam ke.

**UI Notes:**

* Halaman pengaturan: input numerik durasi jam, jam mulai, tombol Save.
* Toggle upacara muncul di dashboard Guru Piket pada hari Senin (modal konfirmasi). Aksi tercatat di log.

**Acceptance criteria:**

* Superadmin dapat mengubah durasi & jam mulai. Guru Piket bisa set upacara hari Senin dan tercatat.

---

# 7. Log Aktivitas Guru Piket

**Tujuan:** Mencatat semua aksi guru piket untuk audit (siapa ubah apa dan kapan).

**Akses:** Hanya Superadmin (view-only), tidak dapat diekspor.

**Fungsi:**

* Tabel `log_aktivitas` (id_log, id_user, aksi, detail TEXT, waktu TIMESTAMP, ip_address, user_agent).
* Setiap tindakan yang mengubah data (absensi hadir, absensi mengajar, set non-KBM, set upacara, ubah jadwal) memicu penyimpanan log.
* Log tidak dapat dihapus oleh UI; penghapusan otomatis bisa diatur (misal > 90 hari) sebagai task terpisah.

**UI Notes:**

* Halaman log: filter tanggal, filter user (guru piket), search text.
* Tampilkan waktu, nama user, ringkasan aksi, detail (klik untuk expand).

**Acceptance criteria:**

* Semua perubahan yang penting menghasilkan entri log; Superadmin dapat mencari & memfilter log.

---

# 8. Dashboard & Rekap Data

**Tujuan:** Menyediakan ringkasan cepat bagi Superadmin & Guru Piket; akses terbatas untuk siswa.

**Akses:** Superadmin, Guru Piket, Siswa (terbatas).

**Fungsi Superadmin:**

* Ringkasan hari ini: jumlah guru hadir/izin/sakit/tidak, jumlah kelas non-KBM, notifikasi (kelas non-KBM, guru belum diabsen).
* Statistik bulanan: grafik kehadiran per bulan, rekap jam mengajar.
* Quick access ke Export (Excel) untuk dataset yang diizinkan.

**Fungsi Guru Piket:**

* Tampilkan daftar guru hari ini dengan status singkat, tombol cepat ke halaman Absensi Kehadiran & Absensi Mengajar.
* Toggle upacara (Senin), set kelas non-KBM.

**Fungsi Siswa:**

* Pilih kelas → lihat jadwal hari ini & status guru (mengajar/izin/tidak hadir/kelas non-KBM/upacara).
* View-only tanpa login.

**UI Notes:**

* Gunakan kartu ringkasan (cards), tabel, dan chart sederhana (misal chart.js jika diperlukan).
* Notifikasi kecil (badge) di header jika ada action required.

**Acceptance criteria:**

* Dashboard menampilkan data real-time (dari DB) dan memudahkan navigasi ke fungsi terkait.

---

# 9. Riwayat Absensi Guru (Personal Log)

**Tujuan:** Memungkinkan guru (jika nanti memiliki akun) dan Superadmin melihat sejarah kehadiran & mengajar.

**Akses:** Superadmin (lihat semua), Guru (lihat miliknya saja — opsional implementasi). Guru Piket tidak memodifikasi riwayat selain input biasa.

**Fungsi:**

* Halaman riwayat: filter tanggal (range), filter status, export (jika permitted).
* Data diambil dari `absensi_guru` dan `absensi_mengajar`.

**UI Notes:**

* Tampilkan tabel dengan pagination & opsi cari.

**Acceptance criteria:**

* Superadmin dan (opsional) guru dapat melihat riwayat sesuai hak akses.

---

# 10. Catatan Keterangan Absensi

**Tujuan:** Menyimpan detail keterangan saat status selain hadir dipilih.

**Akses:** Guru Piket (input), Superadmin (view).

**Fungsi:**

* Field `keterangan` pada `absensi_guru` dan `absensi_mengajar`.
* Keterangan wajib saat memilih alasan tertentu? (misal: alasan tugas luar wajib keterangan).
* Keterangan ikut tersimpan di log aktivitas.

**UI Notes:**

* Field input kecil di samping status (modal atau inline).

**Acceptance criteria:**

* Keterangan disimpan & bisa dilihat di rekap & log.

---

# 11. Notifikasi Internal & Filter/Pencarian

**Tujuan:** Mempermudah operasional harian dengan notifikasi ringan dan fungsi pencarian.

**Akses:** Superadmin & Guru Piket (notifikasi), Semua role sesuai hak akses (filter/pencarian).

**Fungsi notifikasi:**

* Alert di dashboard bila:

  * Ada kelas non-KBM hari ini.
  * Ada guru yang belum diabsen melewati threshold waktu (misal jam 2 belum diisi hadir).
  * Perubahan upacara/KBM baru saja dibuat.
* Notifikasi tidak perlu push/email; cukup UI alerts.

**Fungsi filter/pencarian:**

* Filter berdasarkan tanggal, guru, kelas, mapel.
* Search text across nama guru, kelas, keterangan.

**UI Notes:**

* Filter bar di halaman rekap & jadwal.
* Toast/alert untuk notifikasi.

**Acceptance criteria:**

* Notifikasi tampil sesuai kondisi; filter & search berfungsi seefisien mungkin.

---

# 12. Export ke Excel (.xlsx)

**Tujuan:** Memungkinkan download data penting dalam format Excel.

**Akses & Scope:**

* Superadmin: export semua dataset yang diizinkan.
* Guru Piket: export absensi harian yang ia input (opsional; sesuai kebijakan).
* Log aktivitas: **TIDAK** dapat diexport (sesuai keputusan).

**Fungsi:**

* Export dataset:

  * Absensi Guru Harian (tanggal range, per guru/per kelas).
  * Absensi Mengajar (per tanggal/jadwal).
  * Rekap Kehadiran Bulanan.
  * Jadwal Mapel.
  * Kalender Akademik.
* Implementasi: gunakan **PhpSpreadsheet**; streamed download (php://output) — tidak menyimpan file permanen di server.
* Header kolom disepakati per jenis export (lihat contoh di bawah).

**Example Header (Absensi Harian):**
`Tanggal | Nama Guru | NIP | Kelas | Status Kehadiran | Keterangan | Dicatat Oleh | Waktu Input`

**UI Notes:**

* Tombol Export pada halaman rekap/daftar (dengan modal pilih tanggal / filter).
* Konfirmasi download & akses control.

**Acceptance criteria:**

* Export menghasilkan file .xlsx yang dapat dibuka di MS Excel/LibreOffice, sesuai filter yang dipilih, dan hanya oleh role yang berhak.

---

## Catatan Teknis Tambahan & Asumsi

1. **Zona waktu:** Tetapkan zona waktu server (mis. Asia/Jakarta). Semua operasi tanggal/waktu mengacu ke sana.
2. **Auto-save:** Gunakan AJAX `fetch()` dengan debouncing (mis. 500ms) dan indikator success/error.
3. **Keamanan:** Prepared statements (PDO) untuk semua query; sanitasi input; session handling.
4. **Retention log:** Log aktivitas disimpan minimal 90 hari (bisa dikonfigurasi). Hapus otomatis via cron job (opsional).
5. **Mode Hari Esok:** Superadmin dapat menyiapkan data untuk tanggal mendatang tanpa memengaruhi tanggal hari ini. (Opsional implementasi awal).
6. **Backup DB:** Rencana backup & restore DB (tidak termasuk implementasi awal, tetapi direkomendasikan).
7. **Dokumentasi API/Internal:** Meski PHP Native, sediakan file dokumentasi endpoint (nama file, param, method) untuk memudahkan dev.

---

## Lampiran — Contoh Skema Tabel (ringkasan)

* `users` (id_user, username, password_hash, role, nama, email, aktif)
* `guru` (id_guru, nama, nip, ... )
* `kelas` (id_kelas, nama_kelas, jurusan, tingkat)
* `mapel` (id_mapel, nama_mapel)
* `jadwal_mapel` (id_jadwal, hari, id_kelas, id_mapel, id_guru, jam_ke_mulai, jam_ke_selesai, ruang, catatan)
* `jam_pelajaran` (id_jam, jam_ke, waktu_mulai, waktu_selesai) / atau pengaturan global durasi_jam + jam_mulai_hari
* `absensi_guru` (id_absen, id_guru, tanggal, status, keterangan, id_user_input, waktu_input)
* `absensi_mengajar` (id, id_jadwal, tanggal, status, id_guru_pengganti, keterangan, id_user_input, waktu_input)
* `status_kbm` (id, id_kelas, tanggal, jam_mulai, jam_selesai, alasan, id_user_input, waktu_input)
* `pengaturan_sekolah` (key, value)
* `pengaturan_harian` (tanggal, upacara, id_user_set, waktu_set)
* `log_aktivitas` (id_log, id_user, aksi, detail, waktu, ip, agent)

---
##📁 Struktur Folder Sistem Absensi Guru (PHP Native + PDO)
```
project_absensi_guru/
│
├── 📁 app/
│   ├── 📁 config/
│   │   ├── database.php        # Koneksi PDO ke MySQL
│   │   ├── app.php             # Konfigurasi global (timezone, session, dsb)
│   │   └── auth.php            # Middleware otentikasi dan role
│   │
│   ├── 📁 controllers/
│   │   ├── AuthController.php          # Login, logout
│   │   ├── DashboardController.php     # Halaman utama per role
│   │   ├── GuruController.php          # CRUD data guru
│   │   ├── JadwalController.php        # CRUD jadwal mapel
│   │   ├── AbsensiKantorController.php # Absensi guru hadir/izin/sakit
│   │   ├── AbsensiKelasController.php  # Absensi guru mengajar di kelas
│   │   ├── KelasController.php         # CRUD data kelas
│   │   ├── MapelController.php         # CRUD data mata pelajaran
│   │   ├── SettingController.php       # Setting upacara, durasi jam pelajaran
│   │   ├── LogController.php           # Menampilkan log aktivitas
│   │   └── ExportController.php        # Export data ke Excel
│   │
│   ├── 📁 models/
│   │   ├── Guru.php
│   │   ├── Jadwal.php
│   │   ├── AbsensiKantor.php
│   │   ├── AbsensiKelas.php
│   │   ├── Kelas.php
│   │   ├── Mapel.php
│   │   ├── Setting.php
│   │   └── Log.php
│   │
│   ├── 📁 helpers/
│   │   ├── functions.php        # Fungsi umum (format tanggal, jam, dsb)
│   │   ├── auth_helper.php      # Fungsi cek login dan role
│   │   └── response_helper.php  # Fungsi untuk JSON/AJAX
│   │
│   └── 📁 views/
│       ├── 📁 layouts/
│       │   ├── header.php
│       │   ├── sidebar.php
│       │   ├── footer.php
│       │
│       ├── 📁 auth/
│       │   └── login.php
│       │
│       ├── 📁 dashboard/
│       │   ├── superadmin.php
│       │   └── gurupiket.php
│       │
│       ├── 📁 guru/
│       │   └── index.php
│       │
│       ├── 📁 jadwal/
│       │   ├── index.php
│       │   └── form.php
│       │
│       ├── 📁 absensi_kantor/
│       │   └── index.php
│       │
│       ├── 📁 absensi_kelas/
│       │   └── index.php
│       │
│       ├── 📁 kelas/
│       │   └── index.php
│       │
│       ├── 📁 mapel/
│       │   └── index.php
│       │
│       ├── 📁 setting/
│       │   └── index.php
│       │
│       ├── 📁 log/
│       │   └── index.php
│       │
│       └── 📁 export/
│           └── index.php
│
├── 📁 public/
│   ├── 📁 assets/
│   │   ├── css/
│   │   ├── js/
│   │   └── img/
│   ├── index.php                # Entry point (include controller)
│   └── .htaccess                # Routing mod_rewrite
│
├── 📁 storage/
│   ├── logs/                    # Log aktivitas (jika disimpan file)
│   └── export/                  # Hasil export Excel
│
├── 📁 docker/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── php.ini
│
├── .env                         # Konfigurasi environment (DB, base URL)
├── composer.json                # Jika ingin autoloading namespace
├── README.md                    # Dokumentasi proyek
└── LICENSE                      # Lisensi
```

======================================================================================================================================


 **rancangan komponen per halaman (versi awal / draft 1)** untuk sistem absensi guru ini 👇

---

## 🧩 1. **Halaman Login**

**Tujuan:** autentikasi semua role (Superadmin, Guru Piket, Siswa)

**Komponen:**

* Input `Username / Email`
* Input `Password`
* Tombol `Login`
* Pesan error (jika username/password salah)
* Footer kecil berisi nama sistem & versi

---

## 🧩 2. **Dashboard Superadmin**

**Tujuan:** memantau seluruh aktivitas sistem

**Komponen utama:**

* Ringkasan harian:

  * Total guru hadir / izin / sakit / alpa
  * Total guru yang mengajar hari ini
  * Total kelas aktif KBM
* Statistik singkat (chart):

  * Grafik kehadiran guru mingguan
  * Grafik aktivitas guru piket (berapa kali update)
* Tabel log aktivitas terbaru:

  * Kolom: `Nama Guru Piket`, `Aksi`, `Waktu`
  * Dibatasi 10-20 entri terakhir
* Tombol “Lihat Semua Log” → ke halaman log lengkap

---

## 🧩 3. **Dashboard Guru Piket**

**Tujuan:** akses cepat ke dua mode absensi (kantor dan kelas)

**Komponen:**

* Banner info hari & tanggal (contoh: *Kamis, 13 November 2025*)
* Status “Upacara Hari Ini” (switch aktif/tidak)
* Tombol besar (2 menu utama):

  * `Absensi Guru Hadir Kantor`
  * `Absensi Guru Mengajar di Kelas`
* Ringkasan singkat:

  * Jumlah guru hadir / izin / sakit
  * Jumlah kelas aktif / non-KBM
* Peringatan kalau ada data belum diisi hari itu

---

## 🧩 4. **Halaman Absensi Guru di Kantor**

**Tujuan:** mencatat kehadiran guru (hadir, izin, sakit, alpa)

**Komponen:**

* Dropdown pilih tanggal (default: hari ini)
* Tabel guru:

  * Kolom: `No`, `Nama Guru`, `Status Kehadiran`, `Keterangan`
  * Pilihan status:

    * ✅ Hadir
    * 🚫 Tidak Hadir
    * 💊 Sakit
    * 🏖️ Izin
  * Kolom “Keterangan” (input teks opsional)
  * Semua perubahan **auto-save tanpa tombol Submit**
* Notifikasi kecil “Data tersimpan otomatis”
* Tombol filter (opsional): “Tampilkan hanya yang belum diisi”

---

## 🧩 5. **Halaman Absensi Guru Mengajar di Kelas**

**Tujuan:** mencatat apakah guru benar-benar mengajar di jam pelajarannya

**Komponen:**

* Pilihan tanggal (default hari ini)
* Dropdown `Kelas` → menampilkan semua kelas aktif hari itu
* Tabel per kelas:

  * Kolom: `Jam Ke`, `Mapel`, `Guru`, `Status KBM`, `Guru Mengajar`
  * Switch “KBM Aktif” (kalau kelas sedang kegiatan seperti PKL → nonaktif)
  * Switch “Guru Mengajar” (diaktifkan manual oleh guru piket)
* Semua update **auto-save**
* Info “Data ini masih bisa diubah sampai pergantian hari”

---

## 🧩 6. **Halaman Jadwal Mapel**

**Tujuan:** mengatur jadwal pelajaran per kelas, termasuk guru & jam pelajaran

**Komponen:**

* Dropdown `Kelas`
* Tombol `Tambah Jadwal`
* Form input jadwal:

  * Pilih `Hari`
  * Pilih `Jam ke-berapa (dari – sampai)`
  * Pilih `Mapel`
  * Pilih `Guru` (bisa multi-guru)
* Tabel jadwal per kelas:

  * Kolom: `Hari`, `Jam Ke`, `Mapel`, `Guru`, `Aksi (edit/delete)`
* Tombol `Export Jadwal (Excel)` untuk superadmin
* Validasi agar jam tidak bentrok di kelas yang sama

---

## 🧩 7. **Halaman Data Guru**

**Tujuan:** mengelola data seluruh guru

**Komponen:**

* Tombol `Tambah Guru`
* Kolom data:

  * `NIP`, `Nama`, `Mapel`, `Status`, `Username`, `Aksi`
* Fitur pencarian dan filter
* Aksi: edit / hapus guru
* (Opsional) import Excel untuk menambah guru massal

---

## 🧩 8. **Halaman Kelas**

**Tujuan:** mengelola data kelas & jurusan

**Komponen:**

* Tombol `Tambah Kelas`
* Kolom: `Kode`, `Nama Kelas`, `Jurusan`, `Wali Kelas`, `Aksi`
* (Opsional) checkbox aktif/nonaktif untuk kelas tertentu (misal sudah lulus)

---

## 🧩 9. **Halaman Mapel**

**Tujuan:** mengelola daftar mata pelajaran

**Komponen:**

* Tombol `Tambah Mapel`
* Kolom: `Kode`, `Nama Mapel`, `Kategori (Umum / Kejuruan)`, `Aksi`
* Bisa dikaitkan ke banyak guru

---

## 🧩 10. **Halaman Setting**

**Tujuan:** mengatur parameter sistem global

**Komponen:**

* Durasi jam pelajaran (misal: `45 menit`)
* Waktu mulai sekolah (misal: `07:00`)
* Switch “Hari Senin ada Upacara?”
* Tombol `Simpan` (manual save)
* Info kecil “Perubahan akan diterapkan mulai hari berikutnya”

---

## 🧩 11. **Halaman Log Aktivitas**

**Tujuan:** memantau siapa yang mengubah data dan kapan

**Komponen:**

* Tabel log:

  * Kolom: `Tanggal`, `Nama Guru Piket`, `Aksi`, `Deskripsi`, `Waktu`
* Filter per tanggal atau guru
* Tidak ada tombol export (hanya view)
* Tombol `Hapus Log Lama (>30 hari)` (khusus superadmin)

---

## 🧩 12. **Halaman Export Data**

**Tujuan:** export laporan absensi ke Excel

**Komponen:**

* Pilih jenis data:

  * Absensi Guru Kantor
  * Absensi Mengajar
  * Jadwal Pelajaran
* Filter rentang tanggal
* Tombol `Export ke Excel`
* File tersimpan ke `/storage/export/` + notifikasi berhasil

---

