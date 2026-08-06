---
name: developer-bukaolshop
description: WAJIB dipakai untuk SETIAP proyek yang berhubungan dengan bukaOlshop (aplikasi toko online Android white-label) — tanpa terkecuali. Gunakan skill ini setiap kali user menyebut bukaOlshop, "closed API", "open API bukaolshop", kostum halaman/shortcode {{token_user}}/{{id_user}}, potong/tambah saldo member, notifikasi ke user olshop, callback/webhook transaksi bukaolshop, bot Telegram untuk kelola saldo/transaksi olshop, atau panel admin web untuk olshop — bahkan jika user tidak secara eksplisit menyebut kata "skill" atau "API". Selalu konsultasikan skill ini di awal SEBELUM mulai coding proyek bukaolshop apapun, supaya struktur endpoint, format request/response, dan pola arsitekturnya benar sejak awal. Skill ini sudah menyertakan template kode siap pakai (client API PHP & Node.js, callback listener PHP & Node.js/Express) di folder scripts/ — pakai file-file itu sebagai starting point, jangan tulis client API atau callback listener dari nol.
---

# Developer bukaOlshop

Skill ini merangkum cara kerja platform **bukaOlshop** (aplikasi toko online Android white-label), lengkap dengan contoh request & response tiap endpoint, supaya Claude bisa langsung membangun sistemnya tanpa banyak tanya dan tanpa perlu browsing dokumentasi dari nol setiap kali.

**Aturan pemakaian**: skill ini WAJIB dipakai untuk setiap proyek yang menyentuh bukaOlshop, apapun bentuknya (backend API, bot Telegram, panel admin, kostum halaman, sistem saldo, dsb). Konsultasikan skill ini di awal pengerjaan supaya tidak salah nama parameter/endpoint dan supaya proses pembuatan proyek lebih cepat.

**Prinsip kerja**: jangan interogasi user dengan banyak pertanyaan. Kalau jenis proyek sudah jelas dari permintaan user (misal "bikinin sistem topup otomatis" atau "bikin bot telegram cek saldo"), langsung asumsikan pola standar di bawah dan mulai bangun. Tanya HANYA kalau ada pilihan teknis yang benar-benar tidak bisa ditebak (misal: bahasa backend PHP/Node, mau pakai database apa) — dan itupun cukup 1 pertanyaan singkat, bukan checklist panjang.

---

## 1. Dua jenis API bukaOlshop

| | **Open API** | **Closed API** |
|---|---|---|
| Base URL | `https://openapi.bukaolshop.net/v1` | `https://bukaolshop.net/api/v1` |
| Dipakai di | HTML "Kostum Halaman" yang di-render **di dalam APK olshop** | Backend/server milik developer (PHP/Node/dll) |
| Auth | `token` (token aplikasi) + shortcode `{{token_user}}` & `{{id_user}}` yang auto-di-replace APK | Header `Authorization: Bearer API_KEY` |
| Contoh pakai | Tampilkan saldo/poin/nama user di halaman custom dalam apk | Tambah/kurang saldo, kirim notifikasi, ambil daftar transaksi, kelola produk & member |

Shortcode `{{token_user}}` dan `{{id_user}}` **hanya bekerja kalau HTML diletakkan langsung di fitur "Kostum Halaman" bukaOlshop**, bukan di website eksternal. Kalau butuh data user dari luar, redirect dulu lewat halaman kostum:
```js
window.location.replace("https://domainmu.com/cek.php?token_user={{token_user}}&id_user={{id_user}}");
```

---

## 2. Arsitektur proyek standar

```
[Kostum Halaman / HTML dalam APK]  --shortcode token_user/id_user-->  [Backend Server]
                                                                              |
                                                                    Closed API bukaOlshop
                                                                    (Bearer API key)
                                                                              |
                                          +---------------------------------+---------------------------------+
                                          |                                                                     |
                              [Panel Admin Web (login)]                                          [Bot Telegram (id chat admin)]
                              - setting API key, secret key                                      - notifikasi transaksi masuk
                              - lihat log transaksi/saldo                                        - command cek saldo/transaksi
                              - konfigurasi produk yang di-otomasi                                - approve manual jika perlu
                                          |
                                [Callback Listener endpoint]
                                <-- bukaOlshop kirim webhook transaksi baru / perubahan status -->
```

