# Unlock Premium Challenge

## Ringkasan 

**Judul:** Unlock Premium Challenge

**Kategori:** Cryptographic Issues

---

## Alat yang dipakai

* Browser — untuk menelusuri situs, mengunduh file dari direktori yang terekspos.
* OpenSSL atau skrip Python kecil — untuk mendekripsi string terenkripsi.
* (Opsional) text editor — untuk melihat isi file kunci.

---

## Detail langkah-langkah penyelesaian

### 1) Menemukan direktori kunci yang terekspos

Dengan teknik enumerasi (mis. gobuster / perambanan direktori) saya menemukan endpoint yang mem-list file di direktori `encryptionkeys`:

```
http://127.0.0.1:3000/encryptionkeys
```
<img width="1552" height="883" alt="Screenshot 2025-09-24 at 17 13 02" src="https://github.com/user-attachments/assets/dcdbf3c9-2748-440c-9d24-ebe073d88100" />

Dari sana saya mengunduh file `premium.key`.

### 2) Memeriksa isi file kunci

Isi `premium.key` adalah sebuah string yang dipisahkan oleh titik:

<img width="455" height="63" alt="Screenshot 2025-09-24 at 17 18 34" src="https://github.com/user-attachments/assets/1d457324-f5bc-4288-a002-b3d501f474a8" />

```
1337133713371337.EA99A61D92D2955B1E9285B55BF2AD42
```

Interpretasi praktisnya: file ini berisi dua komponen — bagian pertama (sebelum titik) digunakan sebagai IV (initialization vector) dan bagian kedua adalah kunci simetris (dalam format hex). Dalam konteks ini kunci hex panjangnya mengindikasikan AES dengan panjang kunci yang biasa dipakai (16 bytes = AES-128).

> Catatan: format kunci berbeda-beda antar tantangan; selalu periksa apakah nilai tersebut hex, base64, atau plain text. Di sini bagian kedua jelas hex.

### 3) Menemukan ciphertext yang perlu didekripsi

Berdasarkan petunjuk challenge (dan referensi panduan Juice Shop), ciphertext yang terenkripsi berada di sebuah komentar HTML pada salah satu file sumber aplikasi. Nilai base64-nya adalah:

```
IvLuRfBJYlmStf9XfL6ckJFngyd9LfV1JaaN/KRTPQPidTuJ7FR+D/nkWJUF+0xUF07CeCeqYfxq+OJVVa0gNbqgYkUNvn//UbE7e95C+6e+7GtdpqJ8mqm4WcPvUGIUxmGLTTAC2+G9UuFCD1DUjg==
```

### 4) Menentukan parameter dekripsi yang tepat

Dari `premium.key` kita dapatkan:

* IV (string): `1337133713371337`
* Key (hex): `EA99A61D92D2955B1E9285B55BF2AD42`

Untuk mendekripsi ciphertext (yang adalah base64 dari blok cipher), kita harus memastikan:

* Cipher yang dipakai (dari ukuran key kemungkinan **AES-128-CBC**).
* Key diberikan dalam hex.
* IV harus diberikan dalam bentuk byte sequence. Pada file ini IV tampak sebagai string ASCII `1337133713371337` — dalam banyak solusi Juice Shop, IV digunakan sebagai ASCII 16-byte (bukan hex). Jadi kita perlu menerjemahkan string IV menjadi byte-bytes ASCII sebelum memakainya.

> Saya menyertakan dua cara untuk mendekripsi: **OpenSSL** (jika Anda ingin command-line) dan **Python** (lebih eksplisit tentang konversi IV).

---

## 5) Cara mendekripsi — contoh perintah yang bekerja

### A. Cara cepat (OpenSSL)

Jika IV Anda adalah string ASCII 16 byte seperti `1337133713371337`, Anda perlu menyediakannya kepada OpenSSL dalam bentuk hex. Konversi ASCII → hex untuk `1337133713371337` menghasilkan `31333337313333373133333731333337` (setiap karakter → kode hex ASCII).
Contoh (bash) — perhatikan penggantian `-aes-128-cbc` sesuai panjang kunci:

