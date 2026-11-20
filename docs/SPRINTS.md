# Sprint tracker NataUang

Baseline: 20 November 2026. Status aplikasi: **belum diimplementasikan**. F13 split bill disetujui untuk ditambahkan ke dokumentasi dan target v1; baseline teknis/UX lain belum disetujui. Tidak ada jadwal kalender atau janji durasi; sprint di bawah adalah urutan increment. Setelah akses Mac dan kapasitas kerja diketahui, tambahkan tanggal mulai/target per sprint tanpa menghapus riwayat revisi.

## 1. Cara membaca dan memperbarui

Status yang digunakan: BACKLOG → READY → IN_PROGRESS → REVIEW → DONE. BLOCKED berarti ada hambatan konkret; isi alasan dan tindakan pembukanya. Dependensi pada tabel berarti syarat **mulai**, kecuali gate DONE yang disebut terpisah.

Kolom Bukti berisi link laporan `evidence/<TASK-ID>.md` setelah benar-benar ada, beserta commit/PR bila tersedia. Gunakan [format bukti](evidence/README.md). Tanda `—` berarti belum ada, bukan lulus. Owner `Belum ditetapkan` bukan penugasan otomatis. Tanggal update di setiap baris harus diubah saat status berubah.

**Definition of Ready:** kebutuhan task jelas, dependensi mulai DONE, keputusan terkait diterima, test/acceptance diketahui, lingkungan yang dibutuhkan tersedia. **Definition of Done:** hasil sesuai acceptance pada [rencana](superpowers/plans/2026-11-20-natauang.md), test relevan benar-benar lulus, review selesai, bukti dan status diperbarui, tidak ada bug saldo/keamanan/kehilangan data yang tersisa dalam task.

Progres = jumlah task DONE / seluruh task × 100%. Tidak memakai estimasi persentase subjektif. Saat ini **0/26 task DONE (0%)**; paket dokumentasi tidak dihitung sebagai implementasi fitur. Perbarui angka ini bersamaan dengan tabel ketika ada perubahan.

## 2. Task board

Acceptance rinci setiap ID ada di bagian dengan ID yang sama dalam rencana implementasi. Kode F merujuk [PRD](PRD.md).

