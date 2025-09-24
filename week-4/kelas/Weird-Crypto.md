# Weird Crypto

## Challenge Overview

**Judul:** Weird Crypto
**Kategori:** Cryptographic Issues

Tantangan **Weird Crypto** menguji kemampuan mengidentifikasi penggunaan fungsi kriptografi yang usang atau tidak aman dalam aplikasi. Fokusnya adalah menemukan bahwa aplikasi memakai metode hashing yang tidak layak (MD5) untuk menyimpan/menangani kata sandi — lalu melaporkan temuan tersebut melalui mekanisme laporan di aplikasi.

---

## Alat yang Digunakan

* **Web Browser** — untuk menavigasi aplikasi, membuka DevTools, dan mengekstrak token autentikasi/cookie.
* **Base64 Decoder** / JWT inspector (mis. jwt.io) — untuk mendekode payload token JWT yang di-encode dalam Base64.
* (Opsional) editor teks / notepad — untuk memeriksa nilai hash dengan lebih mudah.

---

## Metodologi dan Solusi — Langkah demi langkah

### 1. Mengambil Token Autentikasi

* Buka aplikasi Juice Shop dan autentikasi sebagai pengguna (atau gunakan sesi yang sudah ada).
* Buka *Developer Tools* → *Application* → *Cookies* (atau *Storage*), lalu temukan cookie yang menyimpan JWT / token autentikasi. Salin nilai token tersebut.

### 2. Men-decode Payload JWT

* JWT terdiri dari tiga bagian (header.payload.signature), dipisahkan oleh titik (`.`). Bagian tengah (payload) biasanya Base64URL-encoded.
* Gunakan Base64 decoder atau langsung paste token ke situs seperti **jwt.io** untuk melihat isi payload dalam format JSON.
* Dari payload yang ter-decode, periksa atribut-atribut yang ditampilkan; cari entri terkait kredensial, mis. `password` atau `hashedPassword`.
  
<img width="1552" height="883" alt="Screenshot 2025-09-24 at 18 45 20" src="https://github.com/user-attachments/assets/9c22433a-f28b-43ee-a378-979ba1088f73" />

*(Catatan: alternatif cepat adalah copy-paste ke jwt.io untuk melihat payload dan klaim secara langsung.)*

### 3. Mengidentifikasi Format Hash

* Dalam payload ditemukan entri password ter-hash:

```json
"password":"0192023a7bbd73250516f069df18b500"
```

* Perhatikan panjang dan pola (32 karakter hex). Pola ini khas **MD5** (128-bit, direpresentasikan sebagai 32 hex digits).
* MD5 sudah lama dianggap tidak aman untuk hashing password karena:

  * Sangat cepat sehingga memudahkan brute-force.
  * Rentan terhadap collision (walau collision lebih relevan untuk integritas, bukan hanya password).
  * Tersedia tabel precomputed (rainbow tables) dan GPU-accelerated cracking tools.

### 4. Konfirmasi dan Analisis Singkat

* Verifikasi: bentuk dan panjang hash cocok dengan MD5; tidak terlihat salt di sekitar nilai hash tersebut pada payload.
* Tanpa salt, hash MD5 untuk kata sandi populer sangat mudah dipetakan kembali menggunakan rainbow tables atau komputasi cepat.
* Oleh karena itu, ini merupakan kerentanan kriptografi yang nyata pada pengelolaan kata sandi di aplikasi.

---

## Pelaporan (melengkapi challenge)

* Gunakan fitur **Contact** atau formulir pelaporan dalam Juice Shop untuk mengirimkan laporan kerentanan.
* Dalam laporan, sertakan deskripsi singkat: lokasi (JWT payload), bukti (cuplikan hash), analisis (kenapa MD5 tidak aman), dan rekomendasi perbaikan.
* Mengirim laporan melalui UI aplikasi menyelesaikan objective challenge.

---

## Penjelasan teknis singkat mengapa ini masalah serius

* **MD5 bukanlah password hashing function** — fungsi yang tepat harus memperlambat hashing (cost factor) dan menggunakan salt unik per password.
* Password hashing modern menggunakan algoritma seperti **bcrypt**, **scrypt**, atau **Argon2** yang dirancang tahan terhadap brute-force dan memiliki parameter cost yang dapat diatur.
* Menyimpan hash MD5 tanpa salt membuat akun rentan terhadap pembobolan massal jika database/secret terekspos.

---

## Rekomendasi Perbaikan (Remediation)

1. **Ganti mekanisme hashing ke algoritma yang aman**

   * Implementasikan *bcrypt*, *scrypt*, atau *Argon2* untuk hashing password. Pilih cost parameter yang sesuai sehingga hashing cukup lambat untuk mempersulit brute-force tapi masih dapat diterima performanya.

2. **Gunakan salt unik per password**

   * Pastikan setiap password di-hash dengan salt yang berbeda (random), sehingga dua pengguna dengan password sama menghasilkan hash berbeda.

3. **Jangan masukkan informasi sensitif ke dalam token yang mudah diinspeksi**

   * Hindari menyimpan hash password (atau informasi sensitif lainnya) langsung di payload JWT yang dipaparkan ke klien. Token hanya perlu memuat klaim minimal (user id, role, expiry), bukan kredensial.

4. **Amankan token dan enkripsi bila perlu**

   * Jika payload token harus berisi informasi sensitif (sebaiknya tidak), gunakan token terenkripsi (JWE) atau jangan simpan di cookie yang dapat dibaca klien.

5. **Audit & rotasi kredensial**

   * Lakukan audit kode untuk menemukan penggunaan hashing usang lainnya; apabila ditemukan hash MD5 pada database, lakukan migrasi terkontrol: minta pengguna mereset password atau lakukan re-hash saat pengguna login berikutnya.

6. **Edukasi dan kebijakan dependency**

   * Pastikan tim dev memahami best-practice penyimpanan kata sandi dan lakukan code review/security scan untuk menghindari pola tidak aman.

---

## Kesimpulan

Tantangan **Weird Crypto** diselesaikan dengan menginspeksi JWT yang tersimpan di browser, mendekode payload, mengidentifikasi hash MD5 pada atribut password, lalu melaporkan temuan melalui mekanisme yang disediakan aplikasi. Kasus ini menonjolkan betapa pentingnya memilih fungsi hashing yang tepat dan tidak mengekspos kredensial atau hash ke area yang dapat diakses/diinspeksi oleh klien.
