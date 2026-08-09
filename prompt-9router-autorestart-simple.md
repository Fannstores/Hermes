Setup auto-restart untuk "9router" (sudah terinstall global via npm) pakai systemd, supaya otomatis nyala lagi kalau mati/crash, dan auto-start saat server reboot.

1. Cari path binary 9router: `which 9router`
2. Buat file /etc/systemd/system/9router.service:

[Unit]
Description=9Router
After=network.target

[Service]
ExecStart=<PATH_HASIL_STEP_1>
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target

3. Ganti <PATH_HASIL_STEP_1> dengan path asli dari step 1
4. Jalankan:
   sudo systemctl daemon-reload
   sudo systemctl enable 9router
   sudo systemctl start 9router
5. Verifikasi: `sudo systemctl status 9router` harus "active (running)"
6. Test auto-restart beneran jalan: cari PID (`pgrep -f 9router`), kill paksa (`kill -9 <PID>`), tunggu 5 detik, cek status lagi — harus balik "active (running)" otomatis dengan PID baru
7. Laporkan hasil: isi service file final + bukti status setelah test kill di atas
