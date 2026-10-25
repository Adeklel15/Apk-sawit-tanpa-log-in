# Panduan Lengkap Konfigurasi & Pengujian Produksi Sawit-Ku
**Aplikasi Manajemen Kebun Kelapa Sawit — Karya Abdul Jalil**

Dokumen ini memuat panduan lengkap konfigurasi Firebase, aturan keamanan Firestore, daftar perubahan perbaikan autentikasi, serta checklist pengujian akun baru dan lama.

## 0. Konfigurasi Aktif yang Telah Diterapkan (Terkoneksi)

Proyek Firebase resmi telah dibuat dan dihubungkan secara otomatis ke aplikasi Sawit-Ku dengan parameter berikut:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSyB3cdTC4HQBELp_PkMx95E3LVOPxdrMbtw",
  authDomain: "omega-generator-4bcxw.firebaseapp.com",
  projectId: "omega-generator-4bcxw",
  firestoreDatabaseId: "ai-studio-sawitkuabduljali-5eb6cb74-c39d-48ab-9cc6-234ae464d09a",
  storageBucket: "omega-generator-4bcxw.firebasestorage.app",
  messagingSenderId: "641302379471",
  appId: "1:641302379471:web:0ba82f94e4e073e64230ba",
  collectionAppId: "sawit-ku-app"
};
```

*Status:* **Aktif & Terkoneksi Langsung**. Anda dapat langsung mendaftar atau masuk ke aplikasi (baik via Email/Password maupun Masuk Instan dengan Google).

---

## 1. Daftar Perubahan yang Diterapkan

1. **Keamanan Konfigurasi Firebase:**
   - Placeholder Firebase ditolak secara aman; aplikasi tidak berpura-pura masuk (tidak ada akun demo palsu).
   - Ditambahkan tombol dan modal **Pengaturan Konfigurasi Firebase** di dalam aplikasi. Pemilik dapat langsung menempelkan kode `firebaseConfig` atau mengisi kolom satu per satu, tersimpan otomatis di perangkat (`localStorage`).
   - Tidak ada private key, service-account, atau kredensial admin yang ditaruh di frontend.

2. **Validasi & Pengalaman Autentikasi:**
   - Pendaftaran email/password menerapkan validasi ketat: minimal 8 karakter, huruf besar (A-Z), huruf kecil (a-z), angka (0-9), dan simbol khusus.
   - Ditambahkan panel checklist visual interaktif di bawah kolom kata sandi yang menyala hijau saat kriteria terpenuhi.
   - Tombol "Lihat Sandi" / "Sembunyikan" untuk kenyamanan pengguna di kebun.
   - Fitur "Lupa Password?" menggunakan `sendPasswordResetEmail` Firebase resmi dengan pesan aman anti-enumerasi email (*"Jika email tersebut terdaftar, tautan reset telah dikirim..."*).
   - Pesan login salah menampilkan *"Email atau kata sandi tidak cocok. Periksa kembali keduanya..."* tanpa membocorkan status registrasi email.
   - Seluruh error Firebase Auth dan Firestore diterjemahkan ke dalam Bahasa Indonesia yang ramah dan solutif (tidak menampilkan `err.message` mentah).

3. **Integritas Data & Kompatibilitas Akun Lama:**
   - Path dokumen Firestore dipertahankan secara identik:
     - `artifacts/{appId}/users/{uid}/laporan_panen`
     - `artifacts/{appId}/users/{uid}/laporan_pupuk`
     - `artifacts/{appId}/users/{uid}/profile/info`
   - Default `appId` adalah `sawit-ku-app` (dapat disesuaikan jika proyek lama menggunakan custom appId).
   - Akun baru dimulai dengan data kosong / `0` (bukan data sampel 108 pohon). Data akun lama tidak pernah tertimpa nilai default.
   - Antrean data offline dipisahkan per-UID (`sawit_ku_pending_sync_${uid}`) dan menggunakan ID dokumen stabil (`crypto.randomUUID()`) agar sinkronisasi bersifat idempoten (tidak menduplikasi data).
   - Logout tidak menghapus data cloud maupun antrean lokal.

4. **Fitur Pengelolaan Data:**
   - Ditambahkan tombol **Hapus Laporan** dengan dialog konfirmasi sebelum penghapusan data panen maupun data perawatan, baik di cloud Firestore maupun cache lokal.
   - Perhitungan BJR, tonase kotor, ongkos panen, biaya perawatan, dan laba bersih (nett) otomatis diperbarui real-time.
   - Peringatan deteksi tandan hilang (*Theft Warning*) dengan perhitungan estimasi kerugian nominal Rupiah.
   - Ekspor Excel/CSV menggunakan BOM (`\ufeff`) dan pemisah titik koma (`;`) yang kompatibel dengan format Microsoft Excel di Indonesia.

5. **PWA & Visualisasi:**
   - Icon SVG resmi kebun sawit hijau tua (`#2D5A27`), `manifest.webmanifest`, dan `service-worker.js` untuk kemampuan offline dan instalasi ke layar utama HP (Add to Home Screen).
   - Simulasi 3D pohon sawit interaktif 360° menggunakan Three.js dengan reaksi warna daun terhadap dosis pupuk Nitrogen/Urea.
   - Grafik riwayat tren 12 bulan menggunakan Chart.js.