Backend inti selalu berisi 3 bagian ini, apapun proyeknya:
1. **Client Closed API** — wrapper untuk request ke `bukaolshop.net/api/v1` (auth Bearer, handle rate limit & error code).
2. **Callback listener** — endpoint publik yang menerima POST dari server bukaOlshop tiap ada transaksi baru / perubahan status.
3. **Aksi otomatis** — logic yang jalan setelah callback masuk: potong/tambah saldo, proses produk (misal beli pulsa ke provider lain), kirim notifikasi ke user, dan/atau lapor ke bot Telegram admin.

---

## 3. Pola proyek yang paling sering diminta

**A. Auto-proses transaksi / auto-order produk digital (PPOB, dsb)**
- Terima callback `tipe_callback: transaksi_baru`.
- Validasi `secret_key_callback` cocok dengan yang di-set di aplikasi bukaOlshop.
- Cek `status_transaksi == "lunas"` — **kalau `pending`, JANGAN diproses**.
- Ambil data produk & catatan (misal nomor HP) dari field yang dikirim, validasi formatnya.
- Eksekusi order ke provider lain, lalu (opsional) kirim notifikasi.

**B. Sistem saldo (potong/tambah otomatis)**
- Pakai `POST /member/saldo` dengan `tipe: tambah` atau `tipe: kurang`.
- Selalu sertakan `catatan_saldo` supaya ada jejak audit di riwayat saldo user.
- Set `notifikasi: true` + `judul_notifikasi`/`pesan_notifikasi` kalau user perlu tahu saldonya berubah.
- Untuk topup dari payment gateway pihak ketiga: override halaman topup bukaOlshop dulu untuk dapat `token_topup`, baru konfirmasi lewat `POST /member/topup`.
- Sebelum `tipe: kurang`, sebaiknya cek saldo dulu via `GET /member/saldo` — pengurangan lebih besar dari saldo akan ditolak (403).

**C. Notifikasi ke user**
- Notifikasi umum (tanpa transaksi): `POST /member/notifikasi`.
- Notifikasi terkait perubahan saldo/topup: cukup pakai parameter `notifikasi` di endpoint saldo/topup — tidak perlu call terpisah.
- Gambar notifikasi wajib pakai image host resmi bukaOlshop.

**D. Bot Telegram**
- Chat ID admin disimpan sebagai konfigurasi (env var / tabel setting), bukan hardcode.
- Fungsi umum: forward notifikasi transaksi baru dari callback listener ke Telegram, command cek saldo/detail member, command trigger tambah/kurang saldo manual.
- Amankan command sensitif (ubah saldo) dengan whitelist chat ID admin.

**E. Panel admin web**
- Login untuk akses setting: API key bukaOlshop, secret key callback, URL callback, daftar aturan otomasi produk.
- Halaman log transaksi & perubahan saldo (simpan salinan callback ke database sendiri untuk audit & replay).
- Desain modern & rapi sesuai identitas proyek (nama toko dari `GET /aplikasi/info`) — pakai skill `frontend-design` kalau panel admin dibangun sebagai web app.

---

## 4. Template kode siap pakai (folder `scripts/`)

Jangan tulis client API atau callback listener dari nol — pakai template ini sebagai starting point, lalu sesuaikan bagian `TODO` dengan kebutuhan proyek:

| File | Fungsi | Kapan dipakai |
|---|---|---|
| `scripts/BukaOlshopClient.php` | Class PHP berisi semua method Closed API (produk, member, saldo, notifikasi, topup, chat, transaksi) | Backend PHP apapun yang perlu manggil Closed API |
| `scripts/bukaOlshopClient.js` | Versi Node.js dari client di atas, pakai native `fetch` (Node 18+), tanpa dependency tambahan | Backend Node.js/Express apapun yang perlu manggil Closed API |
| `scripts/callback-listener.php` | Endpoint publik contoh untuk terima webhook `transaksi_baru`: validasi `secret_key_callback`, cek `status_transaksi == "lunas"`, idempotency check sederhana | Proyek Auto-order / potong saldo otomatis (Pola A) berbasis PHP |
| `scripts/callback-listener.js` | Versi Express dari callback listener di atas, sekaligus contoh route kedua untuk `perubahan_status_transaksi` | Proyek Auto-order / potong saldo otomatis (Pola A) berbasis Node.js |

Cara pakai cepat:
1. Copy file client (`BukaOlshopClient.php` atau `bukaOlshopClient.js`) ke proyek, inject API key dari env var — **jangan hardcode**.
2. Copy `callback-listener.*` yang sesuai stack, ganti bagian idempotency check (placeholder file JSON) dengan tabel/collection database sungguhan di proyek.
3. Isi bagian `// TODO` di callback listener dengan logic bisnis proyek (order ke provider lain, update saldo pakai client di poin 1, forward ke bot Telegram, dsb — sesuai pola di bagian 3).
4. Kalau proyek butuh endpoint yang belum ada method-nya di client (jarang terjadi), tambahkan method baru mengikuti pola `request()` yang sudah ada, jangan bikin cara request baru.

