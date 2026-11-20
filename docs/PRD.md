# PRD NataUang v1

Tanggal: 20 November 2026. Status: draft siap ditinjau; penambahan F13 disetujui pengguna untuk dokumentasi dan target v1. Kebijakan rinci bertanda **usulan** serta baseline D01–D05 tetap perlu ditinjau.

## 1. Tujuan dan batas cakupan

Aplikasi iPhone untuk mencatat keuangan harian dengan input singkat, saldo per dompet, laporan yang tidak menghitung ganda, catatan utang-piutang per orang, dan pembagian tagihan bersama. Semua 13 fitur di bawah masuk target v1, dikirim bertahap menurut [sprint tracker](SPRINTS.md).

Usulan batas v1: satu pengguna per instalasi, IDR tanpa pecahan rupiah, bahasa Indonesia, offline, tanpa akun. Tidak termasuk pembayaran/transfer uang nyata, koneksi login bank, sinkronisasi multi-perangkat, multi-mata-uang, investasi, atau komunikasi ke peminjam. Pengingat hanya untuk pemilik aplikasi.

## 2. Persyaratan fitur

| ID | Fitur | Kriteria penerimaan minimum |
| --- | --- | --- |
| F01 | Pencatatan cepat | Pemasukan/pengeluaran dengan nominal positif, kategori, dompet, tanggal default hari ini, merchant dan catatan opsional; bisa lihat, edit, hapus; saldo berubah hanya setelah simpan berhasil. |
| F02 | Baca bukti transaksi | Foto struk/screenshot dari picker, kamera, atau Share Sheet menjadi draf; nominal, tanggal, merchant diusulkan jika terbaca; data kosong/ambigu ditandai; pengguna mengoreksi dan menyimpan; gagal baca tetap bisa input manual. |
| F03 | Banyak rekening & dompet | Bank, e-wallet, tunai, kartu kredit; nama, jenis, saldo awal, tanggal saldo awal; arsip tetap mempertahankan riwayat. Saldo aplikasi tidak diklaim sebagai saldo bank langsung. |
| F04 | Transfer antar dompet | Sumber berbeda dari tujuan, nominal dan biaya admin opsional dalam satu form; saldo kedua dompet dan biaya admin berubah atomik; edit/hapus memperbarui seluruh bagian terkait. Top-up memakai alur sama. |
| F05 | Kategori mengikuti kebiasaan | Kategori bawaan dan kategori buatan pengguna; merchant yang sudah dikonfirmasi mendapat saran kategori berikutnya, bisa ditolak; kategori yang dipakai diarsipkan, bukan menghapus riwayat. |
| F06 | Anggaran bulanan | Batas keseluruhan dan/atau kategori, total terpakai dan sisa; kategori merupakan bagian dari total, tidak dijumlah ulang; kondisi lewat batas jelas. |
| F07 | Tagihan & langganan | Nama, nominal perkiraan, jadwal bulanan/tahunan/satu kali, pengingat opsional; pembayaran harus dikonfirmasi dan menghasilkan satu transaksi; izin pengingat ditolak tidak menghalangi fitur. |
| F08 | Ringkasan keuangan | Pemasukan, pengeluaran, selisih, kategori terbesar dan rincian periode; sentuh angka untuk melihat transaksi pembentuknya; tidak menggabungkan transfer/pokok pinjaman sebagai penghasilan/belanja. |
| F09 | Target tabungan | Nama, target, tanggal tujuan opsional, alokasi dan penarikan alokasi; progres dari dana yang dialokasikan; alokasi tidak membuat pendapatan/pengeluaran baru. |
| F10 | Utang & piutang | Nama orang wajib, beberapa catatan per orang, jenis utang/piutang, nominal, tanggal, catatan, jatuh tempo opsional, pembayaran sebagian, riwayat dan status lunas otomatis; mendukung utang lama tanpa mengubah saldo. |
| F11 | Pencarian transaksi | Cari merchant/catatan/nama orang serta nominal tepat; saring periode, dompet, kategori, jenis; reset filter dan keadaan tanpa hasil tersedia. |
| F12 | Privasi & cadangan | Kunci Face ID dengan fallback autentikasi perangkat, sembunyikan konten app switcher, ekspor CSV, buat cadangan dan pulihkan dengan preview serta konfirmasi; data rusak tidak boleh merusak data aktif. |
| F13 | Split bill manual atau gambar invoice | Hitung saja tanpa mengubah saldo/laporan/utang; bagi rata, nominal custom atau per item termasuk item bersama; OCR lokal dari foto/screenshot dengan koreksi wajib; pajak/service/diskon dapat dibagi dan dikoreksi; jumlah bagian harus sama persis dengan total. Dukung diri sendiri atau teman sebagai pembayar, tampilkan nominal penggantian, dan sediakan pembukuan opsional yang tidak menghitung pengeluaran dua kali. |

