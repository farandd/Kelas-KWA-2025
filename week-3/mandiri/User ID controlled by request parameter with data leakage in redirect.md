# User ID dikontrol oleh parameter request — kebocoran data sensitif di body redirect

Lab ini berisi **celah kontrol akses** di mana data sensitif (API key) bocor **di dalam body response pada saat redirect**. Biasanya redirect hanya mengandung header `Location` dan bukan payload sensitif — di sini server menaruh data sensitif di body response redirect sehingga klien/interceptor bisa melihatnya.

**Tujuan lab:** dapatkan **API key** milik user `carlos` dan submit sebagai solusi.

Kredensial untuk login:

```
username: wiener
password: peter
```

Referensi:

* [https://portswigger.net/web-security/access-control](https://portswigger.net/web-security/access-control)

---

## 1. Login dan observasi request “Home”

Login ke aplikasi menggunakan akun `wiener:peter`. Setelah login, klik tombol **“Home”** — aksi ini memicu sebuah request GET yang memuat halaman akun berdasarkan parameter `id`. Contoh request yang dihasilkan saat klik Home:

![img](images/User%20ID%20controlled%20by%20request%20parameter%20with%20data%20leakage%20in%20redirect/2.png)

Perhatikan parameter `id` di URL — nilainya menentukan akun mana yang diminta oleh server.

---

## 2. Eksperimen: ubah parameter `id` menjadi `carlos`

Ganti parameter `id` di URL/request menjadi `carlos` (atau nilai yang sesuai untuk pengguna target). Saat kamu lakukan itu, server merespons dengan **redirect** (biasanya status 3xx). Contoh saat `id` diubah ke `carlos`:

![img](images/User%20ID%20controlled%20by%20request%20parameter%20with%20data%20leakage%20in%20redirect/3.png)

Perlu dicatat: redirect itu sendiri terjadi — tetapi yang penting adalah **isi body response redirect**.

---

## 3. Temukan API key di *body* response redirect

Pada kasus ini server menyertakan informasi sensitif (API key `carlos`) di **body** response redirect. Biasanya browser tidak menampilkan body redirect, tapi interceptor (Burp, DevTools Network, curl dengan `-v`, dsb.) bisa membaca body tersebut. Contoh bagian body yang mengandung API key:

![img](images/User%20ID%20controlled%20by%20request%20parameter%20with%20data%20leakage%20in%20redirect/4.png)

Salin API key yang muncul—itulah yang diminta sebagai solusi lab.

---

## 4. Kenapa ini terjadi (penyebab kelemahan)

* Server menerima parameter `id` dari klien untuk memilih resource (akun) yang akan ditampilkan, **tanpa** memverifikasi bahwa peminta berwenang untuk mengakses data tersebut (kekurangan pemeriksaan authorization).
* Selain itu, server menaruh **data sensitif di body response redirect** — praktik yang salah karena redirect seharusnya hanya mengarahkan dan tidak mengekspos payload. Body redirect bisa terekspos ke log/interceptor atau ditangkap oleh tooling, sehingga menyebabkan kebocoran data.
* Kombinasi kedua hal ini (ID yang dikontrol klien + data sensitif di body redirect) menghasilkan kebocoran API key pengguna lain (horizontal access control issue / IDOR + data leakage).

---

## 5. Cara praktis mendapatkan API key (ringkasan langkah)

1. Login sebagai `wiener:peter`.
2. Di browser/proxy, klik **Home** untuk melihat request GET yang memakai `id`.
3. Ubah parameter `id` menjadi `carlos` pada URL/request.
4. Kirim request dan periksa **response redirect** di proxy / DevTools — baca **body** response; salin **API key** yang muncul.
5. Submit API key tersebut sebagai jawaban lab.

> Catatan: Browser biasa sering mengabaikan body redirect; gunakan alat yang bisa melihat isi response (mis. Burp, mitmproxy, atau DevTools → Network → pilih request → Response).

---

## 6. Mitigasi singkat (rekomendasi)

* **Server-side authorization**: setelah menentukan resource dari parameter request, selalu verifikasi ownership/permission (apakah user yang sedang login berhak mengakses resource itu).
* **Hindari meletakkan data sensitif di body redirect**. Jika perlu mengirim informasi, gunakan mekanisme yang aman (mis. halaman yang hanya ditampilkan setelah autentikasi dan authorization yang benar).
* **Minimalisasi eksposur identifier**: jangan publikasikan identifier sensitif di resource publik jika tidak perlu.
* **Audit logging & monitoring** untuk mendeteksi akses tidak sah.