---

## 5. Referensi endpoint lengkap + contoh respons

### Auth (Closed API)
Header wajib di setiap request:
```
Authorization: Bearer API_KEY_ANDA
```
Salah/tidak ada → `401 Unauthorized`.

Contoh PHP:
```php
$token = "API KEY ANDA";
$header = array("Authorization: Bearer " . $token);
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, "URL Endpoint API");
curl_setopt($ch, CURLOPT_HTTPHEADER, $header);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$hasil = curl_exec($ch);
curl_close($ch);
```

### GET Informasi Aplikasi
`GET /aplikasi/info`
```json
{
  "code": 200,
  "status": "ok",
  "nama_toko": "OlshopTest",
  "nama_aplikasi": "OlshopTest",
  "nama_package": "com.olshopgue",
  "icon_aplikasi": "https://xxxxxxxx/xxx/xxx/xxx/xxx.png",
  "masa_aktif_premium": "2021-09-16",
  "status_premium": "lite",
  "tanggal_daftar_aplikasi": "2019-11-08 20:29:30",
  "telah_verifikasi_identitas": false,
  "email_pemilik_olshop": "cek@gmail.com",
  "nama_pemilik_olshop": "testing user",
  "hp_pemilik_olshop": "082234327658"
}
```

### GET Detail Produk
`GET /produk/id?id_barang=1234567`
```json
{
  "code": 200,
  "status": "ok",
  "id_barang": "176941",
  "nama_barang": "Produk 2",
  "harga_barang": 34344,
  "kondisi_barang": "Baru",
  "status_produk": "live",
  "stok_barang": 43,
  "berat": 54,
  "tanggal": "2020-08-08 04:58:48",
  "id_kategori": "65258",
  "data_kategori": { "nama_kategori": "judul kategori", "gambar_kategori": "url gambar kategori" },
  "harga_grosir": [ { "jumlah": "5", "harga": "60000" } ]
}
```

### GET Detail Member
`GET /member/id?id_user=xxx` atau `?email_user=xxx` (pilih salah satu)
```json
{
  "code": 200,
  "status": "ok",
  "id_user": "130173",
  "email_user": "email@gmail.com",
  "nama_user": "Nama user",
  "tanggal_lahir": "2000-01-01",
  "jenis_kelamin": "Laki-laki",
  "nomor_telepon": "0822882432535",
  "status_akun": "verified",
  "kode_referral": "MYTANPA",
  "tanggal_daftar": "2020-09-26 02:16:33",
  "data_alamat": [
    {
      "nama_penerima": "Nama user",
      "kecamatan": "Abiansemal",
      "kota": "Badung",
      "provinsi": "Bali",
      "nomor_telepon": "4949",
      "kode_pos": "80351",
      "alamat_lengkap": "Alamat lengkap user"
    }
  ]
}
```
Response codes: `200` data ditemukan · `400` id_user/email_user belum diisi · `404` data tidak ditemukan.

### GET Saldo Member
`GET /member/saldo?id_user=xxx` atau `?email_user=xxx`
```json
{
  "code": 200,
  "status": "ok",
  "id_user": "289820",
  "email_user": "cek4@gmail.com",
  "nama_user": "Nama member",
  "transaksi_terakhir": "2021-04-06 01:58:39",
  "jumlah_saldo": 90000,
  "riwayat_saldo": [
    {
      "informasi_catatan": "Member mendapatkan komisi referral sebagai Downline",
      "jumlah_dana": 10000,
      "tanggal": "2020-10-05 05:14:10"
    }
  ]
}
```

### POST Kirim Notifikasi ke Member
`POST /member/notifikasi`

Body: `id_user`/`email_user` (pilih salah satu), `judul_notifikasi` (wajib), `pesan_notifikasi` (wajib), `url_gambar` (opsional, wajib pakai image host resmi bukaOlshop).
```php
$post_body = array(
  "email_user" => "blabla@gmail.com",
  "judul_notifikasi" => "Judul notifikasi anda",
  "pesan_notifikasi" => "Pesan notifikasi anda",
  "url_gambar" => "https://images.bukaolshop.com/developer/1/1ab38188996f95728.png"
);
```
Contoh respons:
```json
{
  "code": 200,
  "status": "ok",
  "id_user": "289820",
  "status_gambar": "valid",
  "token_notifikasi": "626c3d67-0190-4a8a-b70f-d014223fd533"
}
```

