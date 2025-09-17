# Login Admin

## Deskripsi

Eksploitasi **SQL Injection** pada halaman login untuk dapat masuk menggunakan akun administrator. Teknik ini memanfaatkan query SQL yang tidak diamankan dengan baik, sehingga memungkinkan penyerang untuk memanipulasi parameter input.

## Resource

* [PortSwigger – SQL Injection](https://portswigger.net/web-security/sql-injection)

## Langkah-Langkah

### 1. Masuk ke Halaman Login

Akses aplikasi lalu masuk ke halaman login. Tampilan form login terdiri dari input **Email** dan **Password**.

<img width="1536" height="897" alt="Screenshot 2025-09-10 at 08 10 37" src="https://github.com/user-attachments/assets/34ccce75-53b4-4659-b7c7-05d1eb42cc51" />

---

### 2. Uji SQL Injection pada Field Email

Masukkan payload berikut di kolom **Email**:

```
admin@juice-sh.op' OR 1=1--
```

Untuk kolom **Password**, isi dengan sembarang nilai (contoh: `pass123`).

<img width="1536" height="895" alt="Screenshot 2025-09-10 at 08 12 25" src="https://github.com/user-attachments/assets/bf9a4f30-fc35-43d7-a425-a91f7edc6ca4" />

---

### 3. Hasil Login

Setelah menekan tombol **Log in**, sistem berhasil mengautentikasi tanpa memverifikasi password asli administrator. Kita berhasil login sebagai admin.

<img width="1536" height="895" alt="Screenshot 2025-09-10 at 08 13 28" src="https://github.com/user-attachments/assets/c8a6f7e3-053e-4b08-9b0b-7db0b3121006" />

---

## Analisis Teknis

### Query SQL Rentan

Struktur query rentan biasanya seperti berikut:

```sql
SELECT * FROM users WHERE email = '$email' AND password = '$password';
```

### Query Setelah Injeksi

Jika input email menggunakan payload `admin@juice-sh.op' OR 1=1--`, maka query yang dijalankan menjadi:

```sql
SELECT * FROM users WHERE email = 'admin@juice-sh.op' OR 1=1--' AND password = 'aaaa';
```

Penjelasan:

* `admin@juice-sh.op'` → bagian email asli.
* `OR 1=1` → kondisi selalu benar.
* `--` → menandakan komentar SQL, sehingga sisa query (termasuk validasi password) diabaikan.

Akibatnya, sistem hanya memerlukan kecocokan `email`, sementara password diabaikan. Hal ini memungkinkan login sebagai admin meskipun password salah.

---

## Alternatif dengan Burp Suite

Selain langsung di form, injeksi juga bisa dilakukan dengan **memodifikasi request** menggunakan Burp Suite:

1. **Intercept Request** login. <img width="1919" height="1131" alt="image" src="https://github.com/user-attachments/assets/27f26d09-08e9-4354-8cec-895708ef24d3" />

2. Ubah parameter **email** menjadi:

   ```
   admin@juice-sh.op' OR 1=1--
   ```

   <img width="1919" height="1134" alt="image" src="https://github.com/user-attachments/assets/0cf49c22-c112-4152-8282-90e56e51b639" />

3. Forward request, dan hasilnya akan berhasil login sebagai admin.

   <img width="1536" height="895" alt="Screenshot 2025-09-10 at 08 13 28" src="https://github.com/user-attachments/assets/c8a6f7e3-053e-4b08-9b0b-7db0b3121006" />

---

## Catatan Penting

* Teknik ini hanya untuk pembelajaran dan praktik di **lingkungan yang legal** (misalnya aplikasi Juice Shop atau lab keamanan).
* SQL Injection dapat dicegah dengan:

  * Menggunakan **Prepared Statement / Parameterized Query**.
  * Melakukan **validasi input**.
  * Menerapkan **least privilege** pada database user.
