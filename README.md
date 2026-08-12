# Panduan Lengkap Penggunaan FFmpeg

## Daftar Isi
1. [Apa itu FFmpeg](#1-apa-itu-ffmpeg)
2. [Instalasi](#2-instalasi)
3. [Konsep Dasar & Struktur Perintah](#3-konsep-dasar--struktur-perintah)
4. [Melihat Informasi File Media](#4-melihat-informasi-file-media)
5. [Konversi Format Video & Audio](#5-konversi-format-video--audio)
6. [Kompresi & Kontrol Kualitas (Codec, Bitrate, CRF)](#6-kompresi--kontrol-kualitas-codec-bitrate-crf)
7. [Memotong (Trim) & Menggabungkan (Concat) Video](#7-memotong-trim--menggabungkan-concat-video)
8. [Mengubah Resolusi, Rasio, dan Rotasi](#8-mengubah-resolusi-rasio-dan-rotasi)
9. [Mengelola Audio](#9-mengelola-audio)
10. [Ekstrak Gambar/Thumbnail dari Video](#10-ekstrak-gambarthumbnail-dari-video)
11. [Ekstrak subtitle dari video](#11-ekstrak-subtitle-dari-video)
12. [Membuat Video dari Kumpulan Gambar](#12-membuat-video-dari-kumpulan-gambar)
13. [Menambahkan Watermark, Teks, dan Subtitle](#13-menambahkan-watermark-teks-dan-subtitle)
14. [Filter Video Populer](#14-filter-video-populer)
15. [Streaming & Merekam](#15-streaming--merekam)
16. [Optimasi Video untuk YouTube](#16-optimasi-video-untuk-youtube)
17. [Tips Mempercepat Proses (Hardware Acceleration)](#17-tips-mempercepat-proses-hardware-acceleration)
18. [Troubleshooting Umum](#18-troubleshooting-umum)
19. [Referensi Cepat (Cheat Sheet)](#19-referensi-cepat-cheat-sheet)

---

## 1. Apa itu FFmpeg

FFmpeg adalah software command-line gratis dan open-source untuk memproses video, audio, dan media lainnya. Dengan FFmpeg, kamu bisa: mengonversi format, memotong/menggabungkan video, mengompres ukuran file, menambahkan watermark/subtitle, ekstrak audio, membuat GIF, streaming, dan banyak lagi — semuanya lewat perintah teks tanpa software editing berat.

Cocok digunakan untuk konten kreator (termasuk keperluan render/export video YouTube), developer, maupun kebutuhan otomatisasi pengolahan video dalam jumlah banyak (batch processing).

---

## 2. Instalasi

### Windows
1. Download build dari situs resmi: https://ffmpeg.org/download.html (pilih build dari gyan.dev atau BtbN)
2. Ekstrak file zip, misalnya ke `C:\ffmpeg`
3. Tambahkan `C:\ffmpeg\bin` ke Environment Variables > Path
4. Cek instalasi lewat Command Prompt:
```bash
ffmpeg -version
```

### macOS (via Homebrew)
```bash
brew install ffmpeg
```

### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install ffmpeg
```

Cek versi setelah instalasi:
```bash
ffmpeg -version
```

---

## 3. Konsep Dasar & Struktur Perintah

Struktur umum perintah FFmpeg:

```bash
ffmpeg [opsi_input] -i input.mp4 [opsi_output] output.mp4
```

Contoh sederhana — konversi file MOV ke MP4:
```bash
ffmpeg -i video.mov video.mp4
```

Beberapa flag penting yang sering dipakai:

| Flag | Fungsi |
|---|---|
| `-i` | Menentukan file input |
| `-c:v` | Codec video (video codec) |
| `-c:a` | Codec audio |
| `-b:v` | Bitrate video |
| `-b:a` | Bitrate audio |
| `-r` | Frame rate (fps) |
| `-s` | Resolusi (ukuran frame) |
| `-t` | Durasi output |
| `-ss` | Waktu mulai (seek) |
| `-y` | Timpa file output tanpa konfirmasi |
| `-vn` | Hilangkan (nonaktifkan) video |
| `-an` | Hilangkan (nonaktifkan) audio |

---

## 4. Melihat Informasi File Media

Gunakan `ffprobe` (satu paket dengan FFmpeg) untuk melihat detail file:
```bash
ffprobe video.mp4
```

Untuk output ringkas format tertentu:
```bash
ffprobe -v error -show_format -show_streams video.mp4
```

---

## 5. Konversi Format Video & Audio

**Video ke format lain:**
```bash
ffmpeg -i input.avi output.mp4
```

**Video ke audio (ekstrak audio saja):**
```bash
ffmpeg -i video.mp4 -vn -acodec copy audio.aac
```

**Audio ke format lain (misal WAV ke MP3):**
```bash
ffmpeg -i audio.wav -codec:a libmp3lame -qscale:a 2 audio.mp3
```

**Konversi cepat tanpa re-encode (copy stream, lebih cepat, kualitas sama persis):**
```bash
ffmpeg -i input.mkv -c copy output.mp4
```
> Catatan: cara ini hanya bisa dipakai jika container tujuan mendukung codec yang sama dari file asal.

---

## 6. Kompresi & Kontrol Kualitas (Codec, Bitrate, CRF)

**Kompres video dengan codec H.264 (paling umum & kompatibel):**
```bash
ffmpeg -i input.mp4 -c:v libx264 -crf 23 -preset medium -c:a aac -b:a 128k output.mp4
```

Penjelasan parameter kunci:
- **CRF (Constant Rate Factor)**: mengatur kualitas vs ukuran file. Skala 0–51 (0 = lossless, 51 = kualitas terburuk). Rentang ideal: **18–28**. Semakin kecil, semakin bagus kualitas tapi ukuran makin besar.
- **preset**: mengatur kecepatan encoding vs efisiensi kompresi. Urutan dari cepat ke lambat: `ultrafast, superfast, veryfast, faster, fast, medium, slow, slower, veryslow`. Preset lebih lambat = ukuran file lebih kecil untuk kualitas yang sama, tapi proses lebih lama.

**Kompres dengan codec H.265/HEVC (ukuran lebih kecil, tapi kompatibilitas lebih terbatas):**
```bash
ffmpeg -i input.mp4 -c:v libx265 -crf 28 -preset medium -c:a aac output.mp4
```

**Kompres berdasarkan target bitrate tertentu (misal untuk target ukuran file):**
```bash
ffmpeg -i input.mp4 -c:v libx264 -b:v 2000k -maxrate 2000k -bufsize 4000k -c:a aac -b:a 128k output.mp4
```

---

## 7. Memotong (Trim) & Menggabungkan (Concat) Video

**Memotong video (trim) tanpa re-encode — cepat, tapi titik potong mengikuti keyframe terdekat:**
```bash
ffmpeg -i input.mp4 -ss 00:00:10 -to 00:00:40 -c copy output.mp4
```

**Memotong dengan re-encode — lebih presisi ke detik/frame yang diinginkan:**
```bash
ffmpeg -i input.mp4 -ss 00:00:10 -to 00:00:40 -c:v libx264 -c:a aac output.mp4
```

**Menggabungkan beberapa video dengan format/codec sama:**

Buat file `list.txt`:
```
file 'video1.mp4'
file 'video2.mp4'
file 'video3.mp4'
```

Jalankan:
```bash
ffmpeg -f concat -safe 0 -i list.txt -c copy output.mp4
```

**Menggabungkan video dengan format berbeda (re-encode, lebih lambat tapi aman):**
```bash
ffmpeg -i video1.mp4 -i video2.mp4 -filter_complex "[0:v][0:a][1:v][1:a]concat=n=2:v=1:a=1[v][a]" -map "[v]" -map "[a]" output.mp4
```

---

## 8. Mengubah Resolusi, Rasio, dan Rotasi

**Ubah resolusi (misal ke 1080p):**
```bash
ffmpeg -i input.mp4 -vf scale=1920:1080 output.mp4
```

**Ubah resolusi menjaga rasio aspek (lebar otomatis menyesuaikan):**
```bash
ffmpeg -i input.mp4 -vf scale=-2:720 output.mp4
```

**Crop video (memotong bagian frame):**
```bash
ffmpeg -i input.mp4 -vf "crop=1080:1080:420:0" output.mp4
```
Format: `crop=lebar:tinggi:offset_x:offset_y`

**Rotasi video 90 derajat searah jarum jam:**
```bash
ffmpeg -i input.mp4 -vf "transpose=1" output.mp4
```
Nilai transpose: `0` = 90° berlawanan jarum jam + flip vertikal, `1` = 90° searah jarum jam, `2` = 90° berlawanan jarum jam, `3` = 90° searah jarum jam + flip vertikal.

**Menambahkan pillarbox/letterbox (misal ubah ke rasio 16:9 tanpa crop):**
```bash
ffmpeg -i input.mp4 -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" output.mp4
```

---

## 9. Mengelola Audio

**Hapus audio dari video:**
```bash
ffmpeg -i input.mp4 -c copy -an output_no_audio.mp4
```

**Ganti/tambah audio baru pada video (menimpa audio lama):**
```bash
ffmpeg -i video.mp4 -i audio_baru.mp3 -c:v copy -map 0:v:0 -map 1:a:0 -shortest output.mp4
```

**Menaikkan/menurunkan volume audio:**
```bash
ffmpeg -i input.mp4 -filter:a "volume=1.5" output.mp4
```
(1.5 = 150% dari volume asli; 0.5 = 50%)

**Normalisasi volume audio (loudness normalization, penting untuk konsistensi audio YouTube):**
```bash
ffmpeg -i input.mp4 -af loudnorm=I=-16:TP=-1.5:LRA=11 output.mp4
```

**Menggabungkan/mixing dua sumber audio (misal voice over + musik latar):**
```bash
ffmpeg -i voice.mp3 -i music.mp3 -filter_complex "[0:a][1:a]amix=inputs=2:duration=longest" output.mp3
```

---

## 10. Ekstrak Gambar/Thumbnail dari Video

**Ambil satu frame di detik tertentu sebagai thumbnail:**
```bash
ffmpeg -i input.mp4 -ss 00:00:05 -vframes 1 thumbnail.jpg
```

**Ekstrak banyak frame (misal 1 gambar tiap detik):**
```bash
ffmpeg -i input.mp4 -vf fps=1 frame_%04d.jpg
```

---

## 11. Ekstrak subtitle dari video (kalau video punya subtitle track)

**Cek dulu apakah videonya punya subtitle:**

```bash
ffmpeg -i input.mp4
```

Lihat bagian Stream — kalau ada baris seperti Stream #0:2(eng): Subtitle: subrip atau mov_text, berarti ada subtitle track.

**Ekstrak subtitle ke file `.srt`:**

```bash
ffmpeg -i input.mp4 -map 0:s:0 subtitle.srt
```

`0:s:0` artinya subtitle track pertama (index 0) dari file input.

**Kalau videonya punya lebih dari satu subtitle (misal beberapa bahasa):**

```bash
ffmpeg -i input.mp4
```
**Cek nomor stream-nya dulu (misal 0:s:0 untuk bahasa Inggris, 0:s:1 untuk bahasa Indonesia), lalu:**

```bash
ffmpeg -i input.mp4 -map 0:s:1 subtitle_id.srt
```
**Kalau formatnya bitmap-based subtitle (misal dari file MKV, format PGS/VobSub), tidak bisa langsung diubah ke .srt karena bukan berbasis teks — harus di-OCR dulu pakai tool lain seperti SubtitleEdit. Tapi bisa diekstrak dulu ke format aslinya:**

```bash
ffmpeg -i input.mkv -map 0:s:0 subtitle.sup
```
Kalau videonya tidak punya subtitle track sama sekali, FFmpeg akan menampilkan error seperti Stream map '0:s:0' matches no streams — berarti subtitle-nya (kalau ada teks di video) sudah hardcode/menempel di gambar, jadi tidak bisa diekstrak sebagai teks.

## 12. Membuat Video dari Kumpulan Gambar

**Slideshow dari sekumpulan gambar (misal frame_001.jpg, frame_002.jpg, dst):**
```bash
ffmpeg -framerate 1 -i frame_%03d.jpg -c:v libx264 -r 30 -pix_fmt yuv420p output.mp4
```
`-framerate 1` berarti tiap gambar tampil selama 1 detik.

**Membuat GIF dari video:**
```bash
ffmpeg -i input.mp4 -vf "fps=10,scale=480:-1:flags=lanczos" output.gif
```

---

## 13. Menambahkan Watermark, Teks, dan Subtitle

**Menambahkan watermark gambar/logo (posisi pojok kanan bawah):**
```bash
ffmpeg -i input.mp4 -i logo.png -filter_complex "overlay=W-w-20:H-h-20" output.mp4
```

**Menambahkan teks langsung ke video (butuh font file, misal .ttf):**
```bash
ffmpeg -i input.mp4 -vf "drawtext=fontfile=/path/font.ttf:text='Teks Contoh':x=(w-text_w)/2:y=h-th-30:fontsize=36:fontcolor=white" output.mp4
```

**Menyisipkan subtitle hardcode (subtitle "menempel" langsung di video, dari file .srt):**
```bash
ffmpeg -i input.mp4 -vf "subtitles=subtitle.srt" output.mp4
```

**Menambahkan subtitle sebagai track terpisah (soft subtitle, bisa dimatikan/nyalakan penonton):**
```bash
ffmpeg -i input.mp4 -i subtitle.srt -c copy -c:s mov_text output.mp4
```

---

## 14. Filter Video Populer

| Kebutuhan | Perintah filter (`-vf`) |
|---|---|
| Blur | `boxblur=5:1` |
| Sharpen | `unsharp=5:5:1.0` |
| Grayscale (hitam putih) | `hue=s=0` |
| Ubah kecerahan/kontras | `eq=brightness=0.06:contrast=1.2` |
| Percepat video (2x) | `setpts=0.5*PTS` (video), `atempo=2.0` (audio) |
| Perlambat video (0.5x) | `setpts=2.0*PTS` (video), `atempo=0.5` (audio) |
| Flip horizontal | `hflip` |
| Flip vertikal | `vflip` |
| Fade in 2 detik di awal | `fade=t=in:st=0:d=2` |
| Fade out 2 detik di akhir | `fade=t=out:st=58:d=2` |

Contoh gabungan mempercepat video 2x beserta audionya:
```bash
ffmpeg -i input.mp4 -vf "setpts=0.5*PTS" -af "atempo=2.0" output.mp4
```

---

## 15. Streaming & Merekam

**Streaming ke platform (misal RTMP, contoh generik untuk YouTube Live/sejenisnya):**
```bash
ffmpeg -re -i input.mp4 -c:v libx264 -preset veryfast -maxrate 3000k -bufsize 6000k -c:a aac -b:a 160k -f flv rtmp://server-streaming/live/STREAM_KEY
```

**Merekam layar (Linux, via X11):**
```bash
ffmpeg -f x11grab -i :0.0 -r 30 output.mp4
```

**Merekam layar (Windows, via gdigrab):**
```bash
ffmpeg -f gdigrab -i desktop -r 30 output.mp4
```

**Merekam layar (macOS, via avfoundation):**
```bash
ffmpeg -f avfoundation -i "1" output.mp4
```

---

## 16. Optimasi Video untuk YouTube

Pengaturan yang direkomendasikan YouTube untuk hasil upload optimal (H.264, kualitas tinggi, ukuran wajar):

```bash
ffmpeg -i input.mp4 \
-c:v libx264 -preset slow -crf 18 -pix_fmt yuv420p \
-c:a aac -b:a 384k \
-movflags +faststart \
output_youtube.mp4
```

Penjelasan:
- `-pix_fmt yuv420p`: memastikan kompatibilitas warna di semua pemutar/platform
- `-movflags +faststart`: memindahkan metadata ke awal file agar video bisa mulai diputar sebelum selesai terunduh sepenuhnya (penting untuk streaming/upload)
- `-crf 18`: kualitas tinggi mendekati lossless, cocok untuk video yang akan di-upload lalu di-compress ulang oleh YouTube

Untuk file lebih ringan tapi kualitas tetap layak upload:
```bash
ffmpeg -i input.mp4 -c:v libx264 -preset medium -crf 21 -pix_fmt yuv420p -c:a aac -b:a 192k -movflags +faststart output_youtube.mp4
```

---

## 17. Tips Mempercepat Proses (Hardware Acceleration)

Jika perangkat mendukung, gunakan encoder berbasis GPU agar proses jauh lebih cepat dibanding encoder software (CPU):

**NVIDIA (NVENC):**
```bash
ffmpeg -i input.mp4 -c:v h264_nvenc -preset fast -b:v 5M output.mp4
```

**Intel Quick Sync (QSV):**
```bash
ffmpeg -i input.mp4 -c:v h264_qsv -b:v 5M output.mp4
```

**AMD (AMF, Windows):**
```bash
ffmpeg -i input.mp4 -c:v h264_amf -b:v 5M output.mp4
```

**Apple Silicon (VideoToolbox, macOS):**
```bash
ffmpeg -i input.mp4 -c:v h264_videotoolbox -b:v 5M output.mp4
```

> Catatan: hasil kompresi hardware encoder umumnya sedikit lebih besar ukurannya dibanding software encoder (libx264) pada kualitas yang sama, tapi jauh lebih cepat prosesnya — cocok untuk render cepat/preview.

---

## 18. Troubleshooting Umum

| Masalah | Penyebab & Solusi |
|---|---|
| `Unknown encoder` | Codec tidak tersedia di build FFmpeg yang terpasang. Install ulang build "full"/"full-shared" (misal dari gyan.dev untuk Windows) |
| Video hasil convert tidak ada suara | Lupa menyertakan `-c:a` atau menambahkan `-an` secara tidak sengaja. Pastikan mapping audio benar dengan `-map` |
| Ukuran file tetap besar meski sudah pakai CRF tinggi | Preset terlalu cepat (`ultrafast`/`superfast`). Coba preset `slow`/`slower` untuk efisiensi kompresi lebih baik |
| Proses macet/lama di komputer lemah | Gunakan hardware acceleration (lihat bagian 16) atau turunkan resolusi dengan `-vf scale=...` |
| Video hasil trim meleset dari waktu yang diinginkan | Trim tanpa re-encode (`-c copy`) mengikuti keyframe terdekat. Gunakan re-encode untuk potongan presisi (lihat bagian 7) |
| Error `moov atom not found` | File source rusak/corrupt atau proses transfer/download terputus. Coba unduh/transfer ulang file asal |

---

## 19. Referensi Cepat (Cheat Sheet)

```bash
# Info file
ffprobe input.mp4

# Konversi format
ffmpeg -i input.avi output.mp4

# Kompres video (kualitas bagus, ukuran wajar)
ffmpeg -i input.mp4 -c:v libx264 -crf 23 -preset medium -c:a aac output.mp4

# Potong video (presisi)
ffmpeg -i input.mp4 -ss 00:00:10 -to 00:00:40 -c:v libx264 -c:a aac output.mp4

# Gabung video (codec sama)
ffmpeg -f concat -safe 0 -i list.txt -c copy output.mp4

# Resize ke 1080p
ffmpeg -i input.mp4 -vf scale=1920:1080 output.mp4

# Ekstrak audio
ffmpeg -i input.mp4 -vn -acodec copy audio.aac

# Ambil thumbnail
ffmpeg -i input.mp4 -ss 00:00:05 -vframes 1 thumb.jpg

# Buat GIF
ffmpeg -i input.mp4 -vf "fps=10,scale=480:-1:flags=lanczos" output.gif

# Tambah watermark
ffmpeg -i input.mp4 -i logo.png -filter_complex "overlay=W-w-20:H-h-20" output.mp4

# Normalisasi audio
ffmpeg -i input.mp4 -af loudnorm=I=-16:TP=-1.5:LRA=11 output.mp4

# Export siap upload YouTube
ffmpeg -i input.mp4 -c:v libx264 -preset slow -crf 18 -pix_fmt yuv420p -c:a aac -b:a 384k -movflags +faststart output_youtube.mp4
```

---

*Panduan ini mencakup penggunaan FFmpeg dari dasar hingga kebutuhan praktis seperti export video siap upload YouTube. Untuk parameter lanjutan lainnya, dokumentasi resmi bisa dicek di https://ffmpeg.org/documentation.html*