### POST Ubah Saldo Member
`POST /member/saldo`

Body: `id_user`/`email_user` (salah satu), `tipe` (`tambah`/`kurang`, wajib), `jumlah` (wajib), `pin` (opsional, PIN 6 digit member), `catatan_saldo` (disarankan diisi), `notifikasi` (opsional, `true` untuk kirim notif), `judul_notifikasi`/`pesan_notifikasi` (opsional).
```php
$post_body = array(
  "email_user" => "blabla@gmail.com",
  "tipe" => "tambah",
  "jumlah" => 20000,
  "notifikasi" => true
);
```
Contoh respons:
```json
{
  "code": 200,
  "status": "ok - Saldo berhasil ditambah",
  "id_user": "289820",
  "email_user": "emailuser@gmail.com",
  "nama_user": "Nama user",
  "jumlah": 30000,
  "tipe": "tambah",
  "id_perubahan": "456406"
}
```
Response codes: `200` OK · `400` id_user/email_user belum diisi · `403` **Forbidden**, pengurangan lebih besar dari saldo member saat ini · `404` data tidak ditemukan.

### POST Konfirmasi TopUp Member
`POST /member/topup`

Dipakai setelah override halaman topup bukaOlshop untuk dapat `token_topup`.
Body: `token_topup` (wajib), `validasi_jumlah_topup` (opsional — isi dengan `total_topup` dari server bukaOlshop, kalau beda topup DITOLAK), `notifikasi`, `judul_notifikasi`, `pesan_notifikasi` (opsional).
```php
$post_body = array(
  "token_topup" => "xxxxxxxxxxxxxxxxxxxx",
);
```
Contoh respons:
```json
{
  "code": 200,
  "status": "ok - Saldo berhasil ditambah",
  "id_user": "289820",
  "email_user": "emailuser@gmail.com",
  "nama_user": "Nama user",
  "jumlah": 30000,
  "tipe": "tambah",
  "id_perubahan": "456406"
}
```
> Catatan: Jumlah topup yang masuk ke saldo = jumlah topup + kode unik.

Response codes: `200` OK · `400` token_topup belum diisi / jumlah konfirmasi tidak cocok dengan server · `404` data tidak ditemukan.

### POST Kirim Pesan (Chat)
`POST /chat/create`

Body: `id_user` (wajib), `pesan_teks` (wajib), `id_barang` (opsional — isi kalau mau kirim sebagai chat produk, default chat umum).
```php
$post_body = array(
  "id_user" => "xxxx",
  "pesan_teks" => "ini adalah pesan anda"
);
```
Contoh respons:
```json
{
  "code": 200,
  "status": "ok - Pesan berhasil dikirim",
  "id_user": "130173",
  "email_user": "emailuser@gmail.com",
  "nama_user": "Nama User",
  "id_barang": "257304",
  "pesan_teks": "Assalamualaikum, Apa kabar?"
}
```

### GET Daftar List Transaksi
`GET /transaksi/list?page=1`
```json
{
  "code": 200,
  "status": "ok",
  "data": [
    {
      "nomor_pembayaran": "2853130173354",
      "id_user": "130173",
      "status_bayar": "pending",
      "status_pengiriman": "belum diproses",
      "tanggal": "2020-10-12 01:15:04"
    },
    {
      "nomor_pembayaran": "81168130173359",
      "id_user": "130173",
      "status_bayar": "lunas",
      "status_pengiriman": "selesai",
      "tanggal": "2020-10-11 08:20:00"
    }
  ]
}
```
`GET /transaksi/id?nomor_pembayaran=xxx` untuk detail satu transaksi (berguna untuk cross-check data callback sebelum diproses).

### Open API — GET Informasi User (dalam Kostum Halaman)
`GET https://openapi.bukaolshop.net/v1/user/info?token=xxx&token_user={{token_user}}&id_user={{id_user}}`
```json
{
  "code": 200,
  "status": "ok",
  "data": {
    "nama_user": "User name",
    "email_user": "user@gmail.com",
    "jumlah_saldo": "50000",
    "jumlah_poin": "500",
    "nama_membership": "Basic reseller",
    "kode_referral": "MYREF"
  }
}
```
Perlu APK versi patch 62.5+.

---

## 6. Callback (Webhook) — WAJIB dipahami untuk proyek otomasi