```bash
# simpan ciphertext ke file ct.b64 atau echo langsung
echo "IvLuRfBJYlmStf9XfL6ckJFngyd9LfV1JaaN/KRTPQPidTuJ7FR+D/nkWJUF+0xUF07CeCeqYfxq+OJVVa0gNbqgYkUNvn//UbE7e95C+6e+7GtdpqJ8mqm4WcPvUGIUxmGLTTAC2+G9UuFCD1DUjg==" > ct.b64

# convert ASCII IV '1337133713371337' to hex: 31333337313333373133333731333337
# use key hex EA99A61D92D2955B1E9285B55BF2AD42
openssl enc -d -aes-128-cbc -K EA99A61D92D2955B1E9285B55BF2AD42 -iv 31333337313333373133333731333337 -in ct.b64 -a -A
```

Perintah di atas mendecode base64 (`-a`) lalu mendekripsi dengan AES-128-CBC menggunakan key hex dan IV hex.

### B. Cara yang lebih aman dan jelas (Python)

Jika Anda lebih nyaman dengan skrip, berikut contoh Python (menggunakan `pycryptodome`) yang otomatis menangani base64 → bytes dan penggunaan IV sebagai ASCII:

```python
# simpan sebagai decrypt_premium.py
from base64 import b64decode
from Crypto.Cipher import AES

cipher_b64 = "IvLuRfBJYlmStf9XfL6ckJFngyd9LfV1JaaN/KRTPQPidTuJ7FR+D/nkWJUF+0xUF07CeCeqYfxq+OJVVa0gNbqgYkUNvn//UbE7e95C+6e+7GtdpqJ8mqm4WcPvUGIUxmGLTTAC2+G9UuFCD1DUjg=="
iv_ascii = "1337133713371337"
key_hex = "EA99A61D92D2955B1E9285B55BF2AD42"

cipher_bytes = b64decode(cipher_b64)
key_bytes = bytes.fromhex(key_hex)
iv_bytes = iv_ascii.encode('ascii')   # menggunakan IV sebagai ASCII bytes

cipher = AES.new(key_bytes, AES.MODE_CBC, iv=iv_bytes)
plaintext = cipher.decrypt(cipher_bytes)

# kemungkinan padding PKCS#7 — hapus
pad_len = plaintext[-1]
plaintext = plaintext[:-pad_len]

print(plaintext.decode('utf-8', errors='replace'))
```

Jalankan:

```bash
pip install pycryptodome
python decrypt_premium.py
```

Output yang saya peroleh (sesuai guideline/solution) adalah URL tersembunyi:

```
http://localhost:3000/this/page/is/hidden/behind/an/incredibly/high/paywall/that/could/only/be/unlocked/by/sending/1btc/to/us
```

> Catatan: tergantung variasi challenge, cipher mode atau format IV/key bisa berbeda — jika dekripsi gagal, coba: (1) gunakan AES-256 jika key panjangnya 32 bytes; (2) pastikan IV panjangnya 16 bytes; (3) jika IV tampak hex, gunakan langsung `bytes.fromhex(iv_str)`.

---

## 6) Mengakses konten premium

Setelah URL berhasil didekripsi, membuka URL tersebut di browser langsung menampilkan halaman premium (konten dibuka tanpa pembayaran). Ini menyelesaikan challenge.

---

## Penjelasan kenapa ini bekerja

* Penyerang mengakses *secret* (kunci) yang seharusnya tidak tersedia ke publik.
* Ciphertext disimpan di tempat yang dapat diakses (mis. komentar di file sumber).
* Dengan kunci dan IV yang benar, ciphertext didekripsi menjadi URL yang membuka konten.
* Risiko nyata: jika kunci/IV terekspos pada server sebenarnya, attacker bisa mengungkap data sensitif terenkripsi atau mem-bypass mekanisme proteksi.

---

## Rekomendasi mitigasi / remediasi

1. **Jangan letakkan kunci di direktori publik** — gunakan vault (HashiCorp Vault, cloud KMS) atau environment variables yang tidak bisa di-request via HTTP.
2. **Pisahkan artefak dev vs prod** — komentar dev/placeholder/artefak uji tidak boleh ikut ke build produksi.
3. **Batasi akses direktori** — server harus menolak listing direktori dan mengamankan file sensitif.
4. **Rotasi kunci** — bila kunci terekspos, rotasi kunci dan enkripsi ulang data yang relevan.
5. **Code review & pipeline checks** — pipeline CI/CD harus memblok deploy yang membawa file kunci atau komentar sensitif.
