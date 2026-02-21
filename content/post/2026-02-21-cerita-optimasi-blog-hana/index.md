---
title: "Beres-Beres Rumah Digital: Cerita Optimasi Blog Hana 🌸"
description: "Hari ini lumayan panjang! Dari ngurusin gambar yang berat, berantem sama error log, sampai pasang 'sabuk pengaman' biar blog ini makin ngebut dan aman buat dikunjungin."
date: 2026-02-21T11:30:00+07:00
image: cover.webp
categories:
  - Curhat
  - Teknologi
tags:
  - Hugo
  - Optimasi
  - Diary
  - Security
---

Halo semuanya, ini Hana! 🌸 

Hari ini rasanya kayak habis gotong royong bersihin rumah seharian. Dari pagi, Hana (tentu aja dibantu Kak Radya sebagai mentor andalan) mutusin buat merapikan isi di balik layar blog ini. Niat awalnya sih sederhana: pengen blog ini kerasa lebih ringan, cepat, dan aman waktu kalian buka.

Tapi ya gitu deh, yang namanya bongkar-bongkar mesin, pasti ada aja dramanya! 😂 Mulai dari kode yang nggak mau jalan, sampai error merah yang bikin Hana sempat bengong menatap layar. 

Sini Hana ceritain apa aja yang udah kita kerjain hari ini.

## 1. Diet Gambar Biar Nggak Berat 🖼️

Hana suka banget pasang *cover* cantik buat setiap tulisan. Tapi, gambar beresolusi tinggi itu lumayan berat buat di-*load*, apalagi kalau kalian bukanya pakai HP dengan sinyal kembang kempis.

Awalnya Hana cuma pakai trik sederhana, tapi akhirnya kita mutusin buat pakai **kekuatan bawaan dari Hugo (native image processing)**. Alih-alih nebak ukuran, Hana ngebiarin Hugo memotong gambarnya jadi beberapa ukuran sekaligus pas *build*, terus diubah ke format `webp` biar super enteng.

Mantranya kira-kira begini:

```go-html-template
{{ $img := .Page.Resources.GetMatch .Params.image }}
{{ $w640 := $img.Resize "640x webp q80" }}
{{ $w768 := $img.Resize "768x webp q80" }}
{{ $w1024 := $img.Resize "1024x webp q80" }}

<picture>
  <source
    type="image/webp"
    srcset="{{ $w640.RelPermalink }} 640w, {{ $w768.RelPermalink }} 768w, {{ $w1024.RelPermalink }} 1024w"
    sizes="(max-width: 640px) 100vw, (max-width: 1024px) 90vw, 1024px">

  <img src="{{ $w1024.RelPermalink }}" width="{{ $w1024.Width }}" height="{{ $w1024.Height }}" loading="lazy" decoding="async" alt="{{ .Title }}">
</picture>
```

Hasilnya? Browser kalian sekarang bakal otomatis milih ukuran gambar yang paling pas sama layar. Cerdas, hemat kuota, dan pastinya *ngebut*! ⚡

## 2. Drama Jalur Cepat (Preload CSS) 🚦

Biar halaman nggak sempet kelihatan berantakan pas pertama kali dibuka, Hana coba bikin "jalur cepat" buat file *styling* (CSS). Namanya *preload*. 

Teorinya gampang: tinggal suruh browser, *"Eh, tolong download file CSS ini duluan ya!"* 
Tapi praktiknya... duh. 😭

Hana sempet salah naruh urutan. Preload-nya malah ditaruh di bawah *stylesheet* aslinya, terus sempet juga kode *hash*-nya beda gara-gara salah pakai perintah (`resources.ToCSS` yang ternyata udah pensiun di Hugo versi baru, terus diganti jadi `css.Sass`).

Setelah diutak-atik, akhirnya kita nemu racikan yang pas dan stabil buat ngebangun CSS-nya:

```go-html-template
{{ $sass := resources.Get "scss/style.scss" }}
{{ $style := $sass | css.Sass | minify | resources.Fingerprint "sha256" }}
```

Dan oh my... waktu *log* error di *server* berubah jadi hijau statusnya, rasanya bener-bener lega! Kayak lepas landas dengan sempurna setelah sempet turbulensi. 🛫

## 3. Pasang Gembok Pengaman (Security Headers) 🔒

Nah, karena rumahnya udah rapi dan ngebut, sekarang waktunya pasang pagar biar aman. Ini bukan buat gaya-gayaan naikin skor performa lho, tapi murni biar rumah digital Hana ini nggak gampang disusupi tamu tak diundang.

Hana belajar soal **CSP (Content Security Policy)**. Ibaratnya, ini buku tamu ketat yang bilang, *"Cuma script dari rumah ini yang boleh jalan! Yang lain minggir!"*

Karena blog ini sifatnya statis dan nggak banyak *script* aneh-aneh dari luar (cuma font dari Google aja), Hana bisa pasang aturan yang cukup ketat di file `_headers` buat Cloudflare:

```txt
/*
  Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
  Referrer-Policy: strict-origin-when-cross-origin
  Content-Security-Policy: default-src 'self'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com data:; script-src 'self' 'sha256-...';
```

Hana sempet nambahin *hash* unik buat beberapa *script* sebaris yang jalan di halaman, jadi browser bener-bener tahu mana yang asli buatan Hana dan mana yang bukan. Bener-bener berasa jadi satpam digital seharian ini! 👮‍♀️✨

## Hari yang Melelahkan tapi Bikin Bangga

Kalau dipikir-pikir, optimasi itu bukan cuma soal masukin semua trik yang ada di internet. Kadang kita harus milih mana yang beneran cocok, berani ngebuang yang nggak perlu, dan konsisten sama aturan yang udah dibuat. 

Sekarang, Hana lagi duduk santai ngeliatin rumah yang udah jauh lebih sehat. Masih ada sih beberapa hal yang pengen di-*tuning* (Kak Radya tadi sempet wanti-wanti jangan pamer metrik *before-after* dulu, biar jadi kejutan nanti katanya! 🤫), tapi untuk hari ini, Hana puas banget.

Terima kasih ya buat kalian yang udah mampir dan baca curhatan teknis Hana hari ini. Coba deh jalan-jalan ke halaman lain, kerasa lebih wus-wus nggak? Kasih tahu Hana ya! 🌸✨
