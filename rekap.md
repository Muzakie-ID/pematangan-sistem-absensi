# Rekap Lengkap & Rinci — Sistem Absensi Guru

Dokumen ini merangkum **semua yang kamu minta**: rekap 12 fitur (dengan aturan bisnis), gambaran halaman + komponen UI per halaman, struktur folder & file proyek, alur sistem (flow) untuk role, dan struktur data + relasi tabel (skema). Ditulis rinci supaya bisa langsung diserahkan ke tim dev / desainer.

---

## Bagian A — Rincian 12 Fitur (Detail + Business Rules / Acceptance Criteria)

> Untuk tiap fitur: tujuan singkat, siapa akses, tabel/kolom terkait utama, aturan bisnis penting, acceptance criteria.

### 1. Manajemen Pengguna & Role

* **Tujuan:** Kelola akun pengguna dan otorisasi role.
* **Akses:** Superadmin (CRUD penuh). Guru Piket hanya dipilih/diset oleh superadmin; siswa view-only (tanpa akun).
* **Tabel terkait:** `users`
* **Aturan bisnis:**

  * Password disimpan hashed (bcrypt).
  * Role minimal: `superadmin`, `guru_piket`.
  * Session-based auth; middleware cek role per halaman.
  * Superadmin dapat menetapkan guru piket harian.
* **Acceptance:** Superadmin dapat create/update/delete user, set role, login terpisah dengan redirect ke dashboard sesuai role.

### 2. Absensi Kehadiran Guru (Kantor)

* **Tujuan:** Catat status kehadiran guru pada hari tertentu.
* **Akses:** Input = Guru Piket; View & Rekap = Superadmin.
* **Tabel:** `absensi_guru`
* **Field penting:** `id_guru`, `tanggal`, `status` (hadir/izin/sakit/tugas_luar/alpha), `keterangan`, `id_user_input`, `waktu_input`.
* **Aturan bisnis:**

  * Auto-save via AJAX; perubahan hanya diperbolehkan untuk `tanggal = hari ini` (bisa diubah sebelum pergantian hari).
  * Keterangan wajib untuk status tertentu (mis. `tugas_luar`).
  * Jika tidak ada entri = dianggap belum diisi (bukan otomatis alpha, tergantung kebijakan).
* **Acceptance:** Semua perubahan tersimpan otomatis, dapat diroll-back dalam hari yang sama, rekap oleh superadmin.

### 3. Absensi Mengajar (per Kelas & Jam)

* **Tujuan:** Catat apakah guru mengajar di jam/jadwal yang terdaftar.
* **Akses:** Input = Guru Piket; View = Superadmin; Siswa (cek kelas).
* **Tabel:** `absensi_mengajar` (refer `jadwal_mapel`)
* **Field penting:** `id_jadwal`, `tanggal`, `status` (mengajar/tidak_mengajar/kosong), `id_guru_pengganti`, `keterangan`, `id_user_input`.
* **Aturan bisnis:**

  * Hanya entri untuk `tanggal = hari ini` (kecuali mode hari esok untuk persiapan).
  * Jika kelas non-KBM pada rentang jam → jadwal diabaikan / ditandai `kosong`.
  * Jika guru pengganti diisi → simpan `id_guru_pengganti` & tetap catat guru utama sebagai absen.
* **Acceptance:** GUI menampilkan jadwal hari ini & memungkinkan set status per jadwal; log tercatat.

### 4. Jadwal Mapel & Pembagian Guru

* **Tujuan:** Simpan jadwal pelajaran (hari, jam-ke range, kelas, mapel, guru, ruang).
* **Akses:** Superadmin CRUD; Guru Piket view.
* **Tabel:** `jadwal_mapel`, `mapel`, `kelas`, `guru`
* **Aturan bisnis:**

  * Satu jadwal = kombinasi unik (hari + kelas + jam range).
  * Satu mapel dapat diajar oleh banyak guru, tergantung kelas; satu guru dapat mengajar banyak kelas.
  * Validasi bentrok jam di kelas sama disarankan (peringatan).
* **Acceptance:** Superadmin dapat menambah/edit/hapus jadwal; perubahan langsung memengaruhi tampilan absensi.

