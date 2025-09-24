# Imaginary Challenge

## Challenge Overview

**Title:** Imaginary Challenge
**Category:** Cryptographic Issues

---

## Tools Used

* **Web Browser** → digunakan untuk berinteraksi langsung dengan aplikasi Juice Shop.
* **Source Code Review** → khususnya pada file `package.json` dan modul terkait untuk mencari dependensi yang berhubungan dengan challenge.
* **Community References** → catatan dari write-up lama dan dokumentasi pihak ketiga yang pernah membahas penggunaan `hashid`.

---

## Methodology and Solution

### Step 1: Identifikasi Permasalahan

Tidak ada petunjuk eksplisit yang muncul di antarmuka aplikasi mengenai challenge ini. Karena disebut “imaginary”, langkah pertama adalah melakukan audit kode sumber, dengan asumsi bahwa mekanisme challenge disimpan di dalam modul back-end.

---

### Step 2: Analisis `package.json`

Saat menelusuri `package.json`, ditemukan adanya dependensi bernama **`hashid`**. Modul ini digunakan untuk encoding dan decoding numeric identifiers menjadi format hash-like string. Dugaan awal: challenge ini pernah memanfaatkan `hashid` untuk menghasilkan **ContinueCode** khusus.

---

### Step 3: Melacak Rujukan Eksternal

Dari penelusuran komunitas dan write-up lama, diketahui bahwa:

* Beberapa tahun lalu, situs resmi `hashid` menyediakan demo berbasis **CodePen**.
* Demo tersebut memungkinkan pengguna untuk menguji encoding angka tertentu dengan salt default.
* Sayangnya, situs tersebut kini sudah diakuisisi oleh perusahaan lain, halaman demo dihapus, sehingga langkah asli tidak dapat lagi dijalankan.

---

### Step 4: Rekonstruksi Solusi Teoretis

Berdasarkan dokumentasi lama, skenario solusi kira-kira sebagai berikut:

1. Menggunakan demo `hashid` untuk **encode angka 999** dengan salt bawaan (default salt).
2. Dari proses tersebut akan dihasilkan sebuah string unik yang disebut sebagai **ContinueCode**.
3. ContinueCode ini kemudian digunakan dalam sebuah request:

   ```bash
   PUT http://localhost:3000/rest/continue-code/apply/{ContinueCode}
   ```

   dengan `{ContinueCode}` diganti hasil dari hashid.
4. Jika berhasil, challenge ini akan ditandai sebagai “solved”.

---

## Solution Explanation

Tantangan ini bukan semata soal teknik, melainkan **pelajaran mengenai kelemahan default values**. Jika sebuah sistem keamanan bergantung pada nilai default bawaan library (seperti salt statis dari hashid), maka penyerang bisa memanfaatkannya untuk mem-bypass mekanisme otentikasi atau validasi.

Dengan kata lain, **security by default tanpa konfigurasi tambahan sangat berisiko**.

---

## Remediation

* **Hindari Default Values**: Selalu lakukan konfigurasi salt/secret secara manual dan unik untuk tiap environment.
* **Dokumentasi Internal**: Fitur eksperimental atau tantangan placeholder sebaiknya diberi catatan resmi agar tidak membingungkan pengguna di masa depan.
* **Dependency Hygiene**: Pastikan dependensi eksternal yang tidak lagi dipelihara tidak dipakai dalam aplikasi, karena bisa menimbulkan celah keamanan.

---
