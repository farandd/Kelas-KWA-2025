# User ID dikontrol oleh parameter request

Lab ini memiliki **keterbatasan kontrol akses horizontal** pada halaman akun pengguna. Artinya satu pengguna biasa dapat melihat/akses data pengguna lain hanya dengan memodifikasi parameter pada request (tanpa harus eskalasi hak istimewa).

**Tujuan lab:** dapatkan **API key** milik user `carlos` dan masukkan API key tersebut sebagai solusi.

Kredensial yang bisa digunakan untuk login:

```
username: wiener
password: peter
```

Referensi:

* [https://portswigger.net/web-security/access-control](https://portswigger.net/web-security/access-control)

---

### 1. Masuk (login)

Login ke web aplikasi menggunakan kredensial `wiener:peter`. Setelah login, navigasikan ke halaman utama atau halaman akun seperti biasa.

---

### 2. Amati request saat klik “Home”

Saat tombol **“Home”** diklik, browser mengirim sebuah request GET yang berisi parameter `id` untuk memuat data pengguna. Contoh request yang dihasilkan saat kamu klik Home:

![img](images/User%20ID%20controlled%20by%20request%20parameter/2.png)

Perhatikan bagian query string / parameter di URL — biasanya ada sesuatu seperti `?id=<username atau user-id>` atau path yang memakai `id`.

---

### 3. Eksploitasi: ubah parameter `id` menjadi `carlos`

Karena server hanya bergantung pada parameter `id` untuk menampilkan data pengguna, kita dapat **mengganti nilai `id`** di URL/request menjadi `carlos`. Dengan mengganti parameter ini, server akan mengembalikan data milik user `carlos` — termasuk API key-nya.

Contoh: ubah `id=wiener` menjadi `id=carlos` (atau langsung ke `/my-account?user=carlos` tergantung struktur). Setelah diubah, load ulang request → akan terlihat respon yang mengandung API key user `carlos`:

![img](images/User%20ID%20controlled%20by%20request%20parameter/3.png)

Itu menunjukkan adanya kontrol akses yang lemah: aplikasi **mengandalkan input klien** untuk menentukan data mana yang boleh ditampilkan, tanpa memverifikasi bahwa peminta memang berhak melihat data tersebut.

---

### 4. Ambil API key dan submit sebagai jawaban

Salin API key yang muncul di respon setelah mengganti `id` menjadi `carlos`. Masukkan nilai API key tersebut ke form/field solusi lab untuk menyelesaikan tantangan.

---

### 5. Penjelasan singkat kenapa ini terjadi

* Aplikasi menggunakan **parameter yang dapat dikontrol pengguna** (user-supplied parameter) untuk menentukan resource yang dikembalikan.
* Tidak ada pemeriksaan authorization pada server untuk memastikan pemanggil hanya mengakses resource miliknya sendiri.
* Ini adalah contoh **horizontal privilege escalation** / **insecure direct object reference (IDOR)**: pengguna biasa bisa mengakses data sesama pengguna.

---

### 6. Rekomendasi mitigasi singkat (untuk catatan)

* Server wajib melakukan **authorization check**: setelah menentukan resource dari parameter, periksa apakah user yang sedang authenticated memang punya hak akses ke resource tersebut.
* Jangan mengandalkan hanya identifier yang diberikan klien untuk akses data sensitif.
* Gunakan identifier internal acak yang tidak mudah ditebak **ditambah** validasi di server.