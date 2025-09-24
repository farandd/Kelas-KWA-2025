# Nested Easter Egg

## Challenge Overview

**Judul:** Nested Easter Egg
**Kategori:** Cryptographic Issues

Tantangan ini meminta kita menemukan pesan tersembunyi (Easter Egg) yang berada di dalam Easter Egg lainnya. Artinya, terdapat **lapisan ganda enkripsi atau penyamaran data** yang perlu dianalisis dan dipecahkan. Tujuan akhirnya adalah mengakses Easter Egg tersembunyi melalui proses dekripsi bertahap.

---

## Tools yang Digunakan

* **Base64 Decoder** → untuk mendekode string yang telah di-encode dengan Base64.
* **ROT13 Cipher Tool** → untuk melakukan dekripsi substitusi huruf menggunakan algoritma ROT13.

---

## Metodologi dan Penyelesaian

### Step 1: Analisis Awal & Decoding Base64

1. Tantangan memberikan sebuah string terenkripsi:

   ```
   L2d1ci9xcmlmL25lci9mYi9zaGFhbC9ndXJsL3V2cS9uYS9ybmZncmUvcnR0L2p2Z3V2YS9ndXIvcm5mZ3JlL3J0dA==
   ```

2. Jalankan string tersebut melalui **Base64 decoder**.
   Hasil decode menghasilkan teks:

   ```
   /gur/qrif/ner/fb/shaal/gurl/uvq/na/rnfgre/rtt/jvguva/gur/rnfgre/rtt
   ```

3. Dari hasil ini terlihat formatnya menyerupai path URL, tetapi isinya masih berupa karakter aneh. Hal ini mengindikasikan ada lapisan enkripsi tambahan.

---

### Step 2: Identifikasi Cipher dan ROT13 Decryption

1. Pola teks terlihat seperti menggunakan substitusi huruf sederhana. Cipher klasik yang sering digunakan dalam Juice-Shop adalah **ROT13**.

2. Setelah menjalankan teks melalui **ROT13 decryption**, didapatkan hasil:

   ```
   /the/devs/are/so/funny/they/hid/an/easter/egg/within/the/easter/egg
   ```

3. Pesan ini jelas menunjukkan path spesifik menuju Easter Egg tersembunyi.

---

### Step 3: Mengakses Nested Easter Egg

1. Gunakan path hasil ROT13 pada URL aplikasi Juice-Shop.
   Buka di browser:

   ```
   http://127.0.0.1:3000/the/devs/are/so/funny/they/hid/an/easter/egg/within/the/easter/egg
   ```
  <img width="1552" height="883" alt="Screenshot 2025-09-24 at 18 33 07" src="https://github.com/user-attachments/assets/2b0a5dab-9ad5-472b-84d2-78fda246b0aa" />

2. Halaman berhasil menampilkan **Nested Easter Egg** yang menjadi tujuan utama tantangan.

---

## Penjelasan Solusi

* **Lapisan Pertama (Base64):** digunakan untuk menyamarkan teks asli agar tidak langsung terbaca manusia.
* **Lapisan Kedua (ROT13):** substitusi sederhana yang tetap efektif untuk menambah kerumitan.

Tantangan ini menekankan bahwa informasi bisa disembunyikan dengan **multi-layer encoding/decoding**, sehingga peserta perlu berpikir kreatif untuk mencoba berbagai teknik dekripsi sebelum menemukan hasil yang benar.

---

## Rekomendasi & Mitigasi

Untuk mencegah teknik penyembunyian data sederhana seperti ini digunakan pada aplikasi nyata:

* **Gunakan enkripsi yang kuat** → bukan sekadar encoding atau cipher klasik seperti ROT13.
* **Multi-layer encryption yang aman** → jika memang perlu, gunakan metode berbeda dengan kunci kuat, bukan hanya substitusi standar.
* **Audit keamanan berkala** → pastikan tidak ada informasi sensitif yang hanya “disamarkan” dengan teknik lemah.
