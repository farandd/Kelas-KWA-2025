## Deskripsi singkat

**Judul:** Forged Coupon

**Kategori:** Cryptography

---

## Alat yang dipakai

* **Browser** — interaksi dengan Juice Shop dan alat daring (Cryptii) untuk percobaan encoding/decoding.
* **CyberChef / Cryptii** — alat bantu online untuk menganalisis dan menguji berbagai skema encoding (termasuk Z85).
* **package.json (sumber proyek)** — referensi dependensi untuk menemukan petunjuk implementasi encoding.
* (Opsional) **Node.js / npm** atau skrip Python bila ingin mengotomatisasi encoding/decoding.

---

## Metodologi & Penyelesaian (langkah-demi-langkah)

### 1. Observasi dataset kupon

Saya memulai dari daftar kupon yang tersedia/terkumpul dari percobaan sebelumnya:

```
n<MibgC7sn
mNYS#gC7sn
o*IVigC7sn
k#pDlgC7sn
o*I]pgC7sn
n(XRvgC7sn
n(XLtgC7sn
k#*AfgC7sn
q:<IqgC7sn
pEw8ogC7sn
pes[BgC7sn
```

Polanya tidak langsung jelas secara visual (karena karakter non-alfanumerik dan panjang yang seragam), sehingga pendekatan yang layak adalah mencari tahu bagaimana kupon tersebut dihasilkan — apakah ada library encoding atau algoritma di source.

---

### 2. Mencari petunjuk di dependency (`package.json`)

Saya menelusuri `package.json` (yang diperoleh sebelumnya dari server/FTP atau repo) untuk menemukan paket yang berkaitan encoding/decoding. Dari hasil penelaahan, ditemukan paket terkait Z85 — implementasi encoding Z85 (ZeroMQ / Ascii85-varian) yang sering dipakai untuk meng-encode byte array menjadi string yang lebih “terbaca”.

<img width="1552" height="883" alt="Screenshot 2025-09-24 at 17 35 46" src="https://github.com/user-attachments/assets/ff21faa9-e4f6-48f7-b592-346ff1e82799" />

**Mengapa Z85?**

* Z85 adalah skema encoding yang mengubah blok byte (biner) menjadi karakter dari 85-character alphabet untuk representasi yang lebih padat dibanding Base64.
* Jika kupon pada dasarnya adalah hasil encoding byte array (mis. struktur `{MONTH}{YEAR}-{DISCOUNT}`), Z85 akan menghasilkan string dengan karakter non-alfanumerik seperti yang terlihat.

---

### 3. Mencoba decode sample kupon

Untuk mengonfirmasi hipotesis, saya menggunakan Cryptii / CyberChef yang mendukung Z85. Dengan memasukkan salah satu kupon sebagai input dan mencoba decode Z85, outputnya konsisten dan terstruktur. Dari beberapa sample, pola yang muncul menunjukkan format yang dapat dibaca setelah decoding:

```
{3-letter MONTH}{2-digit YEAR}-{DISCOUNT}
```

Contoh decoding yang saya lakukan (ilustrasi di Cryptii):

<img width="1552" height="883" alt="Screenshot 2025-09-24 at 17 53 10" src="https://github.com/user-attachments/assets/9e6ef382-1489-496e-b947-1a9a246827ab" />

Dari hasil decode beberapa sampel, diketahui bahwa kupon bukan random full-entropy — melainkan encoding dari string yang bermakna (bulan, tahun, dan persen diskon). Itu menjelaskan mengapa kupon bisa diprediksi/dibuat ulang jika kita tahu format dan algoritma encoding.

---

### 4. Membangun kupon palsu (forge)

Setelah format diketahui, langkah selanjutnya adalah mereplikasi proses encoding: buat string input yang valid lalu Z85-encode menjadi kupon.

Contoh yang saya buat:

