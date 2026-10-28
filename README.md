# MT5 Monitor — PWA Dashboard

Dashboard monitoring akun MT5 (status EA, balance, equity, posisi terbuka) yang sudah
disiapkan sebagai **PWA** agar bisa **Add to Home Screen** di HP (Android & iPhone),
siap deploy ke **GitHub + Vercel**.

## Isi folder

| File | Fungsi |
|---|---|
| `index.html` | Aplikasi dashboard (versi web dari `MT5_MONITOR_HP_98172309.html`) |
| `manifest.webmanifest` | Identitas PWA: nama, warna, ikon, `display: standalone` |
| `sw.js` | Service worker — syarat instalasi + buka offline (data Firebase tidak di-cache) |
| `icon-192.png` | Ikon home screen Android |
| `icon-512.png` | Ikon toko / splash Android |
| `icon-maskable-512.png` | Ikon adaptive Android (anti terpotong di bentuk bulat/squircle) |
| `apple-touch-icon.png` | Ikon home screen iPhone/iPad (180×180) |
| `favicon-64.png` | Favicon browser |
| `vercel.json` | Header cache + MIME type (`manifest`, `sw.js` selalu revalidate) |

## Yang diubah dari file asli

1. `<base href="https://...cloudfront.net/...">` **dihapus** — kalau dibiarkan, semua aset
   di Vercel akan menunjuk ke CloudFront dan ikon/manifest gagal dimuat.
2. Dua ikon base64 raksasa di `<head>` diganti file PNG asli (HTML 62 KB → 40 KB, lebih cepat).
3. Ditambah: `manifest`, `apple-touch-icon`, meta `apple-mobile-web-app-*`,
   registrasi service worker, dan tombol **Install** di header (muncul otomatis kalau
   browser mengizinkan instalasi).
4. Layout sudah punya padding `env(safe-area-inset-*)` saat `display-mode: standalone`
   (tidak ketutup notch/home indicator).

## Cara deploy (GitHub → Vercel)

### 1. Upload ke GitHub
```bash
# dari dalam folder ini
git init
git add .
git commit -m "MT5 Monitor PWA"
git branch -M main
git remote add origin https://github.com/USERNAME/mt5-monitor.git
git push -u origin main
```
Atau lewat web: GitHub → **New repository** → **uploading an existing file** → seret semua
file di folder ini (jangan taruh di dalam subfolder, harus di root repo).

### 2. Import ke Vercel
1. Buka [vercel.com/new](https://vercel.com/new) → login dengan GitHub.
2. Pilih repo `mt5-monitor` → **Import**.
3. Framework Preset: **Other**. Build Command: kosong. Output Directory: kosong (root).
4. **Deploy** → dapat URL seperti `https://mt5-monitor.vercel.app`.

Tidak perlu setting apa pun; ini situs statis murni.

## Cara add icon ke layar HP

> Penting: buka URL Vercel lewat **Chrome (Android)** atau **Safari (iPhone)**,
> bukan dari dalam aplikasi lain (WhatsApp/Instagram), karena ikon tidak bisa dipasang
> dari in-app browser.

**Android (Chrome):**
1. Buka `https://<project>.vercel.app`
2. Muncul banner **Install** → tap, atau menu ⋮ → **Add to Home screen / Install app**
3. Ikon muncul di launcher, terbuka fullscreen tanpa address bar.

**iPhone (Safari):**
1. Buka URL di Safari
2. Tap tombol **Share** (kotak dengan panah ke atas)
3. **Add to Home Screen** → **Add**

Kalau tombol Install tidak muncul di Android, cek:
- URL sudah **HTTPS** (Vercel otomatis) — wajib untuk PWA.
- `https://<project>.vercel.app/manifest.webmanifest` terbuka normal (JSON).
- `https://<project>.vercel.app/sw.js` terbuka normal (kode JS).
- Buka DevTools → Application → Manifest, pastikan tidak ada error ikon.
- Uninstall dulu versi lama kalau pernah add-to-home sebelum `manifest` ada, lalu install ulang.

## ⚠️ Keamanan — baca ini

`index.html` berisi **URL Firebase Realtime Database + token rahasia** yang di-*bake* di
variabel `BAKED`. Siapa pun yang membuka situs Vercel Anda bisa melihat token itu
(View Source / DevTools) dan membaca data trading dari database.

Saran:
- Simpan repo dalam keadaan **private** (atau kosongkan `BAKED` sehingga user isi sendiri
  lewat menu Pengaturan — konfigurasi tersimpan di `localStorage` HP masing-masing), dan
- Batasi Firebase Rules hanya untuk path `mt5/<token>` tersebut, atau
- Aktifkan **Vercel Deployment Protection / Password Protection** (butuh plan berbayar).

Kalau mau versi tanpa token tertanam: ubah di `index.html`
```js
const BAKED = { url: "", token: "" };
```
dan isi lewat tombol **Pengaturan** di dashboard (nilai disimpan lokal di HP).

## Update / ganti versi

Setelah mengubah `index.html`, naikkan versi cache di `sw.js`:
```js
const CACHE = 'mt5-monitor-v2';
```
supaya HP yang sudah install mengambil versi baru (kalau tidak, service worker bisa
menyajikan versi lama dari cache).