| ID | Task / cakupan | Status | Dependensi mulai | Owner | Update | Bukti |
| --- | --- | --- | --- | --- | --- | --- |
| S0-01 | Review baseline PRD, D01–D05 dan UX | READY | — | Belum ditetapkan | 2026-11-20 | — |
| S0-02 | Shell aplikasi, scheme dan lingkungan Mac | BLOCKED | S0-01 | Belum ditetapkan | 2026-11-20 | — |
| S0-03 | Riset capture iOS dan fixture sintetis (F02) | REVIEW | — | Agent utama | 2026-11-20 | [S0-03](evidence/S0-03.md) |
| S1-01 | Model, uang dan tanggal | BACKLOG | S0-01, S0-02 | Belum ditetapkan | 2026-11-20 | — |
| S1-02 | Dompet dan ledger (F03) | BACKLOG | S1-01 | Belum ditetapkan | 2026-11-20 | — |
| S1-03 | Persistence, rollback dan idempotency | BACKLOG | S1-02 | Belum ditetapkan | 2026-11-20 | — |
| S2-01 | Pencatatan dan kategori adaptif (F01, F05) | BACKLOG | S1-03 | Belum ditetapkan | 2026-11-20 | — |
| S2-02 | Transfer/top-up dengan admin (F04) | BACKLOG | S2-01 | Belum ditetapkan | 2026-11-20 | — |
| S2-03 | Kartu kredit dan refund (F03, F08) | BACKLOG | S2-02 | Belum ditetapkan | 2026-11-20 | — |
| S3-01 | Orang, utang baru dan lama (F10) | BACKLOG | S2-03 | Belum ditetapkan | 2026-11-20 | — |
| S3-02 | Pembayaran sebagian dan koreksi (F10) | BACKLOG | S3-01 | Belum ditetapkan | 2026-11-20 | — |
| S4-01 | Ringkasan dan pencarian (F08, F11) | BACKLOG | S3-02 | Belum ditetapkan | 2026-11-20 | — |
| S4-02 | Anggaran dan periode (F06) | BACKLOG | S4-01 | Belum ditetapkan | 2026-11-20 | — |
| S5-01 | Tagihan dan langganan (F07) | BACKLOG | S4-02 | Belum ditetapkan | 2026-11-20 | — |
| S5-02 | Alokasi target tabungan (F09) | BACKLOG | S4-02 | Belum ditetapkan | 2026-11-20 | — |
| S5-03 | Pengingat tagihan/utang (F07, F10) | BACKLOG | S3-02, S5-01 | Belum ditetapkan | 2026-11-20 | — |
| S6-01 | OCR dan parser lokal (F02) | BACKLOG | S0-03, S2-03 | Belum ditetapkan | 2026-11-20 | — |
| S6-02 | Review draf dan duplikasi (F02) | BACKLOG | S6-01 | Belum ditetapkan | 2026-11-20 | — |
| S6-03 | Share Extension dan inbox (F02) | BACKLOG | S6-02 | Belum ditetapkan | 2026-11-20 | — |
| S6-04 | Split bill hitung saja, item dan invoice OCR (F13) | BACKLOG | S6-02 | Belum ditetapkan | 2026-11-20 | — |
| S6-05 | Pembukuan split bill opsional dan atomik (F13) | BACKLOG | S6-04, S3-02, S4-01 | Belum ditetapkan | 2026-11-20 | — |
| S7-01 | App lock dan privasi (F12) | BACKLOG | S0-02 | Belum ditetapkan | 2026-11-20 | — |
| S7-02 | Ekspor CSV (F12) | BACKLOG | S4-01, S7-01 | Belum ditetapkan | 2026-11-20 | — |
| S7-03 | Backup/restore portabel (F12) | BACKLOG | S5-02, S5-03, S6-03, S6-05, S7-01, S7-02 | Belum ditetapkan | 2026-11-20 | — |
| S8-01 | Integrasi, aksesibilitas dan performa | BACKLOG | Semua task S1–S7 | Belum ditetapkan | 2026-11-20 | — |
| S8-02 | Kandidat rilis dan handoff | BACKLOG | S8-01 | Belum ditetapkan | 2026-11-20 | — |

S7-01 dapat mulai lebih awal untuk menyediakan lock adapter; gate DONE S6-03 dan S7-01 memerlukan pengujian integrasi bersama. Nomor sprint merupakan paket milestone; penjadwalan boleh memajukan pekerjaan independen tanpa melewati gate.

S6-04 dapat memulai kalkulator dan OCR setelah S6-02; gate DONE menunggu integrasi Share Sheet/inbox S6-03 dan unlock S7-01. Dua task baru masih BACKLOG karena dependency implementasi belum selesai; persetujuan dokumen F13 bukan bukti fitur selesai.

## 3. Tujuan dan gate tiap sprint

| Sprint | Hasil yang dapat diperiksa | Gate keluar |
| --- | --- | --- |
| 0 | Baseline diterima, app shell jalan, riset capture | D01–D05 tercatat; build/launch pada Mac; matriks capture tersedia |
| 1 | Dompet tersimpan dan saldo dapat dihitung | Restart, rollback, overflow dan retry teruji |
| 2 | Pengeluaran/pemasukan/transfer + admin dan kartu | Saldo dan laporan benar setelah create/edit/delete/refund |
| 3 | Utang-piutang per orang | Baru/lama, sebagian/lunas, koreksi dan riwayat benar |
| 4 | Laporan, pencarian dan anggaran | Total cocok dengan transaksi, periode konsisten |
| 5 | Tagihan, pengingat dan target tabungan | Tidak membukukan pembayaran/alokasi secara keliru |
| 6 | Bukti gambar menjadi draf; split manual/invoice dan pembukuan opsional | Review wajib, total pembagian tepat, hitung saja tanpa posting, pelunasan tidak menggandakan pengeluaran |
| 7 | Lock, CSV, backup/restore | Restore lintas instalasi, kegagalan aman dan data terkunci |
| 8 | Kandidat v1 seluruh 13 fitur | Suite final, uji perangkat, acceptance dan handoff lengkap |

