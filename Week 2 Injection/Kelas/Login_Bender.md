# Login Bender

## Deskripsi

Eksploitasi **SQL Injection** di halaman login untuk masuk menggunakan akun **Bender** tanpa mengetahui password aslinya. Teknik ini bekerja karena query SQL tidak diamankan dengan baik, sehingga input dari pengguna bisa dimanipulasi untuk melewati autentikasi.

## Referensi

* [PortSwigger – SQL Injection](https://portswigger.net/web-security/sql-injection)

---

## Langkah-Langkah

### 1. Temukan Email Bender

Cari user bernama **Bender** di halaman review produk. Dari salah satu review terlihat bahwa email Bender adalah:

```
bender@juice-sh.op
```

<img width="1536" height="895" alt="Screenshot 2025-09-10 at 09 57 08" src="https://github.com/user-attachments/assets/28e7b7fb-b453-4af6-8cb6-33af27eef2cc" />

---

### 2. Buka Halaman Login

Akses halaman login aplikasi. Form login berisi dua field: **Email** dan **Password**.

<img width="1536" height="895" alt="Screenshot 2025-09-10 at 10 09 11" src="https://github.com/user-attachments/assets/9798c3ef-3808-4558-8f27-01aa5ad58c0b" />

---

### 3. Lakukan SQL Injection

Isi kolom **Email** dengan payload berikut:

```
bender@juice-sh.op'--
```

Untuk **Password**, isi sembarang (contoh: `pass123`).

<img width="1536" height="895" alt="Screenshot 2025-09-10 at 10 10 12" src="https://github.com/user-attachments/assets/cd7b5604-416a-4234-9dc8-c1ae375275c6" />

---

### 4. Login Berhasil

Setelah menekan tombol **Log in**, sistem mengabaikan pengecekan password dan langsung menerima login sebagai Bender.

<img width="1536" height="895" alt="Screenshot 2025-09-10 at 10 10 54" src="https://github.com/user-attachments/assets/b9b611c9-bec5-47d9-add3-2c7a7c0068da" />

---

## Analisis Teknis

### Query SQL Asli

Query rawan biasanya ditulis seperti ini:

```sql
SELECT * FROM users WHERE email = '$email' AND password = '$password';
```

### Query Setelah Injeksi

Dengan payload `bender@juice-sh.op'--`, query berubah menjadi:

```sql
SELECT * FROM users WHERE email = 'bender@juice-sh.op'--' AND password = 'aaaa';
```

**Penjelasan:**

* `bender@juice-sh.op'` → email milik Bender.
* `--` → menandakan komentar dalam SQL, sehingga bagian setelahnya (termasuk pengecekan password) diabaikan.

Akibatnya, sistem hanya membaca bagian email dan langsung menganggap login valid meskipun password salah.

---

## Alternatif dengan Burp Suite

Cara lain adalah menggunakan **Burp Suite** untuk memodifikasi request login:

1. **Intercept request** login. <img width="1919" height="1138" alt="image" src="https://github.com/user-attachments/assets/09e74ffc-c771-4634-8d41-b8addcb225b7" />

2. Ubah parameter **email** menjadi:

   ```
   bender@juice-sh.op'--
   ```

   <img width="1919" height="1140" alt="image" src="https://github.com/user-attachments/assets/cc8de883-c9e6-4511-ba77-6b18375e879b" />  

3. Forward request, maka login berhasil sebagai Bender.

<img width="1536" height="895" alt="Screenshot 2025-09-10 at 10 10 54" src="https://github.com/user-attachments/assets/b9b611c9-bec5-47d9-add3-2c7a7c0068da" />

---

## Catatan Penting

* Eksploitasi ini **hanya boleh dilakukan di aplikasi latihan seperti OWASP Juice Shop atau lab keamanan**.
* Untuk mencegah SQL Injection, beberapa langkah yang bisa diterapkan:

  * Gunakan **Prepared Statements / Parameterized Queries**.
  * Terapkan **validasi input**.
  * Batasi hak akses database dengan prinsip **least privilege**.