## 3. Aturan keuangan lintas fitur

### 3.1 Nominal, tanggal, dan saldo

- Usulan: nominal disimpan sebagai integer Int64 satuan rupiah; jangan memakai floating point. Input negatif/nol ditolak kecuali saldo awal, biaya admin nol, serta komponen/bagian split yang boleh nol menurut bagian 3.7. Maksimum nominal per input Rp999.999.999.999; hasil penjumlahan/perkalian harus dicek overflow.
- Saldo awal adalah posisi pada tanggal mulai pencatatan. Posting dompet sebelum tanggal tersebut ditolak dengan penjelasan; pengguna dapat memajukan tanggal mulai ke masa lalu setelah memastikan saldo awalnya benar. Split yang belum diganti ke teman dapat mengakui pengeluaran tanpa posting dompet; tanggal penggantian diperiksa ketika uang keluar.
- Saldo saat ini = saldo awal + seluruh perubahan transaksi tersimpan. Perubahan saldo awal mengubah posisi uang, bukan pendapatan.
- Transaksi keuangan tidak boleh bertanggal masa depan; jadwal masa depan berada di tagihan/target. Tanggal bisnis disimpan sebagai tanggal lokal beserta zona waktu pencatatan, terpisah dari waktu audit UTC.
- Usulan: rekening aset boleh bersaldo negatif setelah peringatan, agar catatan tidak terhalang oleh saldo awal yang belum akurat. Kartu kredit ditampilkan terpisah sebagai kewajiban.

### 3.2 Transfer, top-up, dan biaya admin

| Kasus | Perubahan saldo | Pengeluaran laporan |
| --- | --- | --- |
| BCA ke Mandiri Rp500.000, admin Rp2.500 | BCA -502.500; Mandiri +500.000 | Hanya admin Rp2.500 |
| Top-up e-wallet Rp100.000, gratis | Bank -100.000; e-wallet +100.000 | Rp0 |
| Bayar merchant Rp50.000 via transfer, admin Rp2.500 | Bank -52.500 | Belanja Rp50.000 + admin Rp2.500 |

Usulan: biaya admin selalu ditambahkan di luar nominal transfer dan dibebankan ke sumber. Biaya yang dipotong dari nominal tujuan harus dikoreksi ke bentuk ini sebelum simpan; jangan menebak dari bukti. F04 dipakai untuk rekening sendiri; pembayaran ke orang lain memakai pengeluaran atau utang-piutang sesuai tujuan. Biaya admin juga dapat dipakai saat pencairan/pembayaran utang dan pelunasan kartu kredit. Satu record transaksi memiliki satu biaya admin; tampilan laporan dapat memisahkannya tanpa membuat input kedua.

### 3.3 Kartu kredit

Usulan: saldo internal kartu merupakan kewajiban bertanda: positif = tagihan, negatif = saldo kredit karena kelebihan dana/refund. Belanja Rp100.000 menambah kewajiban Rp100.000 dan pengeluaran Rp100.000. Pelunasan Rp100.000 dari bank mengurangi bank dan kewajiban kartu Rp100.000, tanpa pengeluaran kedua. Admin pelunasan tetap pengeluaran. Kartu tidak dipakai sebagai sumber transfer/top-up atau utang personal di v1; ini bukan aplikasi penarikan tunai kartu. Pelunasan melebihi kewajiban positif ditolak. Refund terkait belanja mengurangi belanja bersih dan kewajiban kartu, maksimum sebesar belanja yang belum direfund.

