AaAi KUITANSI - Aplikasi Manajemen Booking & Kuitansi Digital

Aplikasi web single-page (SPA) untuk manajemen booking dan pembuatan kuitansi/faktur digital. Dibangun menggunakan HTML, Tailwind CSS, dan Firebase (Firestore & Authentication) dengan fungsionalitas admin dan pelacakan afiliasi.

Fitur Utama

Sistem Login: Autentikasi pengguna penuh (Daftar & Login) menggunakan Firebase Authentication.

Manajemen Booking (CRUD): Buat, baca, perbarui (fitur masa depan), dan hapus data booking.

Database Real-time: Daftar booking diperbarui secara otomatis menggunakan listener onSnapshot Firestore.

Ekspor PDF: Membuat kuitansi profesional dalam format PDF (menggunakan jsPDF & html2canvas).

Sistem PRO (Berbayar):

Pengguna "Free" memiliki watermark pada PDF yang dihasilkan.

Pengguna "PRO" tidak memiliki watermark.

Sistem Masa Aktif: Akun PRO memiliki masa aktif 1 tahun dan akan otomatis kembali ke "Free" setelah kedaluwarsa.

Panel Admin:

Panel admin khusus yang hanya muncul untuk email admin.

Memungkinkan admin untuk meng-upgrade pengguna "Free" menjadi "PRO".

Sistem Afiliasi (Voucher):

Panel admin dapat melacak upgrade menggunakan kode voucher (misal: JANA, ADJIE).

Data voucher disimpan di Firestore (voucherUsed) untuk pelacakan komisi manual.

Kustomisasi Kuitansi:

Admin dapat menyimpan info usaha (nama, alamat, kontak, logo) di localStorage untuk kop surat.

Admin dapat menyimpan tanda tangan digital untuk kuitansi.

Teknologi yang Digunakan

Frontend: HTML5, Tailwind CSS, JavaScript (ES6 Modules)

Backend (BaaS): Firebase

Authentication: Untuk login email/password.

Firestore: Database NoSQL real-time untuk menyimpan data booking dan profil pengguna.

Utility:

jsPDF: Untuk membuat dokumen PDF.

html2canvas: Untuk mengubah HTML (kuitansi) menjadi gambar/canvas.

SignaturePad: Untuk tanda tangan digital.

Panduan Setup (Wajib)

Aplikasi ini tidak akan berfungsi tanpa konfigurasi Firebase yang benar.

1. Buat Proyek Firebase

Buat proyek baru di Firebase Console.

2. Aktifkan Layanan

Authentication: Aktifkan penyedia Email/Password.

Firestore: Buat Database Firestore (mulai dalam mode produksi/terkunci).

3. Pasang Aturan Keamanan (Paling Penting)

Ini adalah inti dari keamanan aplikasi. Buka Firestore Database > Rules (Aturan) dan ganti semua aturannya dengan kode di bawah ini:

rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Fungsi helper untuk mengecek apakah pengguna adalah admin
    function isAdmin() {
      // Cek apakah email ada di 'email' ATAU di 'token.email'
      // dan bandingkan dengan email admin (setelah diubah ke huruf kecil)
      let email = request.auth.email;
      let tokenEmail = request.auth.token.email;
      
      return request.auth != null &&
             ( (email != null && email.toLowerCase() == "janawhilayu@gmail.com") ||
               (tokenEmail != null && tokenEmail.toLowerCase() == "janawhilayu@gmail.com") );
    }

    // Path untuk data PROFIL PENGGUNA
    match /artifacts/{appId}/users/{userId} {
      
      // Pengguna bisa membaca data mereka sendiri
      allow read: if request.auth != null && request.auth.uid == userId;
      
      // HANYA ADMIN yang bisa MENULIS (meng-upgrade)
      allow write: if isAdmin();
    }

    // Path untuk data BOOKING
    match /artifacts/{appId}/users/{userId}/bookings/{document=**} {
      
      // Pengguna bisa mengelola booking mereka sendiri
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}


4. Konfigurasi Aplikasi (index.html)

Agar aplikasi dapat terhubung ke proyek Firebase Anda, Anda harus mengedit file index.html.

Dapatkan Konfigurasi Anda: Di Firebase Console, buka Project Settings (Setelan Proyek) dan temukan Firebase SDK snippet (Cuplikan SDK) untuk Config (Konfigurasi).

Edit index.html: Buka file index.html dan cari bagian autentikasiDanInisialisasi (sekitar baris 1630). Ganti objek firebaseConfig fallback dengan konfigurasi Anda:

// ...
const firebaseConfig = (typeof __firebase_config !== 'undefined' && __firebase_config)
    ? JSON.parse(__firebase_config) // Gunakan config dari Canvas
    : { 
        // V V V GANTI SEMUA DI BAWAH INI V V V
          apiKey: "AIzaSy...",
          authDomain: "proyek-anda.firebaseapp.com",
          projectId: "proyek-anda",
          storageBucket: "proyek-anda.appspot.com",
          messagingSenderId: "123456789",
          appId: "1:123456789:web:abcdef12345"
      };
// ...


Atur Email Admin Anda: Di bagian atas tag <script> (sekitar baris 1134), ganti email admin agar Panel Admin muncul untuk Anda.

// (BARU) Tentukan email admin di sini
const ADMIN_EMAIL = "janawhilayu@gmail.com"; // <-- GANTI DENGAN EMAIL ADMIN ANDA


(Catatan: Email ini harus sama dengan yang Anda atur di Aturan Keamanan).

Program Afiliasi (Voucher)

Sistem ini mendukung pelacakan komisi untuk influencer/afiliasi.

Kode Voucher Publik: Kode voucher (misal: JANA) didefinisikan secara publik di index.html (sekitar baris 1517) dalam array VOUCHER_VALID.

Cara Kerja: Saat pengguna memasukkan kode yang valid, mereka melihat harga promo dan mengirim pesan WA ke Admin.

Pelacakan (Admin): Saat Admin meng-upgrade pengguna melalui Panel Admin, Admin memasukkan kode voucher yang digunakan. Kode ini disimpan di field voucherUsed di Firestore.

Menghitung Komisi: Untuk menghitung komisi, Admin dapat memfilter koleksi users di Firebase Console dengan kueri: voucherUsed == "JANA".