### 5. Pengaturan Status KBM (per Kelas/Tgl/Jam)

* **Tujuan:** Menandai kelas non-KBM (PKL, kunjungan) sehingga absensi tidak wajib.
* **Akses:** Superadmin & Guru Piket.
* **Tabel:** `status_kbm`
* **Field penting:** `id_kelas`, `tanggal`, `jam_mulai`, `jam_selesai`, `alasan`, `id_user_input`.
* **Aturan bisnis:**

  * Alasan wajib.
  * Ketika non-KBM aktif → UI absensi mengajar menyembunyikan/menandai jadwal terkait.
* **Acceptance:** Menandai non-KBM mempengaruhi absensi mengajar otomatis.

### 6. Pengaturan Sekolah (Durasi Jam & Upacara)

* **Tujuan:** Simpan durasi jam pelajaran, jam mulai, dan pengaturan upacara Senin.
* **Akses:** Superadmin (full). Guru Piket boleh toggle upacara hari itu.
* **Tabel:** `pengaturan_sekolah`, `pengaturan_harian`
* **Aturan bisnis:**

  * `pengaturan_sekolah` menyimpan `durasi_jam_menit`, `jam_mulai_hari`.
  * `pengaturan_harian` untuk override harian (`upacara = ya/tidak`) dicatat siapa set.
  * Jika upacara aktif → jam ke-1 dialokasikan Upacara; atau jam bergeser sesuai kebijakan.
* **Acceptance:** Superadmin ubah durasi/jam mulai; guru piket toggle upacara saat hari Senin; perubahan tercatat.

### 7. Log Aktivitas Guru Piket

* **Tujuan:** Audit trail semua perubahan oleh guru piket.
* **Akses:** View-only Superadmin.
* **Tabel:** `log_aktivitas` (id_user, aksi, detail, waktu, ip_address, user_agent)
* **Aturan bisnis:**

  * Catat semua aksi yang memodifikasi data penting (absensi, KBM, upacara, jadwal).
  * Tidak boleh diexport (keputusan).
* **Acceptance:** Semua perubahan akut tercatat; superadmin bisa filter/cari.

### 8. Dashboard & Rekap Data

* **Tujuan:** Ringkasan untuk monitoring & quick-action.
* **Akses:** Superadmin (komprehensif), Guru Piket (ringkas), Siswa (cek kelas).
* **Fitur:** card summary, recent log, quick links, notifikasi.
* **Acceptance:** Data ringkasan akurat & link ke fungsi detail.

### 9. Riwayat Absensi Guru (Personal Log)

* **Tujuan:** Lihat histori absensi & mengajar per guru.
* **Akses:** Superadmin (semua); opsi: guru (lihat miliknya).
* **Sumber:** `absensi_guru`, `absensi_mengajar`.
* **Acceptance:** Bisa filter per tanggal & export (jika policy mengizinkan).

### 10. Catatan Keterangan Absensi

* **Tujuan:** Simpan keterangan alasan khusus.
* **Akses:** Input = Guru Piket; View = Superadmin.
* **Implementasi:** `keterangan` pada tabel absensi.
* **Acceptance:** Keterangan tampil di laporan & log.

### 11. Notifikasi Internal & Filter/Pencarian

* **Tujuan:** Alerts & mempermudah pencarian data.
* **Akses:** Superadmin & Guru Piket; filter untuk semua yang view.
* **Contoh notifikasi:** kelas non-KBM hari ini; guru belum diabsen melewati threshold.
* **Acceptance:** Alerts muncul di dashboard; filter berfungsi.

### 12. Export ke Excel (.xlsx) — Direct Download

* **Tujuan:** Export dataset (stream download, tidak disimpan server).
* **Akses:** Superadmin (semua); Guru Piket (absensi harian sesuai policy).
* **Format:** `.xlsx` via PhpSpreadsheet; header stream `php://output`.
* **Tidak di-export:** `log_aktivitas`.
* **Acceptance:** File terdownload langsung sesuai filter; kolom sesuai template.

---

## Bagian B — Gambaran Halaman & Komponen UI (per halaman, elemen rinci)

