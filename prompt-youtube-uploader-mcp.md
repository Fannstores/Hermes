Saya mau setup MCP server "youtube-uploader-mcp" dari GitHub (anwerj/youtube-uploader-mcp) untuk upload video otomatis ke YouTube, terintegrasi dengan Canva MCP yang sudah terhubung. JANGAN bilang selesai sebelum benar-benar diuji dengan bukti nyata.

REPO: https://github.com/anwerj/youtube-uploader-mcp
- Install via script resmi di repo (cek README untuk command install terbaru sesuai OS)
- Auth: OAuth 2.0, butuh client_secret.json dari Google Cloud Console (project + enable YouTube Data API v3 + buat OAuth Client ID)
- Tool utama: authenticate (generate URL OAuth), upload video, update video, schedule video

TAHAP 1 — Install & setup:
1. Install youtube-uploader-mcp sesuai instruksi resmi di README repo
2. Buat folder config di ~/.hermes/mcp-servers/youtube-uploader-mcp (atau lokasi yang direkomendasikan repo)
3. Tunggu saya kasih client_secret.json sebelum lanjut authenticate

TAHAP 2 — Setelah saya kasih client_secret.json:
1. Jalankan tool "authenticate", kasih saya link OAuth buat saya approve manual
2. Tambahkan konfigurasi ke ~/.hermes/config.yaml bagian mcp_servers dengan nama "youtube-uploader" — jangan hapus/ubah entry MCP lain yang sudah ada (termasuk Canva)
3. Reload MCP

TAHAP 3 — Alur kerja video (WAJIB tanya saya dulu tiap kali sebelum generate/upload):
1. Sebelum bikin video baru, tanya saya dulu 2 hal: (a) TARGET/topik video, (b) BAHASA yang dipakai (misal: Indonesia, English, dll)
2. Generate video pakai Canva MCP sesuai target & bahasa yang saya tentukan
3. Tambahkan watermark/logo ke video lewat Canva MCP sebelum export final (pakai file logo yang saya sediakan — tanya saya dulu kalau belum ada logo tersimpan, jangan pakai logo random/placeholder)
4. Generate metadata YouTube yang dioptimasi buat SEO & audiens target:
   - title: singkat, jelas, mengandung keyword utama sesuai TARGET yang saya kasih
   - description: 2-4 paragraf, mengandung keyword relevan di awal, ada call-to-action, sesuai BAHASA yang saya tentukan
   - tags: 5-10 tag relevan
   Tunjukkan draft title/description/tags ini ke saya dulu, saya boleh minta revisi sebelum lanjut
5. Kirim saya preview/link hasil Canva-nya (video + watermark) dan draft metadata YouTube-nya dulu untuk saya approve — JANGAN langsung upload ke YouTube tanpa persetujuan saya atas KEDUANYA (video final & metadata)
6. Setelah saya approve, baru upload ke YouTube dengan title/description/tags yang sudah disetujui, privacy_status="private" (default, kecuali saya bilang publish public)

TAHAP 4 — Verifikasi wajib:
1. Test authenticate dulu, tunjukkan link OAuth-nya berhasil di-generate
2. Setelah saya approve OAuth, test upload SATU video kecil dengan privacy_status="private", tunjukkan response ASLI dari API (video_id, url) sebagai bukti berhasil
3. Cek video itu beneran muncul di YouTube Studio channel saya (private) — saya yang konfirmasi manual
4. Kalau error, diagnosis dulu (baca pesan error lengkap dari Google — 401 = token invalid/expired, 403 = quota habis atau scope kurang, dll) sebelum ubah kode — jangan asal coba-coba
5. Laporkan hasil akhir: apa yang berhasil disetup + bukti video test yang berhasil diupload beserta draft metadata yang dipakai, atau kalau gagal jelaskan di step mana dan kenapa

Mulai dari TAHAP 1, tunggu saya kasih client_secret.json sebelum lanjut ke TAHAP 2.
