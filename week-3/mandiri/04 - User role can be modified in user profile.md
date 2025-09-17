# User role dapat dimodifikasi di profil pengguna

Lab ini memiliki panel admin di path `/admin`. Panel tersebut hanya bisa diakses oleh pengguna yang sedang login dengan **roleid = 2**.

Tujuan: **Selesaikan lab dengan cara mengakses panel admin dan menghapus user bernama `carlos`.**

Kita dapat login ke akun sendiri dengan kredensial berikut:

```
username: wiener
password: peter
```

---

### Referensi:

* [https://portswigger.net/web-security/access-control](https://portswigger.net/web-security/access-control)

---

### 1. Login

Proses login dilakukan dengan sebuah request `POST`:

![img](images/User%20role%20can%20be%20modified%20in%20user%20profile/2.png)

Setelah login berhasil, server memberikan respon `302 redirect` untuk mengarahkan pengguna ke halaman berikutnya:

![img](images/User%20role%20can%20be%20modified%20in%20user%20profile/3.png)

---

### 2. Fitur ubah email

Terdapat sebuah fungsi untuk memperbarui alamat email dengan menggunakan request `POST`:

![img](images/User%20role%20can%20be%20modified%20in%20user%20profile/4.png)

Ketika request dikirim, server merespons dengan menampilkan informasi personal dari pengguna:

![img](images/User%20role%20can%20be%20modified%20in%20user%20profile/5.png)

---

### 3. Eksploitasi – Menambahkan parameter `roleid`

Walaupun form ini seharusnya hanya untuk mengganti email, kita bisa menyisipkan parameter tambahan yaitu `roleid`. Dengan menambahkan `"roleid":2`, kita dapat mengubah peran akun dari user biasa menjadi admin.

Contoh request manipulasi:

```http
POST /my-account/change-email HTTP/2
...

{"email":"test@test.com", "roleid":2}
```

![img](images/User%20role%20can%20be%20modified%20in%20user%20profile/6.png)

Setelah request berhasil, role akun kita berubah menjadi admin:

![img](images/User%20role%20can%20be%20modified%20in%20user%20profile/7.png)

---

### 4. Akses panel admin dan hapus user `carlos`

Karena akun kita sekarang memiliki role admin, kita bisa masuk ke halaman `/admin`. Dari sana tersedia fungsi untuk menghapus pengguna. Kita gunakan fungsi tersebut untuk menghapus user `carlos`.

![img](images/User%20role%20can%20be%20modified%20in%20user%20profile/8.png)