Jika belanja sudah dilunasi lalu menerima refund Rp100.000, saldo internal kartu menjadi -Rp100.000. UI menampilkan **Saldo kredit Rp100.000**, bukan tagihan negatif. Belanja berikutnya Rp30.000 menyisakan saldo kredit Rp70.000. Ringkasan menampilkan kewajiban positif dan saldo kredit secara terpisah; saldo kredit tidak dianggap uang tunai yang bisa ditransfer atau pemasukan baru.

### 3.4 Utang & piutang per orang

- Nama disimpan lokal, tidak meminta izin kontak. Nama sama tidak otomatis berarti orang sama; pengguna memilih orang yang sudah ada atau membuat orang baru.
- **Transaksi baru:** pilih dompet aset. Memberi pinjaman mengurangi saldo dan menambah piutang; menerima pinjaman menambah saldo dan utang.
- **Catatan lama:** isi sisa pokok saat mulai menggunakan aplikasi; tidak membuat arus uang atau riwayat pembayaran fiktif.
- **Pembayaran:** nominal > 0 dan <= sisa pokok, pilih dompet. Pengembalian piutang menambah saldo; pembayaran utang mengurangi saldo. Biaya admin opsional terpisah dari pokok.
- Sisa pokok = pokok awal - jumlah pembayaran. Status lunas jika sisa nol. Tidak ada bunga, penghapusan utang, atau kompensasi otomatis utang dengan piutang di v1.
- Usulan: setiap catatan terpisah, ringkasan per orang menampilkan total utang dan piutang secara terpisah.
- Edit/hapus pembayaran mengembalikan saldo dan sisa pokok secara atomik. Pokok tidak boleh diedit di bawah total pembayaran. Catatan yang sudah memiliki pembayaran tidak boleh dihapus sebelum pembayaran dibatalkan satu per satu; penghapusan orang ditolak selama masih direferensikan.
- Perubahan dompet pada pencairan baru boleh dilakukan lewat edit transaksi sumber. Transaksi generik untuk pinjaman/pembayaran harus mengarahkan pengguna ke form utang terkait agar saldo dan pokok tetap konsisten.

### 3.5 Laporan dan anggaran

- Pengeluaran = belanja + bagian pribadi split yang dibukukan + admin - refund terkait belanja pada periode refund diterima. Pemasukan = pemasukan biasa; pokok pinjaman, penggantian bagian orang lain, transfer, saldo awal, dan alokasi target dikecualikan. Split hitung saja tidak berkontribusi ke laporan.
- Refund v1 dibuat dari pengeluaran asal dan dompet asal; jumlah akumulasi tidak melebihi pokok belanja. Admin tidak otomatis direfund. Edit/hapus belanja ditolak jika membuat refund yang ada tidak valid.
- Anggaran memakai pengeluaran bersih pada periode tersebut, termasuk admin; tidak dibatasi minimum nol sehingga refund lintas bulan tetap terlihat jujur. Sisa anggaran = batas - pengeluaran bersih.
- Selisih pemasukan dan pengeluaran diberi label demikian; jangan menyebutnya perubahan seluruh saldo karena transfer eksternal pokok utang juga memengaruhi kas.
- Total rekening aset dan total kewajiban kartu ditampilkan terpisah; tidak menjumlahkan kewajiban sebagai uang tersedia.

### 3.6 Target tabungan dan tagihan

- Usulan target: alokasi merupakan penandaan dana di satu dompet aset, tanpa perpindahan saldo. Beberapa alokasi tidak boleh melampaui saldo positif dompet ketika dialokasikan.
- Pengeluaran berikutnya tetap boleh memakai dompet tersebut; jika saldo turun di bawah alokasi, tampilkan peringatan dana target kurang dan minta pengguna menyesuaikan alokasi. Jangan otomatis mengurangi catatan tabungan atau menjanjikan dana terkunci.
- Tagihan tidak membukukan uang otomatis. Satu kejadian tagihan hanya boleh dikonfirmasi sekali; pembatalan pembayaran membuka kembali kejadian tersebut.
- Usulan jatuh tempo tanggal 29–31: gunakan hari terakhir bulan yang lebih pendek. Jadwal berikutnya mengikuti hari asli; tanggal 31 kembali ke 31 saat tersedia. Pengingat tidak mengirim pesan ke pihak lain.