bukaOlshop mengirim POST ke URL callback yang di-set di aplikasi Android bukaOlshop, tiap ada event transaksi. Disarankan pakai **2 URL berbeda** untuk `transaksi_baru` dan `perubahan_status_transaksi`.

### Field yang dikirim server saat callback:
| Field | Keterangan |
|---|---|
| `tipe_callback` | `transaksi_baru` atau `perubahan_status_transaksi` |
| `secret_key_callback` | Kode rahasia dari APK bukaOlshop. **WAJIB dicocokkan** di server — kalau beda, JANGAN diproses (kemungkinan request palsu). |
| `nomor_pembayaran` | ID transaksi — bisa dipakai untuk fetch detail via `GET /transaksi/id` |
| `status_transaksi` | `lunas` atau `pending`. **Hanya proses transaksi kalau `lunas`.** |
| `tipe_transaksi` | `COD MANUAL`, `COD JNT`, `DELIVERY COD`, `DELIVERY`, `PRODUK DIGITAL`, `EKSPEDISI`, `MULTI_PENGIRIMAN` |
| `catatan` | Data tambahan yang diisi member (misal nomor HP untuk PPOB) — **wajib divalidasi formatnya** sebelum dipakai untuk auto-order |

### Alur kerja disarankan untuk auto-order (misal PPOB):
1. Terima POST callback.
2. Cocokkan `secret_key_callback`.
3. Cek `tipe_callback == "transaksi_baru"`.
4. Cek `status_transaksi == "lunas"` — kalau bukan, simpan tapi jangan eksekusi.
5. Simpan `nomor_pembayaran` yang sudah diproses (cegah double-proses kalau ada retry callback).
6. Ambil & validasi `catatan` (misal cek format nomor HP).
7. Sarannya: hindari fitur varian produk digital, supaya identifikasi produk cukup lewat `id_barang` saja.
8. Eksekusi order ke provider lain / update saldo / kirim notifikasi sesuai kebutuhan proyek.

---

## 7. Rate Limit & Error Codes

- Rate limit reset tiap hari. Kena limit → `429 Too Many Requests`. Client API wajib cek status code ini dan retry dengan backoff.

| Code | Arti |
|---|---|
| 400 | Bad Request — data yang diinput salah/kurang |
| 401 | Unauthorized — API key salah atau header Authorization tidak terkirim benar |
| 403 | Forbidden — autentikasi berhasil tapi tidak punya hak (misal pengurangan saldo > saldo tersedia) |
| 404 | Not Found — data yang dicari tidak ada |
| 405 | Method Not Allowed — method request salah |
| 429 | Too Many Requests — kena rate limit |
| 500 | Internal Server Error — masalah di server bukaOlshop, coba lagi nanti |
| 503 | Service Unavailable — server bukaOlshop offline/maintenance |

---

## 8. Hal wajib diperhatikan (sering jadi bug)

- **Auth Closed API** selalu `Authorization: Bearer <API_KEY>` di header, bukan di query string.
- **Idempotency callback**: bukaOlshop bisa retry callback. Simpan `nomor_pembayaran` yang sudah diproses supaya tidak double-eksekusi/double-potong saldo.
- **id_user vs email_user**: banyak endpoint terima salah satu saja (pilih satu, jangan isi dua-duanya).
- Jangan pernah expose API key / secret key callback ke frontend atau ke HTML kostum halaman — hanya boleh ada di backend.

---

## 9. Saran optimasi proyek

- **Simpan API key & secret key di environment variable**, bukan hardcode di kode.
- **Buat lapisan queue** (misal Redis/queue table sederhana) untuk callback yang butuh proses ke pihak ketiga (misal order PPOB), supaya tidak memblokir response ke bukaOlshop dan tidak ada transaksi yang "hilang".
- **Log semua request & response API** (termasuk callback mentah) ke database/file, minimal 30 hari — sangat membantu saat debug transaksi yang gagal/dispute.
- **Buat endpoint health-check** terpisah dari callback listener supaya gampang memonitor uptime backend.
- **Pisahkan URL callback** untuk `transaksi_baru` dan `perubahan_status_transaksi` supaya logic tidak bercampur.
- **Retry mechanism dengan backoff** untuk pemanggilan ke Closed API bukaOlshop kalau kena `429`/`500`.
- **Rencanakan dari awal untuk multi-tenant** kalau proyeknya akan dipakai lebih dari satu olshop — pisahkan API key/secret key per tenant di database, jangan satu env var global.
- Untuk bot Telegram: pakai webhook (bukan long-polling) kalau backend sudah punya HTTPS publik — lebih hemat resource dan konsisten dengan pola callback listener yang sudah ada.
