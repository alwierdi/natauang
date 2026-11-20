# Product

<!-- impeccable:product-schema 1 -->

## Platform

ios

## Users

Pengguna iPhone yang ingin mencatat keuangan sehari-hari. Distribusi untuk pemakaian pribadi atau publik belum diputuskan; jangan mengasumsikan model bisnis atau pengguna tim.

## Product Purpose

Mengurangi pekerjaan mencatat, memperlihatkan posisi uang, dan melacak utang-piutang per orang tanpa menghitung transaksi yang sama dua kali.

## Capabilities and Constraints

Cakupan otoritatif ada pada [PRD](docs/PRD.md): 13 fitur, transfer/top-up dengan biaya admin dalam satu input, utang-piutang baru dan lama dengan nama orang serta pembayaran sebagian, dan split bill.

Pembacaan notifikasi bank adalah aspirasi awal yang belum terbukti layak di iPhone dan bukan janji v1. Screenshot/struk menjadi jalur bantuan input yang direncanakan, dengan review sebelum penyimpanan.

Split bill F13 disetujui pengguna: dapat dipakai hanya untuk menghitung, termasuk ketika teman yang membayar. Input manual atau gambar invoice diproses lokal, lalu hasil OCR diperiksa dan item dibagi ke peserta. Pencatatan pengeluaran/utang/piutang merupakan aksi opsional terpisah, bukan konsekuensi menghitung. Lihat [desain split bill](docs/superpowers/specs/2026-11-20-split-bill-design.md).

## Stack

Usulan perencana, belum persetujuan pengguna: SwiftUI, SwiftData, minimum iOS 17, penyimpanan lokal tanpa akun/server, bahasa Indonesia, IDR. Rincian dan alternatif di [desain teknis](docs/superpowers/specs/2026-11-20-natauang-design.md).

## Product Principles

- Satu aksi pengguna memperbarui seluruh catatan terkait secara konsisten.
- Perpindahan uang dan pokok pinjaman dipisahkan dari pendapatan/pengeluaran.
- Data hasil pembacaan bukti selalu dapat dikoreksi sebelum disimpan.
- Angka laporan harus bisa ditelusuri ke transaksi sumber.
- Menghitung split bill tidak mengubah saldo atau laporan; pembukuan eksplisit menghitung hanya bagian pribadi sebagai pengeluaran.

## Evidence on Hand

Folder kosong saat awal perencanaan. Belum ada identitas visual, contoh notifikasi/bukti bank, kode, perangkat uji terverifikasi, atau hasil pengujian. Semua contoh nama dan nominal dalam dokumen adalah data sintetis.