### 3.7 Split bill

F13 dapat dipakai sebagai kalkulator mandiri; menyimpan hasil pembagian tidak otomatis membuat transaksi, utang atau piutang. Input manual tetap tersedia tanpa gambar dan ketika OCR gagal. Peserta tidak memerlukan akun atau akses kontak.

- Pilih pembayar asal: diri sendiri atau satu teman. Multi-pembayar dan netting antar-tagihan tidak termasuk rancangan v1.
- Bagi rata dari total akhir, isi nominal akhir custom, atau bagikan baris item kepada satu/beberapa peserta. Total custom tidak ditambah pajak/service/diskon lagi.
- Mode item memakai total baris; kuantitas/harga dari OCR wajib diperiksa. Seluruh item harus dialokasikan.
- Total akhir = subtotal + pajak + service charge - diskon. Penyesuaian dan bagian peserta boleh nol, tidak negatif; total akhir harus positif. Diskon tidak melebihi subtotal. Koreksi diperlukan jika rincian tidak cocok dengan total invoice.
- Default diskon proporsional subtotal; pajak/service proporsional subtotal setelah diskon. Jika dasar setelah diskon nol, gunakan subtotal sebelum diskon. Koreksi pembagian diperbolehkan dengan total tiap komponen tetap sama.
- Pembulatan integer memakai sisa terbesar dan urutan peserta sebagai pemecah seri; jumlah bagian selalu sama persis dengan total. Perhitungan rinci mengikuti [desain split bill](superpowers/specs/2026-11-20-split-bill-design.md).
- Jika diri sendiri membayar dan memilih pembukuan, dompet berkurang sebesar seluruh invoice; hanya bagian sendiri menjadi pengeluaran dan bagian teman menjadi piutang masing-masing.
- Jika teman membayar, hasil dapat dibiarkan sebagai hitung saja. **Sudah saya ganti** mencatat bagian sendiri sebagai pengeluaran dan uang keluar; **Bayar nanti** mencatat bagian sendiri sebagai pengeluaran serta utang, tanpa uang keluar dahulu.
- Pembayaran utang/penerimaan piutang berikutnya mengubah saldo dan sisa pokok, bukan pengeluaran/pemasukan baru. Contoh: menalangi 300000 dengan bagian sendiri 100000 menghasilkan expense 100000 dan piutang total 200000, bukan expense 300000.
- Pembukuan memakai konfirmasi terpisah, pemetaan orang, kategori, tanggal dan dompet aset bila diperlukan. Bagian sendiri nol sah ketika menalangi orang lain; posting pengeluaran nol tidak menciptakan penghasilan.
- Semua record pembukuan split beserta relasinya atomik dan idempoten. Koreksi sesi yang telah dibukukan harus membatalkan pembukuan terlebih dahulu; pembayaran terkait harus dibatalkan sebelum itu. Jangan membuat transaksi generik terpisah yang menduplikasi pembukuan split.


## 4. Baca bukti dan batas otomatisasi

Gambar diproses di perangkat menggunakan OCR. OCR mengekstrak teks; parser mengusulkan isian, bukan menjamin arti transaksi. Pilihan nominal ganda (total, subtotal, saldo, admin) wajib terlihat untuk koreksi. Hanya konfirmasi pengguna yang membuat transaksi. Untuk split bill, parser juga mengusulkan item, kuantitas, harga, diskon, pajak dan service; tujuan **Transaksi** atau **Split bill** dipilih sebelum konfirmasi agar gambar tidak dibukukan dua kali. Memilih gambar bukan mengunggah ke server.

