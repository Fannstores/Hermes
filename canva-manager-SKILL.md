---
name: canva-manager
description: Skill untuk mengelola desain Canva secara mandiri lewat MCP server Canva — membuat, mengedit, mencari, export, dan resize desain. Skill ini juga belajar dari setiap sesi kerja dan selalu melaporkan hasil akhir ke user dengan jelas.
triggers:
  - canva
  - desain
  - poster
  - thumbnail
  - brand kit
  - export desain
  - resize desain
mcp_required: canva
---

# Canva Manager Skill

## Tujuan
Skill ini membuat Hermes bisa mengelola Canva secara otonom: memahami maksud user meski instruksinya singkat/kurang detail, mengambil keputusan desain yang masuk akal, mengeksekusi lewat MCP `canva`, dan **selalu melaporkan hasil akhir secara eksplisit ke user** — bukan cuma bilang "sudah selesai" tanpa bukti.

## Prinsip Kerja

### 1. Selalu klarifikasi minimum, putuskan sisanya sendiri
Kalau user cuma bilang "buatkan poster promosi", jangan tanya 5 pertanyaan sekaligus. Cukup tanyakan **maksimal 1 hal paling krusial** (misal: platform/ukuran), lalu ambil keputusan wajar untuk sisanya (warna, gaya, font) berdasarkan konteks yang ada — dan sebutkan asumsi itu di laporan akhir.

### 2. Selalu verifikasi hasil sebelum lapor sukses
Setelah tool call (create/edit/export/resize), **jangan langsung bilang berhasil**. Cek balik:
- Apakah hasil dari `mcp_canva_*` benar-benar mengembalikan ID desain / link / status sukses?
- Kalau ada link/preview yang bisa ditunjukkan, tunjukkan itu ke user — jangan cuma deskripsi teks.
- Kalau export, sebutkan format & lokasi/link file hasil.

### 3. Format laporan hasil ke user (WAJIB setiap kali selesai satu tugas Canva)
**Selalu kirim foto/preview hasil desainnya langsung, bukan cuma link.** Setelah desain jadi (baik create maupun edit), lakukan ini sebelum melapor ke user:
1. Export desain ke format gambar (PNG/JPG) lewat `mcp_canva_export_design` atau tool export yang tersedia.
2. Download/ambil file hasil export tersebut.
3. Kirim file gambarnya langsung ke user di chat (pakai kemampuan kirim attachment Hermes), bukan cuma teks link.
4. Baru sertakan caption/laporan singkat menyertai foto tersebut.

Format laporan (menyertai gambar yang dikirim):
```
✅ [Apa yang dikerjakan]
- Judul desain: [nama]
- Link edit di Canva: [url Canva] (kalau user mau edit manual)
- Ukuran/format: [detail]
- Catatan: [asumsi yang diambil, kalau ada]
```
Kalau desain multi-halaman (misal presentasi), kirim preview halaman pertama sebagai gambar, dan sebutkan total jumlah halaman di caption — jangan kirim semua halaman sekaligus kecuali user minta.

Kalau gagal atau parsial, laporkan juga dengan jujur:
```
⚠️ [Apa yang gagal]
- Penyebab: [alasan singkat, misal "resize butuh Canva Pro"]
- Saran: [langkah lanjutan]
```

### 4. Belajar dari pola pemakaian
Setelah menyelesaikan tugas Canva, simpan pola yang berulang ke memory Hermes (misal lewat catatan/skill notes internal) — contoh: ukuran yang sering dipakai user, gaya visual favorit, nama folder yang sering dituju, brand kit yang biasa dipakai. Pakai pola ini di sesi berikutnya supaya makin sedikit tanya-jawab ulang, tapi tetap konfirmasi kalau ada perubahan konteks yang signifikan (klien beda, kampanye beda).

### 5. Optimalkan urutan kerja
- Kalau user minta "buat lalu export", rantai langsung dalam satu alur — jangan berhenti minta konfirmasi di tengah kalau instruksinya sudah jelas.
- Kalau butuh brand kit/template dan user Enterprise, cek brand kit dulu sebelum bikin dari nol, supaya konsisten dengan identitas visual yang sudah ada.
- Kalau ada banyak desain serupa (misal 20 sertifikat), cek dulu apakah ada fitur autofill/brand template yang bisa dipakai daripada bikin manual satu-satu.

## Batasan & Kehati-hatian
- Jangan pernah menghapus desain (`*delete*`) tanpa konfirmasi eksplisit dari user, meski tool-nya tersedia.
- Kalau operasi butuh plan Canva tertentu (Pro/Enterprise) dan gagal karena itu, jelaskan sebabnya ke user, jangan cuma bilang "error".
- Kalau MCP `canva` tidak merespons atau OAuth expired, laporkan itu ke user dengan jelas dan sarankan `hermes mcp login canva` ulang — jangan diam-diam gagal.

## Ganti Akun Canva

Kalau user minta ganti akun Canva (misal "ganti akun canva", "pindah ke akun lain", "logout canva"), **jangan coba oprek config atau tebak-tebak sendiri**. Ikuti alur sederhana ini:

1. Hapus token OAuth lama supaya sesi lama benar-benar putus:
   ```
   rm ~/.hermes/mcp-tokens/canva.json
   ```
2. Jalankan ulang proses login:
   ```
   hermes mcp login canva
   ```
3. Ambil **authorize URL** yang muncul dari output tersebut, lalu kirim URL itu ke user apa adanya — tanpa dipersingkat, tanpa dijelaskan berlebihan. Cukup format begini:
   ```
   Silakan login ke akun Canva yang kamu mau pakai lewat link ini:
   [authorize URL]

   Setelah login & approve, browser akan menuju halaman yang kelihatan error — itu normal. Copy full URL dari address bar (termasuk ?code=...&state=...) dan kirim balik ke saya.
   ```
4. Setelah user kirim balik redirect URL/code-nya, tempelkan ke prompt login yang sedang menunggu, lalu **verifikasi** login berhasil (misal cek `mcp_canva_list_designs` mengembalikan desain milik akun baru) sebelum bilang sukses ke user.
5. Laporkan hasil akhir:
   ```
   ✅ Berhasil ganti akun Canva
   - Akun aktif sekarang: [nama/email akun kalau kebaca dari tool]
   ```

**Prinsip penting:** agent tidak menebak kredensial, tidak mencoba login otomatis tanpa aksi user, dan tidak melanjutkan pekerjaan Canva apa pun sebelum proses ganti akun ini benar-benar selesai dan terverifikasi.

## Contoh Alur
**User:** "buatin ig story buat sale akhir tahun"
**Hermes:**
1. Tanya 1 hal penting kalau perlu (misal: ada preferensi warna/tema toko?)
2. Buat desain 1080x1920 dengan tema yang relevan
3. Export ke PNG, ambil file gambarnya
4. Verifikasi hasil dari MCP
5. Kirim foto desainnya langsung ke chat, dengan caption:
```
✅ Desain IG Story "Sale Akhir Tahun" selesai dibuat
- Link edit: https://canva.com/design/xxxxx
- Ukuran: 1080x1920 (IG Story)
- Catatan: saya pakai tema merah-emas karena konteks "akhir tahun/sale", ganti kalau mau warna lain
```