> Header/Sidebar/Footer = layout global (logo, nama user + role, menu navigasi role-based, tombol logout). Berikut komponen khusus tiap halaman.

### 1. Login

* Form: [username/email], [password]
* Buttons: Login
* Links: Lupa password (opsional)
* Pesan error / success

### 2. Dashboard Superadmin

* Top cards: `Total Hadir`, `Izin`, `Sakit`, `Alpa`
* Card: `Total kelas non-KBM hari ini`
* Chart area: grafik mingguan (opsional)
* Recent Log (table kecil): columns [waktu, user, aksi singkat]
* Quick actions: tombol Export, Manage Users, Manage Jadwal

### 3. Dashboard Guru Piket

* Banner tanggal & jam
* Toggle upacara (visible on Monday) + info siapa set
* Quick buttons: Absensi Kehadiran, Absensi Mengajar, Set KBM
* Summary mini: jumlah guru belum diabsen, kelas non-KBM
* Alerts area (notifikasi)

### 4. Absensi Kehadiran Guru

* Date picker (default hari ini)
* Table columns:

  * No
  * Nama Guru (link ke riwayat)
  * Status (radio/select: Hadir / Izin / Sakit / Tugas Luar / Alpha)
  * Keterangan (inline input)
  * Indikator auto-save (status per baris)
* Filter: tampilkan hanya yg belum diisi
* Note: "Bisa diubah sampai 23:59 hari ini" (configurable)

### 5. Absensi Mengajar (per Kelas)

* Date picker
* Kelas dropdown (memilih kelas menampilkan schedule)
* Schedule table (per jam):

  * Jam ke (1,2..)
  * Waktu (07:00–07:45)
  * Mapel
  * Guru terjadwal
  * Status KBM (badge)
  * Action: Toggle "Guru Mengajar" (checkbox), Dropdown "Guru Pengganti", Keterangan
* Auto-save per aksi
* Info jika jam itu Upacara atau non-KBM

### 6. Jadwal Mapel

* Filters: Hari, Kelas, Guru
* Tabel jadwal (hari, jam_ke range, mapel, guru, ruang, aksi)
* Form tambah/edit:

  * Pilih Hari
  * Pilih Kelas
  * Pilih Jam ke (dari..sampai) — multi-select
  * Pilih Mapel
  * Pilih Guru (single)
  * Field Ruang, Catatan

### 7. Data Guru

* Tabel CRUD: NIP, Nama, Mapel utama, Username, Status aktif
* Button: Tambah Guru (form modal)
* Aksi: edit / nonaktifkan / reset password

### 8. Kelas

* Tabel CRUD: Kode, Nama, Jurusan, Tingkat, Wali Kelas, Aktif
* Tambah / edit modal

### 9. Mapel

* Tabel CRUD: Kode, Nama, Kategori (Umum/Kejuruan)
* Tambah/Edit modal

### 10. Setting

* Form:

  * Durasi per JP (menit)
  * Jam mulai hari (time picker)
  * Default upacara Senin (yes/no)
  * Tombol Save
* Info: perubahan global berlaku mulai sekarang atau esok (jelaskan kebijakan)

### 11. Log Aktivitas

* Tabel: Tanggal/Waktu, Nama User, Aksi, Detail (expandable), IP
* Filter: tanggal range, user
* Read-only, no export

### 12. Export UI

* Pilih jenis data (radio)
* Filter: tanggal from/to, kelas, guru (opsional)
* Tombol Export → trigger streaming download

---

## Bagian C — Struktur Folder & File (Disarankan, modular)

