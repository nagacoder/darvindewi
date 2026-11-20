# darvindewi.com: undangan pernikahan Darvin & Dewi

Static site, tanpa build step, tanpa framework. Cukup upload folder ini ke hosting mana pun.

```
index.html       undangan (semua isi diatur dari CONFIG di bagian bawah file)
link-tamu.html   generator link per tamu + pesan WhatsApp (tidak terindeks Google)
og-image.jpg     gambar preview saat link dibagikan di WhatsApp (1200×630)
fonts/           Bodoni Moda + Hanken Grotesk (self-hosted, lisensi OFL)
```

## 1. Edit isi

Buka `index.html`, cari `const CONFIG`. Semua teks di dalam `[kurung siku]` wajib diganti:

- Nama orang tua kedua mempelai
- Tanggal & jam acara (`start`, `end`), format ISO dengan offset zona waktu
  - Jakarta: `+07:00`, `timeZone: 'Asia/Jakarta'`, `tzLabel: 'WIB'`
  - Singapura: `+08:00`, `timeZone: 'Asia/Singapore'`, `tzLabel: 'SGT'`
- Nama acara (`Upacara pernikahan` bisa diganti `Akad nikah`, `Pemberkatan nikah`, dll.)
- Nama tempat, alamat, link Google Maps (tombol Share di Google Maps)
- Nomor WhatsApp penerima RSVP (format `62812...` / `6590...`, tanpa `+`)

Bagian lain di `CONFIG`:

- `heroPhoto`: foto berdua untuk halaman depan, **PNG dengan background transparan** (seperti referensi deliciasonny). Di iPhone/Mac cukup tekan-tahan subjek foto → *Copy Subject* / *Lift Subject*, atau pakai remove.bg. Simpan ±1000px tingginya, <500 KB. Kosong = hero memakai logo cincin.
- `zones`: jam acara tampil dalam dua zona (WIB + SGT). Zona pertama = lokasi acara.
- `story`: timeline "Cerita kami". Dua langkah awal sudah diisi dari CV kalian (Darvin 2017, Dewi 2016), lalu jalurnya menyatu. Item `together` bisa ditambah/dikurangi bebas; sisi kiri/kanan diatur otomatis. Ikon: `code`, `survey`, `meet`, `heart`, `gem`, `plane`, `book`, `home`, `ring`.
- `gifts`: rekening / e-wallet. Masih placeholder; ganti, atau jadikan `[]` untuk menyembunyikan tombol "Tanda kasih".
- `credit`: kredit di footer (`© 2027 oleh nagacoder`). Kosongkan `name` kalau tidak mau.
- `photos`: galeri opsional, taruh di folder `photos/` (lebar ±1200px, JPG kualitas 80, <300 KB per foto). Kosong = tidak tampil.
- `music`: file mp3 (pakai lagu yang memang boleh dipakai, mis. dari YouTube Audio Library). Kosong = tombol musik hilang.

**Jangan lupa edit `<meta property="og:...">` di `<head>`.** WhatsApp tidak menjalankan JavaScript, jadi tanggal & URL di preview link diambil dari tag ini. Kalau tanggal berubah, render ulang `og-image.jpg` juga (atau minta saya).

Warna ada di 5 variabel CSS paling atas (`--paper`, `--ink`, `--ink-soft`, `--liner`, `--on-liner`).

## 2. Nama tamu di halaman pembuka

Tambahkan `?to=` di URL:

```
https://darvindewi.com/?to=Bapak+Hendra+Simanjuntak+%26+Keluarga
```

Untuk ratusan tamu, buka `https://darvindewi.com/link-tamu.html`, paste daftar nama (satu per baris), lalu klik **Buat link**. Setiap baris punya tombol *Salin pesan*, *Kirim lewat WhatsApp*, dan ada *Unduh CSV* untuk semua link sekaligus.

## 3. RSVP

Default: tamu isi form → WhatsApp terbuka dengan pesan siap kirim ke nomor kalian. Nol backend.

Kalau mau semua RSVP masuk ke Google Sheet:

1. Buat Google Sheet baru → **Extensions → Apps Script**, paste:

   ```js
   function doPost(e) {
     const ss = SpreadsheetApp.getActiveSpreadsheet();
     const sh = ss.getSheetByName('RSVP') || ss.insertSheet('RSVP');
     if (sh.getLastRow() === 0) sh.appendRow(['Waktu', 'Nama', 'Hadir', 'Jumlah', 'Ucapan', 'Diundang sebagai']);
     const d = JSON.parse(e.postData.contents);
     sh.appendRow([new Date(), d.name, d.attend ? 'Ya' : 'Tidak', d.guests, d.wish, d.invitedAs]);
     return ContentService.createTextOutput('ok');
   }
   ```

2. **Deploy → New deployment → Web app**, *Execute as: Me*, *Who has access: Anyone*.
3. Copy URL web app ke `CONFIG.rsvp.endpoint`. Tombol otomatis berubah jadi "Kirim konfirmasi".

## 4. Deploy

### Opsi A: Cloudflare (paling simpel, gratis)

1. Beli `darvindewi.com` di Cloudflare Registrar (harga at-cost, DNS langsung di Cloudflare).
2. Dashboard → **Workers & Pages → Create → Pages → Upload assets**, drag folder ini.
   Atau push ke GitHub lalu connect repo, supaya setiap `git push` otomatis deploy.
3. Di project → **Custom domains** → tambahkan `darvindewi.com` dan `www.darvindewi.com`.

### Opsi B: VPS / server sendiri (Caddy, HTTPS otomatis)

Arahkan A record `darvindewi.com` ke IP server, lalu:

```bash
docker run -d --name undangan --restart unless-stopped \
  -p 80:80 -p 443:443 \
  -v "$PWD":/srv:ro -v caddy_data:/data \
  caddy caddy file-server --root /srv --domain darvindewi.com
```

Kalau dari server rumah tanpa IP publik: jalankan `caddy file-server --listen :8080` lalu expose lewat Cloudflare Tunnel.

## 5. Sebelum disebar

- [ ] Semua `[kurung siku]` sudah diganti
- [ ] Tes buka di iPhone & Android, termasuk dari dalam WhatsApp (in-app browser)
- [ ] Tes RSVP sampai pesan/Sheet benar-benar masuk
- [ ] Kirim link ke diri sendiri dulu, cek preview gambar & judulnya. WhatsApp meng-cache preview; kalau salah, tambahkan `&v=2` di link untuk memaksa refresh
- [ ] Tes tombol *Buka peta* & *Simpan ke kalender*
- [ ] Nyalakan auto-renew domain

## Domain

Pilihan utama **darvindewi.com**, plus **dewidarvin.com** di-redirect ke sana (kalau ada yang salah ketik urutan). Cadangan: `darvinwedsdewi.com`, `darvindewi.id` (butuh KTP). Ketersediaan belum dicek, jadi cek dulu di registrar.
