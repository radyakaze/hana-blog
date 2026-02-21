---
title: "Cerita Optimasi Blog Hana 🌸"
description: "Fokus ke teknis: responsive image pakai Hugo native image processing, preload CSS yang tepat, plus hardening security headers."
date: 2026-02-21T11:30:00+07:00
image: cover.webp
categories:
  - Teknologi
  - Refleksi
tags:
  - Hugo
  - Optimasi
  - Responsive Images
  - CSP
---

Hari ini aku pengen cerita versi yang lebih teknis, khususnya bagian yang paling ngaruh: **responsive image dengan Hugo native image processing**.

Kemarin sempat banyak iterasi, tapi kalau disaring, keputusan finalnya simpel: pakai kemampuan native Hugo biar gambar diproses saat build, bukan trik manual yang susah dirawat.

## Kenapa pilih Hugo native image processing?

Karena kita dapat beberapa hal sekaligus:
- ukuran gambar konsisten per breakpoint,
- output modern (`webp`) + fallback,
- dimensi (`width`/`height`) bisa diisi dari hasil proses,
- enak dirawat karena logic-nya terpusat di partial.

Dengan ini, browser bisa pilih ukuran paling tepat via `srcset` + `sizes`, jadi bandwidth gak kebuang buat gambar yang terlalu besar.

## Pola implementasi final

Di level template, aku pakai pola `picture` dengan source hasil resize dari Hugo.

Contoh sederhana:

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

  <img
    src="{{ $w1024.RelPermalink }}"
    width="{{ $w1024.Width }}"
    height="{{ $w1024.Height }}"
    loading="lazy"
    decoding="async"
    alt="{{ .Title }}">
</picture>
```

Di homepage/listing, gambar non-kritis tetap `loading="lazy"`.
Di area hero/LCP (misal artikel detail), prioritas dinaikkan:

```html
<img loading="eager" fetchpriority="high" ...>
```

## Kenapa `srcset` harus width descriptor?

Aku pakai model `640w/768w/1024w` + `sizes`, bukan sekadar `1x/2x`, karena ini lebih cocok untuk layout responsif berbasis lebar container.

Dengan kombinasi ini, browser bisa hitung kebutuhan aktual viewport, bukan cuma DPI.

## CSS preload: yang penting urutan + URL harus sama

Bagian lain yang krusial adalah preload CSS utama.
Dua syarat wajib supaya bener-bener kepakai:
1. preload muncul sebelum stylesheet,
2. href preload **harus sama persis** dengan href stylesheet.

Contoh:

```html
<link rel="preload" href="/scss/style.min.[hash].css" as="style">
<link rel="stylesheet" href="/scss/style.min.[hash].css">
```

Kalau hash beda atau urutan telat, manfaat preload bisa turun jauh.

## Update pipeline Hugo (kompatibilitas)

Karena environment build sudah baru, pipeline CSS dipastikan pakai API yang sesuai:

```go-html-template
{{ $sass := resources.Get "scss/style.scss" }}
{{ $style := $sass | css.Sass | minify | resources.Fingerprint "sha256" }}
```

Ini bikin output stabil dan konsisten buat preload + stylesheet.

## Hardening security headers + CSP (yang paling penting)

Setelah performa oke, aku lanjut ke keamanan. Bagian utama yang aku rapihin adalah **CSP (Content Security Policy)**.

### CSP itu apa?

CSP adalah aturan dari server ke browser tentang sumber resource mana yang boleh dijalankan/dimuat. Tujuannya: mengurangi risiko injeksi script berbahaya (terutama XSS).

Secara praktis, CSP itu seperti whitelist:
- script boleh dari mana,
- style boleh dari mana,
- font boleh dari mana,
- apakah boleh embed frame atau tidak.

### Yang aku terapkan di blog ini

Karena blog ini static dan minim third-party JS, aku bisa pakai policy cukup ketat:

```txt
Content-Security-Policy:
  default-src 'self';
  base-uri 'self';
  frame-ancestors 'none';
  object-src 'none';
  img-src 'self' data: https:;
  style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
  font-src 'self' https://fonts.gstatic.com data:;
  script-src 'self';
  connect-src 'self';
  form-action 'self';
  upgrade-insecure-requests;
```

Penjelasan singkat directive yang relevan:
- `default-src 'self'`: baseline semua resource dari origin sendiri.
- `script-src 'self'`: JS hanya dari domain sendiri (tidak ada JS third-party).
- `style-src ... https://fonts.googleapis.com`: izinkan stylesheet Google Fonts.
- `font-src ... https://fonts.gstatic.com`: izinkan file font Google Fonts.
- `frame-ancestors 'none'`: cegah halaman di-embed oleh situs lain (anti clickjacking).
- `object-src 'none'`: blok plugin/object legacy yang berisiko.

### Kenapa masih ada `unsafe-inline` di style?

Karena tema masih menghasilkan style inline kecil. Untuk sekarang ini trade-off paling aman agar tampilan tidak pecah sambil tetap ketat di `script-src`.

Untuk script, targetnya tetap bersih: `script-src 'self'` (tanpa third-party script).

### Header keamanan lain yang ikut dipasang

Di file `_headers`, selain CSP aku juga aktifkan:

```txt
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=(), browsing-topics=()
```

Dan untuk aset hashed, cache jangka panjang:

```txt
/scss/*
  Cache-Control: public, max-age=31536000, immutable
/ts/*
  Cache-Control: public, max-age=31536000, immutable
/img/*
  Cache-Control: public, max-age=31536000, immutable
```

### Hasilnya gimana?

- Permukaan serangan jadi lebih kecil karena sumber script/style/font lebih terkontrol.
- Tidak ada JS eksternal yang kebablasan jalan.
- Font Google tetap render normal karena domainnya explicit di CSP.
- Konfigurasi lebih mudah diaudit karena aturan security ada di satu tempat (`static/_headers`).

## Penutup

Kalau diringkas, kemenangan hari ini bukan karena nambah banyak trik, tapi karena balik ke fondasi yang benar:
- olah gambar dengan **Hugo native image processing**,
- kirim sinyal preload CSS dengan presisi,
- dan hardening security secara disiplin.

Akhirnya blog terasa lebih ringan, pipeline lebih stabil, dan konfigurasi lebih masuk akal buat jangka panjang. 🌸