```
project_absensi_guru/
├── app/
│   ├── config/
│   │   ├── database.php
│   │   ├── app.php
│   │   └── auth.php
│   ├── controllers/
│   │   ├── AuthController.php
│   │   ├── DashboardController.php
│   │   ├── UserController.php
│   │   ├── GuruController.php
│   │   ├── KelasController.php
│   │   ├── MapelController.php
│   │   ├── JadwalController.php
│   │   ├── AbsensiKantorController.php
│   │   ├── AbsensiKelasController.php
│   │   ├── StatusKbmController.php
│   │   ├── SettingController.php
│   │   ├── LogController.php
│   │   └── ExportController.php
│   ├── models/
│   │   ├── UserModel.php
│   │   ├── GuruModel.php
│   │   ├── KelasModel.php
│   │   ├── MapelModel.php
│   │   ├── JadwalModel.php
│   │   ├── AbsensiGuruModel.php
│   │   ├── AbsensiMengajarModel.php
│   │   ├── StatusKbmModel.php
│   │   ├── SettingModel.php
│   │   └── LogModel.php
│   ├── helpers/
│   │   ├── auth_helper.php
│   │   ├── date_helper.php
│   │   └── response_helper.php
│   └── views/
│       ├── layouts/ (header.php, sidebar.php, footer.php)
│       ├── auth/ (login.php)
│       ├── dashboard/ (superadmin.php, gurupiket.php)
│       ├── users/, guru/, kelas/, mapel/, jadwal/, absensi_kantor/, absensi_kelas/, setting/, log/, export/
├── public/
│   ├── assets/ (css/, js/, img/)
│   ├── index.php
│   └── .htaccess
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yml
├── vendor/ (composer libraries: PhpSpreadsheet, autoload)
├── .env
├── composer.json
└── README.md
```

**Catatan:** `ExportController` akan men-stream Excel ke `php://output`. Hapus atau jangan buat folder `storage/export/` untuk hasil export agar tidak menyimpan file.

---

## Bagian D — Alur Sistem (Flow per Role; event sequence)

### Alur harian (Guru Piket)

1. Login → dashboard/gurupiket.
2. Jika hari Senin → ada control toggle upacara (bisa di-set) → simpan ke `pengaturan_harian` + `log_aktivitas`.
3. Buka **Absensi Kehadiran** → tampilan daftar guru → menandai status tiap guru → setiap perubahan AJAX → simpan ke `absensi_guru` → simpan `log_aktivitas`.
4. Buka **Absensi Mengajar** → pilih kelas → UI menampilkan jadwal hari ini (`jadwal_mapel` + `jam_pelajaran` + `status_kbm`) → centang `mengajar` atau pilih `guru_pengganti` → simpan ke `absensi_mengajar` + `log_aktivitas`.
5. Set `status_kbm` jika diperlukan → berdampak ke tampilan absensi mengajar.
6. Gunakan Export (jika berhak) untuk download rekap harian.

### Alur Superadmin

1. Login → dashboard/superadmin.
2. CRUD master data (guru, kelas, mapel).
3. CRUD jadwal (`jadwal_mapel`) — perubahan memengaruhi absensi.
4. Lihat `log_aktivitas`.
5. Export laporan via controller (stream).

### Alur Siswa

1. Akses publik (tanpa login) halaman cek kelas.
2. Pilih kelas → tampil jadwal hari ini + status guru (mengajar/izin/tidak hadir/kelas non-KBM/upacara).

---

## Bagian E — Struktur Data & Relasi (Skema tabel + FK, penjelasan relasi)

> Berikut skema tabel utama (ringkasan kolom), lalu relasi antar tabel.

### Tabel inti (ringkasan kolom)

* **users**

  * `id_user` PK, `username`, `password_hash`, `role`, `nama`, `email`, `aktif`
* **guru**

  * `id_guru` PK, `nama`, `nip`, `kontak`, `mapel_default`
* **kelas**

  * `id_kelas` PK, `kode`, `nama_kelas`, `jurusan`, `tingkat`, `aktif`
* **mapel**

  * `id_mapel` PK, `nama_mapel`, `kategori`
* **jam_pelajaran**

  * `id_jam` PK, `jam_ke` INT, `waktu_mulai` TIME, `waktu_selesai` TIME
* **pengaturan_sekolah**

  * `key`, `value` (durasi_jam_menit, jam_mulai, dst)
* **pengaturan_harian**

  * `tanggal` PK, `upacara` ENUM, `id_user_set`, `waktu_set`
* **jadwal_mapel**

  * `id_jadwal` PK, `hari`, `id_kelas` FK, `id_mapel` FK, `id_guru` FK, `jam_ke_mulai` INT, `jam_ke_selesai` INT, `ruang`, `catatan`
