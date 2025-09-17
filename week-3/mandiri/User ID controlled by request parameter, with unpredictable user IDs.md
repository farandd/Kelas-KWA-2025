# User ID dikontrol oleh parameter request, dengan user-ID yang tidak tertebak (GUID)

Lab ini memiliki **keterbatasan kontrol akses horizontal** pada halaman akun pengguna, tetapi alih-alih memakai username yang mudah ditebak, aplikasi mengidentifikasi pengguna menggunakan **GUID** (Globally Unique Identifier) — membuat penebakan langsung lebih sulit.

**Tujuan lab:** temukan **GUID** untuk user `carlos`, lalu gunakan GUID tersebut untuk mengambil **API key** milik `carlos` dan submit API key itu sebagai solusi.

Kredensial untuk login:

```
username: wiener
password: peter
```

Referensi:

* [https://portswigger.net/web-security/access-control](https://portswigger.net/web-security/access-control)

---

### 1. Login dan observasi request “Home”

Login ke aplikasi memakai akun `wiener:peter`. Setelah login, klik tombol **“Home”** — aksi ini memicu sebuah request GET yang menampilkan halaman akun berdasarkan parameter `id` yang berisi GUID pengguna yang sedang diload. Contoh request yang dihasilkan:

![img](images/User%20ID%20controlled%20by%20request%20parameter,%20with%20unpredictable%20user%20IDs/2.png)

Perhatikan URL/parameter `id` — sekarang berisi GUID, bukan username biasa. Ini berarti untuk melihat data pengguna lain kita perlu mengetahui GUID mereka.

---

### 2. Cari GUID target (`carlos`) dari sumber lain (mis. posting blog)

Agar dapat mengakses data `carlos`, kita perlu mengetahui GUID miliknya. Di lab ini, GUID pengguna lain (termasuk admin atau user) dapat ditemukan dari sumber aplikasi lain — misalnya **daftar posting blog** yang memuat metadata penulis atau link yang menyertakan GUID. Periksa halaman publik (blog, posting, profil) untuk menemukan GUID yang terkait dengan username target.

Contoh: kita menemukan GUID untuk administrator atau pengguna lain pada posting blog:

![img](images/User%20ID%20controlled%20by%20request%20parameter,%20with%20unpredictable%20user%20IDs/3.png)

Dengan teknik yang sama, cari GUID untuk `carlos` (biasanya ada di posting, URL author, atau resource publik lain).

---

### 3. Buat request baru menggunakan GUID `carlos`

Setelah mendapatkan GUID `carlos`, susun ulang request GET yang sebelumnya digunakan untuk menampilkan halaman akun — namun kali ini ganti parameter `id` menjadi GUID milik `carlos`. Contoh perubahan request:

* Sebelumnya (contoh): `GET /my-account?id=GUID-milik-aku`
* Setelah diganti: `GET /my-account?id=GUID-carlos`

Kirim request tersebut — server akan merespon dengan halaman akun user yang `id`-nya cocok, dan karena server tidak melakukan pengecekan authorization yang memadai (hanya mengandalkan parameter `id`), respons akan menampilkan data `carlos`, termasuk **API key** miliknya.

Ilustrasi membuat request baru ini:

![img](images/User%20ID%20controlled%20by%20request%20parameter,%20with%20unpredictable%20user%20IDs/4.png)

---

### 4. Ambil API key dan submit sebagai jawaban

Di respon halaman akun yang dimuat dengan GUID `carlos`, salin nilai **API key** yang tampil. Masukkan nilai API key tersebut ke form penyelesaian lab untuk menyelesaikan tantangan.

---

### 5. Kenapa ini berhasil (penyebab kelemahan)

* Aplikasi menentukan resource yang ditampilkan hanya berdasarkan parameter yang dikirim klien (`id` = GUID).
* **Tidak ada pemeriksaan authorization** di server untuk memastikan bahwa pengguna yang meminta memang berhak mengakses resource tersebut.
* Ini termasuk varian **Insecure Direct Object Reference (IDOR)** / **horizontal privilege escalation**: pengguna biasa bisa mengakses data pengguna lain jika mengetahui identifier mereka (walau berupa GUID).

---

### 6. Rekomendasi mitigasi singkat

* Terapkan **server-side authorization**: setelah menentukan resource dari parameter request, verifikasi bahwa user authenticated memang berhak mengakses resource tersebut.
* Jangan andalkan identifier yang dikirim klien sebagai satu-satunya mekanisme kontrol akses — selalu cek ownership/permission di server.
* Kurangi eksposur GUID di halaman publik bila tidak perlu; gunakan mapping internal atau batasi informasi publik yang memuat identifier sensitif.
