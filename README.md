Perfect 🎯 — keputusan yang *sangat matang*!
Dengan kamu memasukkan semua 7 usulan tambahan ini, sistem kamu sekarang **naik kelas jadi versi “operasional penuh”** — artinya bukan cuma mencatat absensi, tapi benar-benar bisa **mewakili dinamika sekolah nyata.**

Jadi mulai sekarang kita punya satu paket fitur utuh yang sudah disepakati:
semua fitur inti + tambahan opsional sudah **masuk ke dalam rancangan sistem resmi proyek ini.**

---

## 🧩 **Rangkuman Fitur Lengkap Sistem Absensi Guru (Versi Pematangan Penuh)**

---

### 🧍‍♂️ **1. Manajemen Pengguna & Role**

* Role: **Superadmin**, **Guru Piket**, **Siswa**
* Guru piket bergantian setiap hari
* Superadmin bisa tambah/edit akun guru & siswa
* Siswa hanya bisa melihat status guru
* Penentuan guru piket harian oleh superadmin

---

### 🏫 **2. Absensi Kehadiran Guru di Kantor**

* Guru piket mencatat: Hadir / Izin / Sakit / Tugas luar / Tanpa keterangan
* Bisa diberi *catatan keterangan singkat*
* Auto-save setiap perubahan
* Bisa diubah selama hari berjalan
* Superadmin bisa lihat rekap
* Aktivitas tercatat di *log piket*

---

### 📚 **3. Absensi Guru Mengajar**

* Berdasarkan jadwal mapel & jam pelajaran
* Status: Mengajar / Tidak mengajar / Kosong
* Bisa isi *guru pengganti* jika guru utama absen
* Otomatis menyesuaikan kelas yang non-KBM
* Auto-save setiap update
* Masuk ke log guru piket
* Dilengkapi kolom *catatan* untuk keterangan

---

### 📅 **4. Jadwal Mapel & Guru Pengajar**

* Jadwal berisi: Hari, Kelas, Mapel, Guru, Jam ke-, Ruang
* Satu mapel bisa diajar oleh **banyak guru berbeda** tergantung kelasnya
  (contoh: Bu Jihan & Pak Fatur mengajar mapel PAI di kelas berbeda)
* Satu guru bisa mengajar di banyak kelas
* Pilih jam mulai & selesai (misal: jam ke-2 s.d ke-3)
* Otomatis menghitung waktu berdasarkan pengaturan durasi jam
* Semua perubahan jadwal masuk log

---

### 🏷️ **5. Pengaturan KBM**

* Menentukan apakah suatu kelas aktif KBM atau tidak
* Non-KBM (misal PKL, kunjungan, acara) → tidak perlu isi absensi mengajar
* Bisa diatur oleh Superadmin atau Guru Piket
* Tercatat di log

---

### 🧭 **6. Pengaturan Sekolah (Upacara & Jam Pelajaran)**

* **Durasi 1 jam pelajaran (menit)** bisa diubah Superadmin
* **Jam mulai sekolah** diatur agar waktu otomatis sinkron
* **Status upacara (hari Senin)** bisa diubah oleh Superadmin dan Guru Piket

  * Jika aktif → jam pertama jadi “Upacara”
  * Jadwal otomatis menyesuaikan
* Semua perubahan tersimpan di log

---

### 🧾 **7. Log Aktivitas Guru Piket**

* Mencatat semua perubahan:

  * Siapa yang ubah
  * Apa yang diubah
  * Waktu perubahan
* Hanya Superadmin yang bisa melihat log
* Tidak bisa dihapus atau diekspor

---

### 📊 **8. Dashboard**

* **Superadmin:**

  * Rekap kehadiran guru & KBM aktif
  * Statistik bulanan
  * Log aktivitas guru piket
* **Guru Piket:**

  * Status absensi guru hari ini
  * Daftar jadwal mengajar per jam
  * Status KBM & upacara hari ini
* **Siswa:**

  * Cek status guru pengajar di kelasnya (mengajar / izin / tidak hadir)

---

### ✨ **9. Tambahan Fitur Baru (Disetujui)**

| No | Fitur Baru                     | Akses                  | Fungsi Utama                                                |
| -- | ------------------------------ | ---------------------- | ----------------------------------------------------------- |
| 1  | **Riwayat Absensi Guru**       | Superadmin, Guru       | Lihat riwayat kehadiran & mengajar per guru                 |
| 2  | **Catatan Keterangan Absensi** | Guru Piket             | Menulis alasan singkat izin/sakit/tugas                     |
| 3  | **Guru Pengganti**             | Guru Piket, Superadmin | Tandai siapa yang menggantikan guru absen                   |
| 4  | **Notifikasi Internal**        | Superadmin, Guru Piket | Pengingat jika ada kelas non-KBM, guru belum hadir, dll     |
| 5  | **Filter & Pencarian Data**    | Semua role             | Cari data absensi berdasarkan kelas, guru, tanggal          |
| 6  | **Mode Hari Esok**             | Superadmin             | Siapkan jadwal & piket untuk hari berikutnya                |
| 7  | **Kalender Akademik**          | Superadmin             | Tandai hari libur, ujian, acara — otomatis nonaktif absensi |

---

### ⚙️ **Teknologi Disepakati**

* Backend: PHP 8+ Native (Prosedural) + PDO
* Database: MySQL
* Frontend: HTML5, CSS3, Bootstrap 5, JavaScript (ES6+)
* Server: Apache (Docker)
* Deployment: Docker & Docker Compose

---

🔥 Jadi sekarang status sistem kamu:

> ✅ **Sudah 100% matang secara fitur (versi konseptual)**
> Belum tahap implementasi — tapi semua logika & fungsinya sudah lengkap untuk sistem sekolah profesional.

---

Langkah selanjutnya supaya makin mantap:

1. **🧠 Tahap 1: Desain struktur data (ERD)**

   * Menentukan tabel, relasi, dan dependensi antar fitur (terutama yang multi-guru & multi-kelas).
2. **🎨 Tahap 2: Wireframe UI**

   * Menentukan alur tampilan dan posisi menu tiap role.
3. **📘 Tahap 3: Dokumentasi Fitur Teknis**

   * Penjelasan logika tiap fitur sebelum masuk ke kode.

---

Kamu ingin kita lanjut ke **desain ERD (relasi data)** dulu,
atau mau bahas dulu **alur UI (tampilan dan menu per role)** biar bisa kebayang bagaimana pengguna pakainya?
