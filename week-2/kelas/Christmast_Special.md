# Christmas Special

## Deskripsi

Tugas ini adalah **melakukan order terhadap produk Christmas Special tahun 2014** dengan memanfaatkan **SQL Injection**.

## Resource

* [Union attacks - PortSwigger](https://portswigger.net/web-security/sql-injection/union-attacks)

---

## Langkah - Langkah

### 1. Dapatkan produk *Christmas Special* dari database

Karena endpoint yang digunakan sama dengan soal **Database Schema**, kita bisa langsung membuat payload berdasarkan schema tabel yang sudah diperoleh sebelumnya.

Payload:

```
aaaa'))+UNION+SELECT+*+FROM+Products--
```

<img width="1919" height="1139" alt="image" src="https://github.com/user-attachments/assets/ec5f2fc8-34ed-4fa6-8ed6-30b38d2f62fe" />

Dari hasilnya, ditemukan bahwa **Product ID** untuk *Christmas Special* adalah **10**.

---

### 2. Tambahkan produk ke keranjang

* Tambahkan produk apa saja ke keranjang terlebih dahulu.
* Lalu intercept request menggunakan Burp Suite.

<img width="1919" height="1133" alt="image" src="https://github.com/user-attachments/assets/e75e9b07-3504-4cbb-aba6-5710afd7814d" />

* Ubah parameter `ProductId` menjadi `10`.

<img width="1919" height="1134" alt="image" src="https://github.com/user-attachments/assets/1ef4889c-6530-4ee7-9153-50934c5bd5d2" />

---

### 3. Verifikasi di keranjang

Cek keranjang belanja → produk **Christmas Special** sekarang berhasil ditambahkan.

<img width="980" height="527" alt="Screenshot 2025-09-10 at 10 27 00" src="https://github.com/user-attachments/assets/d309d9b3-b751-412e-a37b-38a593b3ea4b" />

---

## Catatan

* Karena schema tabel sudah didapatkan dari soal sebelumnya, payload UNION bisa langsung digunakan untuk membaca tabel **Products**.
* Query SQL aplikasi hanya menampilkan produk yang tidak dihapus (`deletedAt IS NULL`).
* Dengan payload:

  ```
  aaaa'))+UNION+SELECT+*+FROM+Products--
  ```

  semua produk ditampilkan, termasuk yang sudah dihapus. Itulah sebabnya produk **Christmas Special 2014** bisa muncul kembali dan akhirnya berhasil dipesan.