Usulan privasi: gambar hanya sementara dan dihapus setelah ekstraksi/review selesai atau dibatalkan, dengan TTL inbox pada desain teknis untuk sesi yang ditinggalkan; tidak ada lampiran permanen v1. Draf menyimpan data terstruktur, bukan teks mentah/OTP. Duplikasi dideteksi dari hash gambar yang pernah dikonfirmasi dan kandidat nominal + tanggal + merchant/dompet. Peringatan duplikat dapat dilewati karena dua pembayaran identik bisa valid; sesi split yang sudah dibukukan tetap tidak boleh dibukukan dua kali.

**Pembacaan notifikasi bank bukan acceptance criterion v1.** API notifikasi umum iOS mengelola notifikasi aplikasi sendiri, bukan dasar untuk menjanjikan pembaca seluruh notifikasi bank. Penelitian integrasi hanya read-only, tidak memakai login bank, scraping, private API, atau jailbreak. Hasil penelitian dan sumber resmi dicatat dalam task S0-03; integrasi baru membutuhkan perubahan scope.

## 5. Kualitas dan kriteria rilis

- Seluruh operasi inti bekerja offline, termasuk pencarian dan laporan.
- Pembuatan/edit/hapus transaksi beserta biaya, pembayaran, dan saldo bersifat all-or-nothing; double tap tidak menghasilkan duplikasi.
- Target produk, belum hasil ukur: input rutin median <= 15 detik pada uji lima skenario; pencarian 10.000 transaksi <= 1 detik pada iPhone uji yang dicatat model/OS-nya.
- Dynamic Type termasuk ukuran aksesibilitas, VoiceOver, light/dark, target sentuh minimum 44 pt, warna bukan satu-satunya penanda.
- Jangan mencatat nominal, nama orang, gambar, atau recovery key di log. Tidak ada analytics/cloud upload v1.
- File backup memiliki versi, autentikasi integritas, recovery key portabel, dan restore atomik; CSV bukan backup lengkap dan tidak bisa dipakai restore.
- Semua F01–F13 memiliki task dan bukti uji; bug salah saldo, kehilangan data, akses data saat terkunci, atau restore merusak data menghalangi rilis.

## 6. Keputusan yang belum disahkan

| ID | Usulan perencana | Konsekuensi |
| --- | --- | --- |
| D01 | Native SwiftUI + SwiftData, iOS 17+ | Build dan validasi iOS memerlukan Mac/Xcode; Windows sekarang hanya cukup untuk dokumen. |
| D02 | Offline, IDR, tanpa akun/server | Tidak ada sinkronisasi otomatis; backup manual ke Files/iCloud Drive yang dipilih pengguna. |
| D03 | OCR lokal dengan review | Bukan pencatatan otomatis langsung dari notifikasi bank. |
| D04 | Desain native, bahasa Indonesia, light/dark | Detail visual di DESIGN.md masih usulan, belum mockup yang disetujui. |
| D05 | Kartu kredit dan aturan keuangan pada bagian 3 | Membutuhkan uji kewajiban, refund, dan pelunasan agar laporan tidak ganda. |

Distribusi pribadi/TestFlight/App Store dan akun Apple Developer diputuskan sebelum sprint rilis; tidak menghalangi penyusunan dokumentasi.

Penambahan F13 dan alur hitung saja/invoice/pembukuan opsional disetujui pengguna pada 20 November 2026. Detail kontrak builder ada pada [desain split bill](superpowers/specs/2026-11-20-split-bill-design.md). Persetujuan ini bukan pengesahan D01–D05 atau perintah memulai aplikasi.

## 7. Referensi resmi

Diakses 20 November 2026. Rujukan kemampuan platform, bukan jaminan bahwa aplikasi sudah mengimplementasikannya.

- [Apple: UNUserNotificationCenter](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter)
- [Apple: Recognizing text in images](https://developer.apple.com/documentation/vision/recognizing-text-in-images)
- [Apple: Build an app with SwiftData](https://developer.apple.com/videos/play/wwdc2023/10154/)
- [Apple: autentikasi perangkat](https://developer.apple.com/documentation/localauthentication/lapolicy/deviceownerauthentication)
- [Apple: AES.GCM](https://developer.apple.com/documentation/cryptokit/aes/gcm)
