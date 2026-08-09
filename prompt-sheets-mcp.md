Saya mau setup MCP server custom untuk Google Sheets — bisa baca data, tulis/update data, dan bikin laporan/rekap otomatis ke sheet. JANGAN bilang selesai sebelum benar-benar diuji dengan bukti nyata.

KONTEKS API:
- Google Sheets API v4, base: https://sheets.googleapis.com/v4/spreadsheets
- Auth: OAuth 2.0 (bukan API key) — karena butuh akses baca/tulis ke spreadsheet pribadi/tim, bukan data publik
- Scope yang dibutuhkan:
  - https://www.googleapis.com/auth/spreadsheets (baca + tulis)
  - https://www.googleapis.com/auth/drive.file (kalau perlu bikin sheet baru dari scratch)
- Setup yang dibutuhkan sebelum ngoding:
  1. Buat/pakai project di Google Cloud Console (bisa reuse project yang sama dengan YouTube kalau sudah ada)
  2. Enable "Google Sheets API" (dan "Google Drive API" kalau butuh bikin file baru)
  3. Buat OAuth 2.0 Client ID tipe "Desktop app"
  4. Download client_secret.json

TAHAP 1 — Buat MCP server sederhana (Python, pakai library mcp resmi + google-api-python-client + google-auth-oauthlib):
1. Buat folder ~/.hermes/mcp-servers/sheets-mcp
2. Buat server MCP dengan tools:
   - read_sheet(spreadsheet_id, range) — baca data dari range tertentu (misal "Sheet1!A1:E50"), return sebagai list baris
   - write_sheet(spreadsheet_id, range, values) — tulis/update data ke range tertentu, values berupa list-of-list (baris x kolom)
   - append_row(spreadsheet_id, sheet_name, values) — tambah baris baru di akhir data yang sudah ada, tanpa menimpa data lain
   - create_report_sheet(spreadsheet_id, new_sheet_name, headers, rows) — bikin sheet/tab baru di dalam spreadsheet yang sama, isi header + data laporan
   - list_sheets(spreadsheet_id) — list semua tab/sheet dalam satu spreadsheet, buat cek nama sheet yang benar sebelum baca/tulis
2. Kredensial (client_id, client_secret, refresh_token) disimpan sebagai environment variable / file token terenkripsi, JANGAN hardcode
3. SAFETY: tool write_sheet dan append_row WAJIB validasi dulu spreadsheet_id & range yang dikasih itu valid (panggil list_sheets dulu buat cek nama sheet ada), supaya nggak salah tulis ke sheet yang salah
4. Tunggu saya kasih client_secret.json dan spreadsheet_id yang mau dipakai, sebelum lanjut testing

TAHAP 2 — Setelah saya kasih client_secret.json + spreadsheet_id:
1. Jalankan OAuth consent flow, kasih saya link buat login & approve
2. Simpan refresh_token hasil consent ke .env / file token
3. Tambahkan konfigurasi ke ~/.hermes/config.yaml bagian mcp_servers dengan nama "sheets" — jangan hapus/ubah entry MCP lain yang sudah ada
4. Reload MCP

TAHAP 3 — Verifikasi wajib:
1. Panggil list_sheets pada spreadsheet_id yang saya kasih, tunjukkan hasil ASLI (nama-nama tab yang ada) sebagai bukti auth & koneksi berhasil
2. Panggil read_sheet pada salah satu range yang ada isinya, tunjukkan data ASLI yang kebaca (minimal beberapa baris)
3. Test append_row dengan 1 baris data test (tandai jelas ini data test, misal isi kolom pertama "TEST - boleh dihapus"), tunjukkan hasilnya dan minta saya cek manual bahwa barisnya masuk dengan benar
4. Kalau error, diagnosis dulu (baca pesan error lengkap dari Google — 403 biasanya scope kurang / sheet nggak di-share ke akun yang dipakai OAuth, 404 biasanya spreadsheet_id salah) sebelum ubah kode — jangan asal coba-coba
5. Laporkan hasil akhir: tool mana yang berhasil disetup + bukti nyata dari tiap tool yang berhasil dites, atau kalau gagal jelaskan di step mana dan kenapa

Mulai dari TAHAP 1, tunggu saya kasih client_secret.json dan spreadsheet_id sebelum lanjut ke TAHAP 2.
