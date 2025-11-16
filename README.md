# recon-automation-alief
# 🚀 Recon Automation Lief

Sebuah skrip Bash sederhana untuk mengotomatiskan alur kerja enumerasi subdomain dasar. Proyek ini dibuat untuk mempelajari otomatisasi *recon* untuk *bug bounty* dan *penetration testing*.

Skrip ini mengambil daftar domain, menjalankan `subfinder` untuk menemukan subdomain, menggunakan `anew` untuk menyimpan hasil unik, dan terakhir menggunakan `httpx` untuk memfilter host yang *live* (hidup).

```bash

git clone https://github.com/aliefnugroho18/recon-automation-alief.git

cd recon-automation-alief

# Instal Subfinder
go install -v [github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest](https://github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest)

# Instal HTTPX
go install -v [github.com/projectdiscovery/httpx/cmd/httpx@latest](https://github.com/projectdiscovery/httpx/cmd/httpx@latest)

# Instal Anew
go install -v [github.com/tomnomnom/anew@latest](https://github.com/tomnomnom/anew@latest)

# Pindahkan semua tools ke /usr/local/bin agar dapat ditemukan
sudo mv ~/go/bin/subfinder /usr/local/bin/
sudo mv ~/go/bin/httpx /usr/local/bin/
sudo mv ~/go/bin/anew /usr/local/bin/

chmod +x scripts/recon-auto.sh

./scripts/recon-auto.sh

```

🚀 Penjelasan Skrip (scripts/recon-auto.sh)
Skrip ini dirancang untuk portabel dan aman. Berikut adalah cara kerjanya:

Konfigurasi Path (Baris 13-16)

SCRIPT_DIR & PROJECT_ROOT: Bagian ini secara otomatis mendeteksi di mana skrip berada. Ini memungkinkan Anda menjalankan skrip dari direktori mana pun tanpa merusak path ke file input atau output.

Fungsi log() (Baris 29-34)

Ini adalah fungsi logging kustom. Setiap pesan yang dikirim ke log "pesan" akan dicetak ke layar DAN ditambahkan ke logs/progress.log menggunakan tee -a, lengkap dengan timestamp.

Pengecekan Dependensi (Baris 40-52)

Skrip berhenti jika tidak dapat menemukan subfinder, anew, atau httpx di PATH sistem. Ini mencegah skrip berjalan setengah jalan dan gagal.

Enumerasi (Loop) (Baris 70-87)

Skrip membaca file input/domains.txt baris demi baris menggunakan while read domain.

Ini memungkinkan skrip untuk memproses satu domain pada satu waktu.

Alur (Pipeline) Inti (Baris 80)

subfinder -d "$domain" -silent | anew "$OUTPUT_SUBDOMAINS"

Ini adalah "jantung" dari skrip. Output (subdomain) dari subfinder dialirkan (|) langsung ke anew.

anew kemudian memeriksa apakah subdomain tersebut sudah ada di output/all-subdomains.txt. Jika belum, ia menambahkannya ke file. Ini sangat efisien untuk deduplikasi.

Penanganan Error (Baris 80)

2>> "$ERROR_LOG": Ini adalah bagian penting. Angka 2 mewakili Standard Error (stderr). >> berarti menambahkan ke file.

Setiap pesan error dari subfinder atau anew (misalnya, "API key not configured") tidak akan mengotori layar, tetapi akan dialihkan dengan rapi ke logs/errors.log untuk ditinjau nanti.

Pengecekan Live Host (Baris 96-103)

httpx -l "$OUTPUT_SUBDOMAINS" ... -o "$OUTPUT_LIVE"

Setelah semua subdomain terkumpul, httpx membaca (-l) file all-subdomains.txt dan memeriksa mana yang hidup, lalu menyimpan (-o) hasilnya ke live.txt.

Laporan Akhir (Baris 110-120)

wc -l < "$FILE": Ini adalah cara yang efisien untuk menghitung jumlah baris (-l) dalam sebuah file.

Skrip mengakhiri dengan memberi Anda ringkasan tentang berapa banyak subdomain total dan host live yang ditemukannya.

🖼️ Screenshot

Eksekusi Terminal
Menampilkan skrip saat sedang berjalan dan laporan akhir.

<img width="916" height="245" alt="image" src="https://github.com/user-attachments/assets/9355b248-a947-464a-a93b-85d21cb54f70" />
<img width="1002" height="182" alt="image" src="https://github.com/user-attachments/assets/9b3cedf2-ab27-4fc6-99e0-84d024fc8557" />

Hasil live.txt
Menampilkan isi dari file output/live.txt setelah skrip selesai.

<img width="421" height="264" alt="image" src="https://github.com/user-attachments/assets/2c06d2f4-1401-4ee4-ba72-69c101ecfbcf" />


