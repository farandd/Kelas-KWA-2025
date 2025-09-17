# NoSQL Manipulation

## Ringkasan Challenge

Challenge ini mengeksploitasi kerentanan **NoSQL Injection** untuk memanipulasi query sehingga dapat mengubah banyak ulasan produk sekaligus pada database NoSQL (MongoDB).

---

## Tools yang Digunakan

* **Interception Proxy (contoh: Burp Suite):** Untuk menangkap dan memodifikasi HTTP request.
* **Pengetahuan Teknik NoSQL Injection:** Terutama terkait penggunaan operator query di MongoDB.

---

## Metodologi dan Penyelesaian

### 1. Analisis Request Normal

Pertama, saya menangkap request normal untuk mengubah satu ulasan produk. Request tersebut menggunakan metode `PATCH` dengan payload JSON seperti berikut:

```http
PATCH /rest/products/reviews HTTP/1.1
Host: 127.0.0.1:3000
Content-Type: application/json

{
  "id": "specific_review_id",
  "message": "updated_review_text"
}
```

Visualisasi di Burp Suite:

<img width="501" height="199" alt="Screenshot 2025-09-10 at 12 15 19" src="https://github.com/user-attachments/assets/40b86455-6073-4da5-b2d4-63796db100f3" />

---

### 2. Membuat Payload NoSQL Injection

Target challenge ini adalah mengubah semua review sekaligus. Hal ini bisa dilakukan dengan memanipulasi field `id` pada JSON payload.

MongoDB memiliki operator `$ne` (**not equal**) yang dapat dipakai untuk memilih dokumen dengan nilai tertentu yang tidak sama dengan kondisi yang diberikan. Jika digunakan `{"$ne": null}`, maka kondisi ini akan berlaku untuk semua data karena tidak ada `id` yang bernilai `null`.

---

### 3. Mengirimkan Request yang Dimodifikasi

Payload yang sudah dimodifikasi menjadi seperti berikut:

```http
PATCH /rest/products/reviews HTTP/1.1
Host: 127.0.0.1:3000
Content-Type: application/json

{
  "id": {"$ne": null},
  "message": "updated_review_text"
}
```

Ketika dikirimkan, server mengeksekusi query tersebut dan hasilnya **seluruh ulasan produk berubah menjadi pesan yang sama**, bukan hanya satu ulasan saja.

---

## Penjelasan Solusi

* Payload `{"$ne": null}` berhasil mengeksploitasi kelemahan query NoSQL yang tidak aman.
* Aplikasi seharusnya hanya mengizinkan update pada `id` tertentu, tetapi karena input tidak difilter dengan baik, query ini justru berlaku pada semua data di tabel.
* Inilah yang membuat seluruh review dapat dimanipulasi sekaligus.

---

## Rekomendasi Keamanan

1. **Validasi dan Sanitasi Input:** Pastikan semua input pengguna divalidasi dan difilter sebelum diproses di backend.
2. **Gunakan Parameterized Queries:** Manfaatkan mekanisme query terparameter dari library database agar input tidak dieksekusi langsung sebagai bagian dari query.
3. **Batasi Akses:** Terapkan kontrol otorisasi sehingga hanya pemilik review yang berhak mengedit ulasannya.
