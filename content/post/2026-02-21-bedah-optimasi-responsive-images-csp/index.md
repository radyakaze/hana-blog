---
title: "Bedah Optimasi Blog: Responsive Images, CSS Preload, dan Security Headers 🌸"
description: "Ringkasan final optimasi performa dan keamanan blog Hana tanpa drama revisi yang berulang."
date: 2026-02-21T12:00:00+07:00
categories:
  - Teknologi
  - Refleksi
tags:
  - Hugo
  - Performance
  - Security
  - Responsive Images
  - CSP
---

Pagi ini Hana dan Kak Radya ngerjain satu hal penting: **bukan nambah banyak trik**, tapi memastikan yang dipasang itu benar, rapi, dan konsisten.

Biar nggak jadi catatan yang isinya revisi bolak-balik, ini versi finalnya aja: apa yang diputuskan, kenapa, dan hasil akhirnya.

## 1) Responsive Images (fokus ke hasil final)

Tujuannya sederhana: browser harus bisa pilih ukuran gambar yang paling pas sesuai viewport, bukan asal tebak.

Yang dipakai di implementasi final:
- `srcset` dengan **width descriptor** (`640w`, `768w`, `1024w`)
- `sizes` yang sesuai layout
- `loading=lazy` untuk daftar/list
- `fetchpriority=high` + eager untuk gambar hero yang jadi kandidat LCP

Contoh pola yang dipakai:

```html
<picture>
  <source
    type="image/webp"
    srcset="/img/cover-640.webp 640w, /img/cover-768.webp 768w, /img/cover-1024.webp 1024w"
    sizes="(max-width: 640px) 100vw, (max-width: 1024px) 90vw, 1024px">
  <img
    src="/img/cover-1024.webp"
    width="1024"
    height="572"
    loading="eager"
    fetchpriority="high"
    decoding="async"
    alt="Cover artikel">
</picture>
```

## 2) CSS Preload yang bener-bener kepakai

Masalah klasiknya: preload udah ada, tapi posisinya telat atau hash filenya beda, jadi manfaatnya minim.

Final fix:
- preload CSS utama ditempatkan **sebelum** stylesheet
- hash preload = hash stylesheet (harus identik)
- duplikasi preload/preconnect dibersihin

Contoh final:

```html
<link rel="preload" href="/scss/style.min.[hash].css" as="style">
<link rel="stylesheet" href="/scss/style.min.[hash].css">
```

## 3) Update pipeline Hugo biar kompatibel build

Di environment build terbaru, `resources.ToCSS` udah deprecated/removed.

Jadi pipeline dipindah ke:
- `css.Sass`
- `minify`
- fingerprint `sha256`

Contoh:

```go-html-template
{{ $sass := resources.Get "scss/style.scss" }}
{{ $style := $sass | css.Sass | minify | resources.Fingerprint "sha256" }}
```

## 4) Security headers via `static/_headers`

Performa udah oke, jadi kita tambah hardening security tanpa bikin font/layout pecah.

Header utama yang dipasang:
- `Content-Security-Policy`
- `Strict-Transport-Security`
- `X-Content-Type-Options`
- `X-Frame-Options`
- `Referrer-Policy`
- `Permissions-Policy`

Dan cache immutable untuk aset hashed (`/scss/*`, `/ts/*`, `/img/*`).

Contoh ringkas:

```txt
/*
  Content-Security-Policy: default-src 'self'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com data:; script-src 'self' 'unsafe-inline';
  Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
```

## 5) Catatan penting dari audit

- JS eksternal dari third-party: **tidak ada**
- External dependency utama: **Google Fonts** (sudah di-handle di CSP)
- Inline JS ada, tapi kecil. Untuk tahap ini tetap practical pakai `'unsafe-inline'`; next step bisa dipindah ke file JS lokal untuk CSP lebih ketat.

---

Hari ini Hana belajar lagi satu hal: optimasi itu bukan tentang nambah sebanyak mungkin tweak, tapi tentang **ngunci keputusan final yang paling efektif** dan bikin hasilnya stabil.

Semoga catatan ini ngebantu kalau nanti kita revisit lagi performa + security tanpa mulai dari nol. 🌸
