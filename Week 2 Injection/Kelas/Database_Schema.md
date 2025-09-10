# Database Schema

## Deskripsi

Tugas ini adalah **mengekstrak seluruh definisi skema database** (database schema definition) melalui celah **SQL Injection**.

## Resource

* [Examining the database - PortSwigger](https://portswigger.net/web-security/sql-injection/examining-the-database)
* [Union attacks - PortSwigger](https://portswigger.net/web-security/sql-injection/union-attacks)

---

## Langkah - Langkah

### 1. Identifikasi endpoint yang rentan

Selain halaman login, terdapat endpoint pencarian produk yang menggunakan parameter `q`.
Parameter ini berpotensi rentan karena digunakan dalam query SQL.

<img width="1919" height="1136" alt="image" src="https://github.com/user-attachments/assets/221971e4-9967-4d93-947b-5cc6b9879201" />

---

### 2. Kirim request ke **Repeater**

Agar payload bisa dimodifikasi dan diuji berulang kali, kirim request tersebut ke tab **Repeater** di Burp Suite.

<img width="1919" height="1141" alt="image" src="https://github.com/user-attachments/assets/9a40d574-3c96-4b5c-ae66-0fd5a854c1bd" />

---

### 3. Uji modifikasi query

Lakukan manipulasi pada parameter `q` untuk memastikan query benar-benar diproses oleh database.

<img width="1919" height="1136" alt="image" src="https://github.com/user-attachments/assets/a8c0a61b-af78-4417-b3bd-cac1e324b353" />

---

### 4. Cari jumlah kolom untuk UNION

Karena akan menggunakan **UNION injection**, jumlah kolom di payload harus sama dengan jumlah kolom pada query asli.

* Payload awal:

  ```
  orange'))+UNION+SELECT+'a'--
  ```

  Masih error → artinya jumlah kolom belum cocok.

* Tambahkan `'a'` hingga tidak ada error.

  ```
  orange'))+UNION+SELECT+'a','a','a','a','a','a','a','a','a'--
  ```

  <img width="1919" height="1134" alt="image" src="https://github.com/user-attachments/assets/ee6dd2dd-f909-44d1-840e-411f24082cda" />  

  **Hasil:** Tidak ada error → jumlah kolom = **9**.

---

### 5. Ekstrak schema database

Gunakan `sqlite_schema` untuk mendapatkan definisi seluruh tabel dan struktur database.

Payload:

```
orange'))+UNION+SELECT+'a','a','a','a','a','a','a','a',sql+FROM+sqlite_schema--
```

<img width="1919" height="1140" alt="image" src="https://github.com/user-attachments/assets/59ac8d25-3050-4922-a6a8-ff5aee2ba325" />

---

## Catatan

* **Langkah penting:** menentukan jumlah kolom dengan benar sebelum menggunakan `UNION`.
* Dari hasil query terhadap `sqlite_schema`, terlihat bahwa database yang digunakan adalah **SQLite**.
* Dengan teknik ini, seluruh struktur tabel, kolom, dan relasi dalam database dapat dieksfiltrasi.