* Input plain: `MAY24-80` (bulan 3 huruf, tahun 2 digit, dash, persen diskon)
* Encode via Z85 → menghasilkan: `o*I]li4$` (contoh; value persis bergantung implementasi Z85 yang dipakai)

**Contoh skrip singkat (Node.js) — cara menghasilkan Z85 (ilustrasi):**

```javascript
// butuh lib z85, mis: npm install z85
const z85 = require('z85');

const plain = 'MAY24-80';
const buf = Buffer.from(plain, 'utf8');
const coupon = z85.encode(buf);
console.log(coupon); // contoh output: o*I]li4$
```

**Catatan:** Pastikan encoding/decoding diimplementasikan persis sama (endianness / blok size) seperti yang dipakai server agar hasil cocok.

---

### 5. Mencoba kupon pada aplikasi

Kupon hasil encoding saya masukkan/ujicobakan pada UI transaksi di Juice Shop. Sistem menerima dan mem-validasi kupon tersebut sebagai kupon yang sah — diskon diterapkan pada transaksi. Hal ini mengonfirmasi bahwa server memverifikasi kupon hanya dengan melakukan decode Z85 dan pengecekan pola/value, tanpa mekanisme autentikasi/validasi tambahan (mis. HMAC, signature, database lookup eksklusif atau nonce).

<img src="../assets/difficulty4/ephemeral_accountant_2.png" alt="decrypting coupons" width="500px">

---

## Analisis kenapa teknik ini berhasil

1. **Reversibility (encoding dapat dibalik):** Z85 adalah encoding — bukan enkripsi. Semua data yang di-encode dapat didecode kembali menjadi plaintext. Jika plaintext berisi informasi deterministik (bulan+tahun+diskon), siapa pun yang mengetahui format itu dapat membuat kupon valid.
2. **Informasi publik / kode sumber:** Petunjuk tentang pemakaian Z85 tersedia dalam `package.json`, sehingga seorang peneliti/pentester bisa menemukan metode pembuatan kupon hanya dari dependency list.
3. **Kurangnya verifikasi server-side yang kuat:** Server tampaknya hanya melakukan decode + pengecekan pola, bukan memverifikasi bahwa kupon dikeluarkan/ditandatangani oleh entitas tepercaya (mis. HMAC dengan secret) atau mencocokkan ke entri database yang disetujui.

---

## Rekomendasi perbaikan (remediation)

1. **Gunakan tanda tangan kriptografis (HMAC/Signature):** Daripada hanya meng-encode plaintext, sertakan signature berbasis secret server (mis. HMAC-SHA256) pada payload. Server harus memverifikasi signature sebelum menerima kupon.
2. **Simpan kupon di server / database:** Kupon yang sah harus berpasangan dengan entri server-side (ID unik, masa berlaku, jumlah penggunaan). Server tidak boleh membuat keputusan hanya berdasarkan isi kupon yang dikirim klien.
3. **Gunakan token acak yang tidak dapat diprediksi:** Alihkan dari format yang mudah ditebak (mis. `MONTHYEAR-DISCOUNT`) ke token acak panjang (cryptographically secure random), lalu simpan mapping ke potongan di server.
4. **Minimalkan exposure dependency info di client:** Jangan publish informasi internal yang menunjukkan algoritma pembuatan kupon di klien (package.json di bundle klien). Jika paket itu hanya dipakai server-side, jangan sertakan paket atau manifest yang menyingkapkan library itu ke publik.
5. **Monitoring & rate-limiting:** Pasang logging dan limit untuk percobaan redeem kupon yang berulang atau pola penggunaan abnormal.

---

## Kesimpulan

Forged Coupon berhasil dibuat karena kombinasi: penggunaan encoding reversible (Z85) untuk mewakili kupon, dan kurangnya verifikasi kriptografis/penyimpanan authoritative server-side. Dengan mengubah alur pembuatan kupon ke skema yang melibatkan secret server dan pencatatan server-side, kerentanan ini dapat ditutup.