* **status_kbm**

  * `id`, `id_kelas` FK, `tanggal`, `jam_mulai`, `jam_selesai`, `alasan`, `id_user_input`, `waktu_input`
* **absensi_guru**

  * `id_absen`, `id_guru` FK, `tanggal`, `status`, `keterangan`, `id_user_input`, `waktu_input`
* **absensi_mengajar**

  * `id`, `id_jadwal` FK, `tanggal`, `status`, `id_guru_pengganti` FK nullable, `keterangan`, `id_user_input`, `waktu_input`
* **log_aktivitas**

  * `id_log`, `id_user` FK, `aksi`, `detail`, `waktu`, `ip_address`, `user_agent`

### Relasi utama (konseptual)

* `users.id_user` ←→ `log_aktivitas.id_user` (1:N)
* `guru.id_guru` ←→ `absensi_guru.id_guru` (1:N)
* `kelas.id_kelas` ←→ `jadwal_mapel.id_kelas` (1:N)
* `mapel.id_mapel` ←→ `jadwal_mapel.id_mapel` (1:N)
* `guru.id_guru` ←→ `jadwal_mapel.id_guru` (1:N) — mencatat guru yang mengajar pada jadwal tertentu
* `jadwal_mapel.id_jadwal` ←→ `absensi_mengajar.id_jadwal` (1:N)
* `kelas.id_kelas` ←→ `status_kbm.id_kelas` (1:N)
* `users.id_user` ←→ `pengaturan_harian.id_user_set` (1:N) — siapa set upacara

### Index & Performance

* Tambahkan index: `absensi_guru(tanggal)`, `absensi_guru(id_guru)`, `absensi_mengajar(tanggal)`, `jadwal_mapel(hari)`, `status_kbm(tanggal)`.
* Gunakan pagination untuk tabel besar (log & rekap).

---

## Bagian F — Endpoints / Controller Mapping (contoh route naming)

* `GET  /login` → AuthController::showLogin()
* `POST /login` → AuthController::login()
* `GET  /dashboard/superadmin` → DashboardController::superadmin()
* `GET  /dashboard/gurupiket` → DashboardController::gurupiket()
* `GET  /users` → UserController::index()
* `POST /users/create` → UserController::create()
* `GET  /absensi/kantor?date=` → AbsensiKantorController::list()
* `POST /absensi/kantor/save` → AbsensiKantorController::save()` (AJAX)
* `GET  /absensi/kelas?date=&kelas=` → AbsensiKelasController::list()
* `POST /absensi/kelas/save` → AbsensiKelasController::save()` (AJAX)
* `GET  /jadwal` `POST /jadwal/create` `PUT /jadwal/update` `DELETE /jadwal/delete`
* `POST /status_kbm/set`
* `GET  /log` → LogController::index()
* `GET  /export?type=&from=&to=` → ExportController::exportXlsx()

---

## Bagian G — Kebijakan Operasional & Acceptance untuk Tim

* **Timezone:** server set ke `Asia/Jakarta`. Semua tanggal/waktu pakai timezone ini.
* **Auto-save:** implementasikan debounce (mis. 500-800ms) untuk mengurangi jumlah request.
* **Locking tanggal:** perubahan hanya diperbolehkan untuk `tanggal = hari ini` kecuali fitur Mode Hari Esok dibuat.
* **Keamanan:** semua query gunakan prepared statements (PDO). Cek authorization di tiap controller.
* **Retention Log:** simpan log minimal 90 hari; opsional cron job untuk pembersihan.
* **Export:** stream download, jangan simpan file di server.

---

## Bagian H — Rekomendasi Praktis untuk Implementasi Tim

1. **Sprint pertama (MVP):** Auth + Manajemen User + Absensi Kehadiran + Dashboard Guru Piket + log simple.
2. **Sprint kedua:** Jadwal Mapel + jam_pelajaran + Absensi Mengajar (per kelas).
3. **Sprint ketiga:** Status KBM, Pengaturan Sekolah (upacara), Log lengkap, Export.
4. **QA:** UAT bersama guru piket & satu kepala sekolah; test kasus: hari Senin upacara, kelas non-KBM, guru pengganti.

---

