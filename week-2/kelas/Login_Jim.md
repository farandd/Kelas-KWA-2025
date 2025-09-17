# Login Admin (User Jim)

## Deskripsi

Eksploitasi **SQL Injection** pada halaman login untuk bisa masuk sebagai user **Jim** tanpa mengetahui password aslinya. Teknik ini dilakukan dengan memanfaatkan query SQL yang tidak diamankan, sehingga input dari pengguna dapat dimanipulasi untuk melewati proses autentikasi.

## Referensi

* [PortSwigger – SQL Injection](https://portswigger.net/web-security/sql-injection)

---

## Langkah-Langkah

### 1. Temukan Email Milik Jim

Pertama, cari user bernama **Jim** pada bagian review produk. Dari salah satu review terlihat bahwa email Jim adalah:

```
jim@juice-sh.op
```

<img width="1536" height="895" alt="Screenshot 2025-09-10 at 08 28 34" src="https://github.com/user-attachments/assets/994c4a66-1e12-44fc-b0de-957a459d4302" />

---

### 2. Buka Halaman Login

Masuk ke halaman login aplikasi yang menyediakan input **Email** dan **Password**.

<img width="1536" height="895" alt="Screenshot 2025-09-10 at 08 29 23" src="https://github.com/user-attachments/assets/531dc262-60c1-4561-b24d-7abe94e675f7" />

---

### 3. Lakukan SQL Injection

Isi kolom **Email** dengan payload berikut:

```
jim@juice-sh.op' OR 1=1--
```

Untuk kolom **Password**, isi sembarang (misalnya: `pass123`).

<img width="1536" height="895" alt="Screenshot 2025-09-10 at 08 32 09" src="https://github.com/user-attachments/assets/c0b61676-346b-4ff0-a066-62099b929d86" />

---

### 4. Login Berhasil

Setelah klik **Log in**, sistem langsung menerima login sebagai Jim tanpa melakukan validasi password.

<img width="1536" height="895" alt="Screenshot 2025-09-10 at 08 32 47" src="https://github.com/user-attachments/assets/0d8c2d05-e71f-4c40-a5e6-1b379e1765a5" />

---

## Analisis Teknis

### Query SQL Asli

Query login yang rawan biasanya terlihat seperti ini:

```sql
SELECT * FROM users WHERE email = '$email' AND password = '$password';
```

### Query Setelah Injeksi

Dengan payload `jim@juice-sh.op' OR 1=1--`, query menjadi:

```sql
SELECT * FROM users WHERE email = 'jim@juice-sh.op' OR 1=1--' AND password = 'aaaa';
```

**Penjelasan:**

* `jim@juice-sh.op'` → email milik Jim.
* `OR 1=1` → kondisi selalu benar.
* `--` → komentar SQL, membuat sisa query (termasuk pengecekan password) diabaikan.

Akibatnya, sistem hanya butuh kecocokan pada bagian email, sedangkan password tidak diperiksa.

---

## Alternatif dengan Burp Suite

Langkah ini juga bisa dikerjakan menggunakan **Burp Suite** dengan memodifikasi request login:

1. **Intercept request** login menggunakan Burp Suite.

   <img width="1919" height="1142" alt="image" src="https://github.com/user-attachments/assets/e69119c3-26bb-43dd-9442-b9b8add40a5a" />

4. Ubah parameter **email** menjadi:

   ```
   jim@juice-sh.op'--
   ```

   <img width="1919" height="1134" alt="image" src="https://github.com/user-attachments/assets/aabcbf4f-7465-4953-84c1-fd8b97db0aec" />

5. Forward request. Hasilnya, login sebagai Jim berhasil tanpa memasukkan password asli.

<img width="1536" height="895" alt="Screenshot 2025-09-10 at 08 32 47" src="https://github.com/user-attachments/assets/0d8c2d05-e71f-4c40-a5e6-1b379e1765a5" />

---

## Catatan Penting

* Eksploitasi ini **hanya boleh dilakukan di lingkungan yang legal**, seperti aplikasi **OWASP Juice Shop** atau lab keamanan.
* Untuk mencegah SQL Injection, beberapa langkah yang dapat dilakukan:

  * Gunakan **Prepared Statements / Parameterized Queries**.
  * Lakukan **validasi input** sebelum diproses.
  * Terapkan **least privilege** pada user database agar dampak lebih terbatas.
