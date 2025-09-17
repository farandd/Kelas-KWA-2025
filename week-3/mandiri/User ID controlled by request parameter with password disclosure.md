# User ID dikontrol oleh parameter request — pengungkapan password pada halaman akun

Lab ini memiliki **kebocoran data sensitif** pada halaman akun pengguna: kolom password pengguna di-prefill (diisi otomatis) dengan password saat ini tetapi dalam elemen `<input type="password">`. Dengan memanipulasi parameter `id` pada request yang menampilkan halaman akun, kita dapat memuat halaman akun pengguna lain (mis. `administrator`), lalu melihat password yang tersembunyi dengan mengubah `type` input menjadi `text`. Setelah mendapat password administrator, gunakan kredensial admin tersebut untuk menghapus user `carlos`.

**Tujuan lab:** ambil password administrator → gunakan untuk menghapus user `carlos`.

Kredensial untuk percobaan awal:

```
username: wiener
password: peter
```

Referensi:

* [https://portswigger.net/web-security/access-control](https://portswigger.net/web-security/access-control)

---

## 1. Login dan klik “Home” untuk melihat request GET

Login ke aplikasi dengan akun `wiener:peter`. Setelah login, klik tombol **“Home”** — ini memicu request GET yang memuat halaman akun pengguna berdasarkan parameter `id`. Contoh request yang muncul saat klik Home:

![img](images/User%20ID%20controlled%20by%20request%20parameter%20with%20password%20disclosure/2.png)

Perhatikan bahwa URL/request memakai parameter `id` untuk memilih akun mana yang dimuat.

---

## 2. Ubah parameter `id` menjadi `administrator` untuk memuat halaman admin

Ganti nilai parameter `id` di URL/request menjadi `administrator` (atau identifier akun admin yang sesuai). Dengan begitu server akan mengembalikan halaman akun milik administrator — karena aplikasi tidak melakukan pengecekan authorization yang memadai. Contoh eksploitasi:

![img](images/User%20ID%20controlled%20by%20request%20parameter%20with%20password%20disclosure/3.png)

---

## 3. Ungkap password yang tersembunyi dengan mengubah `type` input

Pada halaman akun administrator, field password akan berisi password admin tetapi terlihat sebagai titik/asterisk karena `type="password"`. Untuk membacanya, buka Developer Tools (Inspect Element) di browser, temukan elemen `<input>` untuk password, lalu ubah atribut `type="password"` menjadi `type="text"`. Setelah diubah, password asli akan terlihat jelas di halaman:

![img](images/User%20ID%20controlled%20by%20request%20parameter%20with%20password%20disclosure/4.png)

Langkah praktis (browser biasa):

1. Klik kanan pada field password → Inspect / Periksa.
2. Di panel Elements, cari `<input type="password" ...>` untuk field tersebut.
3. Ganti `type="password"` → `type="text"`.
4. Password plaintext akan muncul di value atribut / di field.

Catatan: beberapa aplikasi menambahkan proteksi client-side, tapi dalam banyak kasus pengubahan DOM ini sudah cukup karena nilai sudah ada di HTML.

---

## 4. Gunakan password administrator untuk melakukan aksi sensitif (hapus `carlos`)

Setelah mendapatkan password administrator, masuk (login) atau gunakan mekanisme otentikasi yang diperlukan dengan credential `administrator:<password_yang_didapat>`. Akses halaman admin atau endpoint penghapusan user dan hapus user `carlos`. Contoh alur yang digambarkan di lab:

![img](images/User%20ID%20controlled%20by%20request%20parameter%20with%20password%20disclosure/5.png)

(Jika aplikasi meminta konfirmasi password atau autentikasi ulang untuk operasi sensitif, masukkan password admin yang telah diperoleh.)

---

## 5. Mengapa ini merupakan kelemahan

* **Data sensitif (password)** ditempatkan di HTML halaman (prefilled) sehingga dapat diakses oleh klien — idealnya password tidak pernah dikembalikan/ditampilkan kembali setelah set.
* **Aplikasi mengizinkan pemuatan halaman akun lain** berdasarkan parameter `id` tanpa pemeriksaan authorization server-side → **IDOR / horizontal access control issue**.
* Kombinasi: server mengembalikan password admin di halaman yang bisa diakses hanya lewat parameter yang dikontrol klien → memungkinkan pengungkapan password admin.

---

## 6. Rekomendasi mitigasi singkat

* **Jangan me-prefill atau mengirimkan password asli** kembali ke klien. Simpan hanya hash password di server; password plaintext tidak boleh dikirim ke browser setelah pembuatan/registrasi.
* **Server-side authorization**: selalu verifikasi ownership/permission sebelum mengembalikan data akun. Jangan menentukan resource hanya berdasarkan input pengguna tanpa cek hak akses.
* Untuk operasi sensitif (mis. hapus user), minta re-authenticaton atau verifikasi tambahan.
* Minimalisasi data sensitif di halaman dan lakukan validasi / filtering terhadap apa yang boleh ditampilkan di klien.
