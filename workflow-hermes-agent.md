# Workflow Hermes Agent: Task → Plan → Code → Deploy → Test

## Alur Umum

```
User kasih instruksi
        ↓
[1] Hermes: Pahami & Susun Logika
        ↓
[2] Hermes: Konfirmasi ke User
        ↓
   User approve? ──No──→ Revisi logika, ulang ke [2]
        │
       Yes
        ↓
[3] Jalankan OpenCode (Planning Task)
        ↓
[4] OpenCode: Coding / Implementasi
        ↓
[5] Hermes: Deploy ke cPanel
        ↓
[6] Hermes: Testing Otomatis
        ↓
   Ada project lama di cPanel? ──Yes──→ Tanya: Hapus & replace / Buat baru?
        │                                        │
        No                                    User pilih
        │                                        │
        └──────────────→ [7] Deploy Final ←──────┘
                                ↓
                     [8] Laporan Hasil ke User
```

---

## Detail Tiap Tahap

### 1. Pahami & Susun Logika
Hermes membaca instruksi user, memecahnya jadi:
- **Goal**: apa yang mau dicapai
- **Scope**: fitur/komponen apa saja yang terlibat
- **Constraint**: batasan (misal: harus pakai stack tertentu, gak boleh ubah file X)
- **Output logika sederhana**: ringkasan step-by-step sebelum eksekusi

### 2. Konfirmasi ke User
Sebelum eksekusi, Hermes menampilkan ringkasan rencana:
```
Rencana saya:
1. ...
2. ...
3. ...

Lanjut eksekusi? (ya/tidak/revisi)
```
Ini penting supaya user bisa koreksi logika sebelum resource (waktu, token, compute) kepakai buat hal yang salah arah.

### 3. Jalankan OpenCode — Planning
Setelah user konfirmasi, Hermes trigger OpenCode dalam mode **planning**:
- OpenCode memecah task jadi sub-task teknis (file yang perlu dibuat/diubah, dependency, urutan kerja)
- Output: task list/plan teknis yang lebih granular dari langkah 1

### 4. OpenCode — Coding
OpenCode eksekusi coding berdasarkan plan tadi:
- Tulis/edit file
- Jalankan test lokal (unit test bila ada)
- Commit perubahan (kalau pakai git)

### 5. Deploy ke cPanel
Hermes ambil hasil coding, deploy ke cPanel via salah satu jalur (pilih sesuai kapasitas hosting):
- **Git deployment** (kalau cPanel support Git Version Control)
- **FTP/SFTP upload** otomatis
- **cPanel API** (UAPI/WHM API) kalau butuh kontrol lebih (buat/hapus domain, database, dll)

### 6. Testing Otomatis
Setelah deploy, Hermes jalankan pengecekan dasar:
- Hit endpoint/URL project → cek response code (200 dsb)
- Cek log error cPanel kalau ada
- (Opsional) smoke test fungsi utama

### 7. Cek Project Existing → Tanya User
Kalau nama project yang mau di-deploy **sudah ada** di cPanel:
```
Project "nama-project" sudah ada di cPanel.
Pilih:
[a] Hapus yang lama, replace dengan versi baru
[b] Buat sebagai project baru (nama/subdomain berbeda)
```
Baru lanjut deploy final sesuai pilihan user.

### 8. Laporan Hasil
Hermes kasih ringkasan akhir: apa yang di-deploy, di URL mana, hasil testing gimana, dan next step kalau ada error.

---

## Catatan Implementasi

- **Konfirmasi di step 2 wajib** — jangan skip walau instruksi user kelihatan jelas, karena mencegah salah eksekusi yang mahal untuk di-revert.
- **Deploy ke cPanel** butuh kredensial disimpan aman (bukan hardcode di script) — idealnya di environment variable/config file Hermes yang gak ke-commit ke repo.
- Kalau cPanel gak expose Git/API, opsi paling stabil adalah SFTP + script deploy manual yang dipicu Hermes.
- Testing step (6) sebaiknya punya *rollback plan*: kalau testing gagal, siapkan opsi revert ke versi sebelumnya sebelum lapor ke user.