## 4. Hambatan dan keputusan

| ID | Terkait | Kondisi sekarang | Tindakan pembuka |
| --- | --- | --- | --- |
| B01 | S0-01/S0-02 | Usulan teknis/UX belum disetujui | Review PRD, UX, desain teknis dan catat D01–D05 |
| B02 | S0-02 dan semua verifikasi iOS | Windows: `swift`, `xcodebuild` dan `xcrun` tidak ditemukan pada PATH; Mac/Xcode belum terverifikasi | Sediakan/verifikasi Mac dan simulator atau runner macOS yang diizinkan; rekam akses dan versi |
| B03 | S6-01 | Belum ada contoh bukti bank yang diuji | Mulai fixture sintetis S0-03, lalu evaluasi format secara eksplisit; tidak meminta kredensial bank |
| B04 | S8-02 | Jalur distribusi belum dipilih | Putuskan pribadi/TestFlight/App Store sebelum gate rilis |

B03/B04 belum memblokir dokumentasi atau seluruh proyek. Jangan menandai semua task BLOCKED hanya karena belum waktunya dikerjakan.

## 5. Matriks cakupan

| Fitur | Task utama | Bukti acceptance yang diperlukan |
| --- | --- | --- |
| F01 | S2-01 | Create/edit/delete pemasukan/pengeluaran; input cepat |
| F02 | S0-03, S6-01–S6-03 | OCR/parser, review, gagal baca, share dan duplikat |
| F03 | S1-02, S2-03 | Saldo awal/arsip, aset/kewajiban/saldo kredit |
| F04 | S2-02 | Admin nol/berbayar, edit/hapus, rollback |
| F05 | S2-01 | Kategori custom/arsip, saran dan koreksi merchant |
| F06 | S4-02 | Global/kategori, sisa, lewat batas, refund dan periode |
| F07 | S5-01, S5-03 | Jadwal, konfirmasi bayar, izin dan pembatalan pengingat |
| F08 | S4-01 | Angka laporan dan drill-down cocok |
| F09 | S5-02 | Alokasi/penarikan, progres dan kekurangan dana |
| F10 | S3-01, S3-02, S5-03 | Nama orang, utang lama/baru, pembayaran dan pengingat |
| F11 | S4-01 | Pencarian teks/nominal dan filter gabungan |
| F12 | S7-01–S7-03 | Lock/fallback, CSV, backup dan restore aman |
| F13 | S0-03, S6-04, S6-05 | Hitung saja/manual/invoice, rata/custom/item bersama, penyesuaian/pembulatan, kedua arah pembayar, pembukuan atomik dan pelunasan tanpa hitung ganda |

## 6. Log perubahan dan handoff

| Tanggal | Perubahan | Dampak |
| --- | --- | --- |
| 2026-11-20 | Baseline dokumen pertama dibuat atas permintaan pengguna | Tidak ada kode atau task implementasi selesai |
| 2026-11-20 | Review desain menemukan kasus refund setelah pelunasan kartu | Ditambahkan usulan saldo kredit kartu dan test S2-03; bukan fitur pembayaran baru |
| 2026-11-20 | Pengguna menyetujui split bill hitung saja, pembukuan opsional dan baca gambar invoice lokal | Cakupan v1 menjadi F01–F13; tambah S6-04/S6-05 dan perluas backup/CSV/gate integrasi. D01–D05 tetap belum disahkan; [bukti pembaruan dokumen](evidence/DOC-002.md) |
| 2026-11-20 | Memulai implementasi; S0-03 menghasilkan riset Apple dan 18 fixture sintetis | S0-03 REVIEW; Windows tidak memiliki swift/xcodebuild/xcrun pada PATH, S0-02 tetap BLOCKED; tidak ada build iOS |

Handoff saat ini: S0-03 menunggu review, dengan [bukti riset dan fixture](evidence/S0-03.md). S0-01 masih memerlukan pencatatan keputusan baseline D01–D05; S0-02 memerlukan Mac/Xcode. Task implementasi berikutnya belum READY. Tidak ada build atau test aplikasi yang diklaim tersedia.
