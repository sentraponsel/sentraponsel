# Sentra Ponsel v26 — Firebase Cloud Sync (offline-first)

Pembaruan berbasis build v25. Fitur pembukuan lama dipertahankan, dengan tambahan konfigurasi Firebase, login pemilik/kasir, sinkronisasi record-level Cloud Firestore, antrean offline Firestore, migrasi lokal eksplisit, serta log backup terpisah per perubahan.

## Yang perlu diketahui

- Aplikasi tetap menyimpan data lokal di browser. Saat login Firebase, data transaksi disinkronkan ke Firestore; SDK web menggunakan cache persisten IndexedDB untuk mendukung operasi offline dan mengirim perubahan saat online kembali.
- Sinkronisasi membutuhkan koneksi HTTPS dan konfigurasi Firebase milik pemilik toko. ZIP ini tidak menyertakan project Firebase atau kredensial pengguna.
- Konflik dua perangkat yang mengedit record ID yang sama memakai perilaku penulisan terakhir yang diterima server. Data transaksi baru umumnya memiliki ID unik; hindari mengedit record yang sama secara bersamaan.
- Setiap perubahan record dikirim ke koleksi `backups` sebagai snapshot terpisah. Backup Firestore ini memakai kuota database dan bukan layanan Backup bawaan Google. Unduh backup JSON lokal secara berkala juga tetap disarankan.
- Google Drive backup lama tetap tersedia secara terpisah dan masih membutuhkan konfigurasi OAuth Google jika digunakan.
- Tidak ada jaminan penggunaan selalu gratis jika kuota Spark terlampaui. Firestore backup/restore bawaan berbayar dan tidak dipakai oleh fitur ini.

## Setup Firebase (sekali oleh pemilik)

1. Buka https://console.firebase.google.com/ dan buat proyek Firebase.
2. Di Project settings, tambahkan aplikasi Web dan salin `apiKey`, `authDomain`, `projectId`, serta `appId` ke panel **Sinkronisasi Cloud Firebase** di aplikasi.
3. Authentication → Sign-in method → aktifkan **Email/Password**.
4. Buat akun email/password untuk pemilik dan setiap kasir di Authentication → Users. Jangan membagikan akun pemilik ke kasir.
5. Buat database Cloud Firestore dalam mode produksi.
6. Salin isi file `firestore.rules` ke tab Firestore → Rules lalu terbitkan.
7. Di Firestore Data, buat dokumen `stores/sentra-ponsel/members/UID_PEMILIK` dengan field `role` (string) = `owner`. Ganti `sentra-ponsel` bila Store ID yang dipilih berbeda. UID ditemukan pada Authentication → Users.
8. Buat dokumen untuk tiap kasir: `stores/sentra-ponsel/members/UID_KASIR`, field `role` (string) = `cashier`.
9. Di setiap HP, buka aplikasi melalui hosting HTTPS, masukkan konfigurasi Firebase dan Store ID yang sama, lalu login dengan akun masing-masing.
10. Login sebagai pemilik dan tekan **Gabungkan/migrasikan data lokal ke cloud** pada perangkat yang memegang data lokal paling lengkap. Pastikan sudah mengunduh backup JSON terlebih dahulu. Migrasi hanya menambahkan record lokal yang belum ada di cloud; record cloud dengan ID yang sama tidak ditimpa dan tidak ada record cloud yang dihapus.
11. Login di HP lain dan pastikan data tampil. Lakukan uji transaksi kecil di satu HP dan periksa HP lainnya sebelum mulai digunakan operasional.

## Peran

- **Pemilik (`owner`)**: dapat menulis semua koleksi, mengelola stok/paket, pengeluaran, pengaturan toko, dan mengakses log backup.
- **Kasir (`cashier`)**: dapat membaca data toko dan membuat/memperbarui transaksi barang, servis, serta pulsa. UI menyembunyikan aksi pemilik; Firestore Rules juga membatasi penulisan koleksi kasir.
- Peran disimpan di dokumen Firestore `stores/{storeId}/members/{uid}`. Mengubah label di UI saja tidak memberi hak akses; Firestore Rules adalah pengaman utama.

## Backup dan pemulihan

- Tombol **Unduh backup JSON sekarang** membuat backup lokal yang dapat dipindahkan ke tempat aman.
- Snapshot perubahan disimpan terpisah di `stores/{storeId}/backups`. Akses baca hanya untuk pemilik melalui aturan yang disertakan. Pemulihan dari snapshot cloud saat ini dilakukan secara terkontrol oleh pemilik melalui Firebase Console; jangan mengedit dokumen database secara manual tanpa memahami struktur data.
- Sebelum mengganti ZIP, selalu unduh backup JSON dari perangkat dengan data terbaru. Jangan menghapus data browser sebelum hasil migrasi dan sinkronisasi antarperangkat diuji.

## Batasan/tes

- Aplikasi ini tidak bisa membuat proyek Firebase, membuat akun, atau menerbitkan Security Rules otomatis; langkah setup di atas perlu dilakukan oleh pemilik.
- Sinkronisasi offline menggunakan cache persisten Firestore pada browser yang didukung. Jika browser menghapus data situs/IndexedDB, data lokal yang belum tersinkron dapat hilang; ekspor backup rutin sangat disarankan.
- Kuota Firebase Spark berlaku per proyek. Pantau penggunaan Firestore di Firebase Console.

## Instalasi/update

1. Unduh backup JSON di v25/v26 sebelum mengganti versi.
2. Unggah semua isi ZIP ini ke hosting HTTPS yang sama.
3. Buka ulang aplikasi/refresh. Service worker memakai cache `sentra-ponsel-v26`.
4. Data lama di localStorage tidak dihapus otomatis. Setelah login owner, gunakan tombol migrasi dan verifikasi perangkat kedua.
