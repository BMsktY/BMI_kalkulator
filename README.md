# bodyfit — Kalkulator BMI & Kebugaran Harian

bodyfit adalah aplikasi web interaktif berbasis *Single-Page Application* (SPA) untuk menghitung Indeks Massa Tubuh (BMI), memantau riwayat kesehatan, dan menjalankan tantangan kebugaran harian berdasarkan kategori BMI.

Aplikasi ini dibuat dengan HTML5, CSS3 responsif, JavaScript ES Modules, Firebase Authentication, Cloud Firestore, dan Chart.js.

## Fitur Utama

- **Autentikasi pengguna:** pendaftaran dan login menggunakan email dan kata sandi melalui Firebase Authentication, termasuk fitur tampilkan/sembunyikan kata sandi.
- **Profil pengguna:** menyimpan nama lengkap, jenis kelamin, dan tanggal lahir; menghitung usia secara otomatis; serta menyediakan fitur edit profil.
- **Kalkulator BMI:** menghitung BMI dari berat badan (kg) dan tinggi badan (cm), lalu menampilkan kategori kurus, ideal, berlebih, atau obesitas.
- **Siluet dinamis:** menampilkan ilustrasi karakter pria atau wanita dengan ukuran yang menyesuaikan skor BMI.
- **Rekomendasi kebugaran:** menyajikan skala BMI berwarna dan kartu rekomendasi dalam tata letak grid 2 × 2.
- **Tantangan harian:** menawarkan aktivitas olahraga yang dapat diterima atau ditolak, dicentang, dan disimpan ke Firestore.
- **Grafik dan riwayat:** menampilkan lima perhitungan BMI terbaru dan menyinkronkan data dengan Firestore. Penghapusan entri riwayat juga menghapus task terkait.
- **Desain responsif:** mendukung tampilan desktop dan ponsel serta dapat ditambahkan ke layar utama melalui fitur *Add to Home Screen*.

## Teknologi

| Bagian | Teknologi |
| --- | --- |
| Antarmuka | HTML5, CSS3 (Flexbox, Grid, dan CSS Variables) |
| Logika aplikasi | JavaScript ES Modules |
| Ikon | Phosphor Icons (CDN) |
| Visualisasi data | Chart.js (CDN) |
| Backend-as-a-Service | Google Firebase v11.6.1 (Authentication dan Cloud Firestore) |

## Alur Kerja Aplikasi

### 1. Inisialisasi dan sesi pengguna

Firebase App, Authentication, dan Firestore diinisialisasi melalui CDN. `onAuthStateChanged` memeriksa status login pengguna.

- Jika pengguna sudah login, aplikasi mengambil profil dari `artifacts/{appId}/users/{uid}/profile/info`, menampilkan sapaan, memuat riwayat BMI, dan mengaktifkan listener real-time.
- Jika pengguna belum login, aplikasi menampilkan modal autentikasi.

### 2. Autentikasi dan profil

Pengguna dapat beralih antara mode masuk dan daftar. Saat mendaftar, aplikasi membuat akun menggunakan `createUserWithEmailAndPassword` dan menyimpan nama lengkap, jenis kelamin, tanggal lahir, serta usia ke Firestore. Pesan kesalahan autentikasi ditampilkan melalui notifikasi toast.

### 3. Perhitungan BMI dan tantangan harian

Pengguna memasukkan berat badan dalam kilogram dan tinggi badan dalam sentimeter. BMI dihitung dengan rumus:

$$
\text{BMI} = \frac{\text{Berat Badan (kg)}}{\left(\frac{\text{Tinggi Badan (cm)}}{100}\right)^2}
$$

Setelah perhitungan, aplikasi menyimpan data ke koleksi `history`, memperbarui skala siluet SVG, dan menampilkan modal tantangan olahraga harian. Pengguna dapat menerima tantangan untuk menambahkannya ke tab **Task Saya**, atau menolaknya.

### 4. Pengelolaan dan penyimpanan task

Di tab **Task Saya**, pengguna dapat mencentang aktivitas yang telah dilakukan. Tombol **Kumpulkan Task / Simpan Permanen** membuka modal konfirmasi. Setelah disetujui, task disimpan ke koleksi `tasks` dan masuk ke riwayat task tersimpan.

### 5. Sinkronisasi dan penghapusan riwayat

Listener Firestore `onSnapshot` memantau koleksi `history` dan `tasks` secara real-time. Grafik Chart.js diperbarui dengan lima data BMI terbaru. Menghapus entri riwayat BMI juga menghapus dokumen task terkait berdasarkan timestamp.

### 6. Logout

Saat pengguna keluar, aplikasi menghentikan listener Firestore (`unsubscribe`), mengosongkan formulir, grafik, dan riwayat, lalu menampilkan kembali halaman login.

## Struktur Repository

```text
.
├── index.html  # Aplikasi utama: antarmuka, styling, dan logika JavaScript
└── README.md   # Dokumentasi proyek
```

## Menjalankan Secara Lokal

Aplikasi berjalan di sisi klien, sehingga tidak memerlukan proses build atau instalasi paket npm. Karena aplikasi menggunakan JavaScript ES Modules, jalankan melalui server lokal dan bukan dengan membuka berkas langsung melalui `file://`.

1. Clone repository dan masuk ke folder proyek:

   ```bash
   git clone https://github.com/BMsktY/BMI_kalkulator.git
   cd BMI_kalkulator
   ```

2. Jalankan server lokal. Contohnya, menggunakan Python:

   ```bash
   python -m http.server 8000
   ```

   Atau gunakan ekstensi **Live Server** di VS Code dengan mengeklik kanan `index.html`, lalu memilih **Open with Live Server**.

3. Buka `http://localhost:8000` atau URL yang diberikan Live Server di browser.

## Konfigurasi Firebase

### 1. Siapkan proyek Firebase

1. Buat proyek di [Firebase Console](https://console.firebase.google.com/).
2. Aktifkan **Authentication** dengan metode **Email/Password**.
3. Aktifkan **Cloud Firestore**. Lokasi yang disarankan adalah `asia-southeast1`.
4. Tambahkan domain aplikasi di **Authentication → Settings → Authorized domains**:
   - `localhost` untuk pengujian lokal.
   - `bmskty.github.io` untuk deployment GitHub Pages.

### 2. Atur Firestore Security Rules

Tambahkan aturan berikut di tab **Firestore → Rules** agar pengguna hanya dapat membaca dan menulis data miliknya sendiri:

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /artifacts/{appId}/users/{userId}/{document=**} {
      allow read, write: if request.auth != null
                         && request.auth.uid == userId;
    }
  }
}
```

### 3. Hubungkan konfigurasi Firebase

Buka `index.html` dan perbarui objek `firebaseConfig` menggunakan konfigurasi dari Firebase Console:

```javascript
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT_ID.firebasestorage.app",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

## Deployment

Aplikasi bodyfit tersedia melalui GitHub Pages:

**URL:** [https://bmskty.github.io/BMI_kalkulator/](https://bmskty.github.io/BMI_kalkulator/)

## Pengembang

Proyek ini dikembangkan untuk kebutuhan pemantauan kesehatan mandiri dan dikelola oleh [BMsktY](https://github.com/BMsktY).