---

## 2. Langkah Konfigurasi Firebase (Pemilik Proyek)

Untuk menghubungkan aplikasi ke database produksi Anda:

### Langkah A: Buka Proyek Firebase Lama
1. Buka [Firebase Console](https://console.firebase.google.com/).
2. Masuk menggunakan akun Google Anda dan pilih **proyek Firebase yang sama** yang sebelumnya digunakan oleh Sawit-Ku agar data riwayat tidak hilang.

### Langkah B: Ambil Konfigurasi Web App
1. Klik ikon roda gigi ⚙️ di sebelah *Project Overview* &rarr; pilih **Project settings**.
2. Gulir ke bawah ke bagian **Your apps**.
3. Jika belum ada Web App, klik ikon `</>` (Web) dan beri nama `Sawit-Ku`.
4. Salin objek `firebaseConfig`, contoh:
   ```javascript
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "sawit-ku-xxxx.firebaseapp.com",
     projectId: "sawit-ku-xxxx",
     storageBucket: "sawit-ku-xxxx.firebasestorage.app",
     messagingSenderId: "1234567890",
     appId: "1:1234567890:web:abcdef..."
   };
   ```

### Langkah C: Aktifkan Authentication Email/Password
1. Di menu navigasi kiri Firebase Console, klik **Build** &rarr; **Authentication**.
2. Masuk ke tab **Sign-in method**.
3. Klik **Email/Password** &rarr; Aktifkan sakelar **Enable** pertama (Email/Password) &rarr; Klik **Save**.
*(Pilihan Email link tidak perlu diaktifkan).*

### Langkah D: Aktifkan Cloud Firestore Database
1. Di menu navigasi kiri, klik **Build** &rarr; **Firestore Database**.
2. Jika belum aktif, klik **Create database** &rarr; pilih lokasi region terdekat (misal `asia-southeast1` atau `asia-southeast2`) &rarr; mulai dalam *production mode*.

### Langkah E: Pasang Firestore Security Rules (Akses Khusus Pemilik)
1. Di halaman Firestore Database, buka tab **Rules**.
2. Ganti seluruh teks aturan dengan aturan owner-only berikut:
   ```text
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       // Akses hanya untuk pemilik subtree UID
       match /artifacts/{appId}/users/{userId}/{document=**} {
         allow read, write: if request.auth != null
                            && request.auth.uid == userId;
       }

       // Default deny untuk path lainnya
       match /{document=**} {
         allow read, write: if false;
       }
     }
   }
   ```
3. Klik **Publish** untuk menerapkan aturan.

### Langkah F: Daftarkan Authorized Domains
1. Buka **Authentication** &rarr; tab **Settings** &rarr; **Authorized domains**.
2. Pastikan domain tempat aplikasi berjalan telah terdaftar (misal domain Google AI Studio Preview / Cloud Run / GitHub Pages / custom domain Anda).

### Langkah G: Terapkan ke Aplikasi
1. Buka aplikasi Sawit-Ku di browser.
2. Di layar login, klik tombol **Konfigurasi Firebase**.
3. Tempelkan cuplikan `firebaseConfig` yang Anda salin dari Firebase Console &rarr; klik **Ekstrak Otomatis ke Formulir** &rarr; klik **Simpan & Hubungkan**.
4. Halaman akan dimuat ulang dan status akan langsung terhubung ke Firebase produksi Anda!

---

## 3. Checklist Pengujian (Testing Verification)

Lakukan pengujian berikut sebelum digunakan operasional di lapangan:

### 1. Uji Registrasi Akun Baru:
- [ ] Buka aplikasi, masukkan email baru (contoh: `petani1@sawit.com`).
- [ ] Ketik kata sandi pendek (misal `sawit`): perhatikan checklist persyaratan kata sandi tetap abu-abu dan pendaftaran ditolak dengan pesan yang ramah.
- [ ] Ketik kata sandi memenuhi syarat (misal `SawitKu#2026`): seluruh 5 indikator berubah hijau.
- [ ] Klik **Daftar Akun**: akun berhasil dibuat dan diarahkan ke dashboard utama.
- [ ] Periksa bahwa jumlah pohon awal adalah `0`, bukan angka sampel dummy.

### 2. Uji Pencatatan Panen & Perawatan:
- [ ] Masukkan data panen: 150 JJG, berat 2.100 Kg, harga Rp 2.500/Kg, upah panen Rp 300/Kg.
- [ ] Periksa live preview: BJR otomatis terhitung (14.0 Kg/JJG), kotor Rp 5.250.000, ongkos Rp 630.000, uang didapat Rp 4.620.000.
- [ ] Klik **SIMPAN LAPORAN PANEN**: data muncul di riwayat panen dan neraca keuangan.
- [ ] Buka tab **Halaman 4 (Perawatan)**: masukkan pemupukan Urea 1.5 Kg/pohon, biaya bahan Rp 800.000, upah Rp 200.000.
- [ ] Klik simpan: data muncul di riwayat perawatan dan otomatis mengurangi laba bersih di Halaman 2.

### 3. Uji Penghapusan Data:
- [ ] Klik ikon tempat sampah pada salah satu catatan panen.
- [ ] Dialog konfirmasi muncul. Jika dibatalkan, data tetap ada. Jika disetujui, data terhapus dari cloud dan neraca diperbarui.

### 4. Uji Isolasi Data (Keamanan Multi-Akun):
- [ ] Logout dari Akun A (`petani1@sawit.com`).
- [ ] Daftarkan Akun B (`petani2@sawit.com`).
- [ ] Masuk ke Akun B: pastikan riwayat panen dan neraca Akun B kosong (Akun B tidak dapat melihat data milik Akun A).
- [ ] Logout dan login kembali ke Akun A: seluruh riwayat data Akun A tetap utuh dan lengkap.

### 5. Uji Reset Password:
- [ ] Klik **Lupa Password?** pada layar login.
- [ ] Masukkan email terdaftar &rarr; pesan aman konfirmasi pengiriman muncul &rarr; periksa kotak masuk/spam email.

### 6. Uji Kemampuan Offline (PWA):
- [ ] Matikan koneksi internet (mode pesawat di HP atau DevTools offline).
- [ ] Indikator status di pojok kanan atas berubah menjadi *"Offline (Lokal)"*.
- [ ] Simpan satu catatan panen: muncul pemberitahuan bahwa data tersimpan di antrean offline.
- [ ] Nyalakan kembali koneksi internet: aplikasi otomatis menyinkronkan data offline ke cloud Firestore tanpa duplikasi.

---

## 4. Informasi Deployment

- **Format Aplikasi:** Web Application & Progressive Web App (PWA). Dapat dipasang langsung ke layar utama Android (Chrome) dan iPhone (Safari) tanpa melalui Play Store.
- **Penyimpanan:** Google Cloud Firestore (Database terdistribusi tingkat enterprise).
- **Karya:** Seluruh hak cipta dan hak cipta karya dilindungi atas nama **Abdul Jalil**.
