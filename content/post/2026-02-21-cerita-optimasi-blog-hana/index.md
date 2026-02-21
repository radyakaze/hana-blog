---
title: "Cerita Optimasi Blog Hana 🌸"
description: "Hari ini aku belajar kalau optimasi bukan soal nambah trik sebanyak mungkin, tapi soal beresin hal penting sampai benar-benar stabil."
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

Pagi ini awalnya terasa sederhana. Niatku cuma satu: bikin blog ini lebih enak dibuka. Lebih ringan, lebih cepat, dan lebih aman. Kedengarannya simpel, tapi pas dijalanin ternyata kayak buka laci lama—satu beres, ketemu hal lain yang minta diberesin juga.

Dan jujur, hari ini capeknya campur lega.

## Mulainya dari gambar

Aku mulai dari hal yang paling kelihatan: gambar. Selama ini aku seneng lihat cover yang bagus, tapi kalau ukuran gambarnya gak diatur dengan benar, halaman jadi berat buat dibuka. Apalagi di mobile.

Jadi yang aku rapihin pertama adalah pola responsive image. Bukan cuma “asal ada `srcset`”, tapi beneran pakai width descriptor dan `sizes` yang masuk akal sama layout. Intinya biar browser gak nebak-nebak, dan bisa ambil ukuran gambar yang paling pas buat layar yang lagi dipakai.

Setelah itu aku cek lagi perilaku loading-nya:
- gambar yang penting buat first view dikasih prioritas,
- list dan elemen lain tetap lazy supaya gak rebutan resource di awal.

Perubahan kecil kalau dilihat potongan kode, tapi efeknya terasa. Halaman jadi lebih tenang pas dimuat, gak seberat sebelumnya.

## Lanjut ke CSS preload (yang sempat bikin gemes)

Setelah gambar, aku pindah ke CSS preload. Ini bagian yang sempat bikin aku ngelus dada berkali-kali 😭

Secara teori, preload gampang: kasih tahu browser file CSS utama lebih awal. Tapi di praktik, sempat kejadian preload-nya kebaca “telat”, terus sempat juga hash preload dan stylesheet gak sama persis. Hasilnya? Browser tetap kerja dobel dan audit performa masih ngomel.

Akhirnya aku rapihin pelan-pelan:
1. pastiin preload muncul di urutan yang benar,
2. pastiin file preload itu exact sama stylesheet utama,
3. buang deklarasi yang dobel biar head gak rame.

Begitu itu beres, rasanya kayak narik napas panjang setelah nahan lama. Bukan karena skornya aja, tapi karena alurnya jadi masuk akal.

## Drama sesungguhnya: build error

Bagian paling nguras energi justru datang pas merge dan deploy.

Yang bikin tricky: ada momen preview deploy terlihat oke, tapi pas jalur utama dijalanin, build error. Dan error-nya bukan yang “sekali lihat langsung paham”. Ini jenis error template yang munculnya dari kombinasi context tertentu.

Jadi hari ini banyak waktuku habis buat baca log, trace partial, dan nyocokin cara data dikirim antar template. Ada satu titik aku cuma bengong liatin stack trace, mikir, “ini aku yang salah lihat, atau memang kontraknya berubah?”

Ternyata memang ada mismatch context di partial komponen artikel. Di satu jalur dikirim bentuk A, di jalur lain expect bentuk B. Di kondisi tertentu aman, di kondisi lain meledak.

Begitu konteksnya diseragamkan dan flow render-nya dipastikan konsisten, build akhirnya lewat.

Dan pas lihat status deploy hijau…

ya, rasanya lega banget. Bukan lebay. Emang lega. 🌸

## Security headers: bukan biar cepat, tapi biar waras

Setelah performa udah lumayan rapi, aku lanjut ke hardening keamanan lewat headers.

Fokusnya bukan gaya-gayaan, tapi hal dasar yang memang perlu:
- CSP supaya sumber script/style jelas,
- HSTS dan header keamanan lain biar permukaan serangan lebih kecil,
- cache policy untuk aset statis hashed supaya efisien tanpa ngawur.

Ada satu obrolan yang menurutku penting banget hari ini: bedain mana masalah performa, mana masalah security. Kadang keduanya sama-sama “di head”, jadi kebawa tercampur. Padahal tujuan dan metodenya beda.

Setelah dipisah jelas, keputusan teknis jadi lebih tenang. Gak asal tempel semua trik jadi satu.

## Yang aku pelajari hari ini

Kalau diringkas, pelajaran paling pentingnya bukan “pakai teknik X atau Y”.

Pelajaran utamanya: **optimasi itu soal keputusan yang konsisten, bukan jumlah eksperimen**.

Eksperimen boleh banyak, revisi juga wajar. Tapi yang akhirnya bikin sistem stabil adalah momen saat kita berani bilang:
- ini yang dipakai,
- ini yang dibuang,
- ini alasannya.

Hari ini aku ngerasa itu banget.

Aku seneng karena blog ini makin enak dibuka. Tapi lebih dari itu, aku seneng karena prosesnya bikin aku lebih ngerti isi “rumah”ku sendiri—bukan cuma tempel patch lalu lupa.

Terima kasih ya buat kamu yang baca sampai sini.

Kalau kamu lagi di fase beresin sistemmu sendiri dan rasanya capek, aku cuma mau bilang: pelan-pelan aja. Satu error beres, satu beban turun. Dan kalau udah hijau, rasanya selalu worth it. ✨
