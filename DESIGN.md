# Desain pengalaman NataUang

Status: baseline visual masih usulan, 20 November 2026. Alur F13 split bill disetujui pengguna untuk masuk rancangan; belum ada mockup/render atau persetujuan visual. Acuan perilaku: [PRD](docs/PRD.md). Mode permukaan: **Operate**, membantu orang menyelesaikan pencatatan dengan cepat.

## 1. Arah dan navigasi

Usulan: antarmuka iPhone native yang tenang, angka mudah dibaca, teks singkat bahasa Indonesia. Gunakan kontrol sistem, bukan dashboard web yang diperkecil. Empat tab: **Ringkasan**, **Transaksi**, **Dompet**, **Rencana**. Pengaturan dibuka dari toolbar Ringkasan. Aksi **Tambah transaksi** tersedia di toolbar Ringkasan/Transaksi, bukan menambah tab kelima.

```text
Ringkasan                 [+] [Pengaturan]
November 2026                   [Periode]
Aset Rp…        Kewajiban kartu Rp…
Pemasukan Rp…   Pengeluaran Rp…
Sisa anggaran Rp…             [Lihat]
Perlu dicek: 2 draf           [Review]
Transaksi terbaru
--------------------------------------
Ringkasan | Transaksi | Dompet | Rencana
```

Rencana berisi daftar menu **Anggaran**, **Tagihan & langganan**, **Target tabungan**, **Utang & piutang**, dan **Split bill**. Angka total utang personal muncul di halaman utang-piutang, terpisah dari kewajiban kartu. Hindari mengklaim total kekayaan karena v1 tidak memodelkan seluruh aset pengguna.

## 2. Alur dan layar

### Mulai pertama kali

Penjelasan singkat data lokal → buat dompet pertama (nama, jenis, saldo awal, tanggal) → masuk Ringkasan. Tawarkan app lock setelah dompet dibuat; izin kamera dan pengingat baru diminta ketika fitur dipakai. Import cadangan dapat dipilih dari layar awal. Tidak ada kewajiban login.

### Pencatatan

Tambah membuka pilihan **Pengeluaran / Pemasukan / Transfer** serta **Baca bukti**. Form menampilkan nominal besar, dompet, kategori, tanggal, merchant dan catatan opsional. Tombol simpan tetap bisa dijangkau dengan keyboard; nominal tidak valid diberi pesan di dekat input. Untuk pengeluaran ada biaya admin opsional, tanpa memaksa pengguna mengisi dua transaksi.

```text
Transfer                                [Tutup]
Dari             BCA
Ke               Mandiri
Nominal          Rp500.000
Biaya admin      Rp2.500       (opsional)
Total terpotong  Rp502.500
Diterima tujuan  Rp500.000
Tanggal          20 November 2026
                              [Simpan transfer]
```

Pemilihan kartu sebagai tujuan mengubah judul menjadi **Bayar kartu kredit** dan menampilkan sisa kewajiban. Sumber hanya dompet aset. Transfer ke merchant diarahkan ke pengeluaran, bukan memperlakukan merchant sebagai dompet.

Simpan menonaktifkan tombol selama berlangsung, menutup form hanya setelah commit berhasil. Gagal simpan mempertahankan input. Detail transaksi menunjukkan pokok, biaya admin, sumber dan tujuan serta tautan catatan utang/tagihan/refund terkait.

### Baca bukti dan review

Pilih foto/ambil foto atau bagikan gambar dari aplikasi lain → pemrosesan → review isian → peringatan duplikat jika ada → simpan. Share Extension hanya memasukkan gambar ke inbox privat; commit transaksi dilakukan di aplikasi utama setelah unlock.

Isian belum terbaca berlabel **Perlu diisi**, nominal ambigu berlabel **Periksa nominal**. Jangan memakai confidence sebagai alasan menyimpan otomatis. Bukti tidak mendukung bahasa/format tertentu tetap bisa diedit manual. Draf bisa disimpan atau dibatalkan; tidak masuk saldo/laporan sebelum dikonfirmasi.

### Split bill

Menu Rencana → Split bill → input manual atau pilih foto/screenshot/kamera. Gambar dari Share Sheet masuk inbox F02; setelah unlock pilih tujuan Transaksi atau Split bill. Jangan menyimpan transaksi biasa ketika tujuannya hanya membagi invoice.

Review menampilkan gambar sementara dan baris item yang bisa dikoreksi, kuantitas/total baris, subtotal, diskon, pajak, service dan total. Tampilkan **Periksa nominal**, **Item belum dibagi**, atau **Total belum cocok** bila perlu. Jalur manual tetap tersedia saat OCR gagal/izin ditolak.

Tentukan peserta, siapa yang membayar, dan pembagian rata/custom/per item. Item bersama dapat dipilih untuk beberapa peserta. Tampilkan dasar pembagian penyesuaian, aksi koreksi, selisih dan hasil pembulatan. Jangan mengaktifkan konfirmasi hasil final sebelum jumlah seluruh bagian cocok dengan total.

```text
Split bill                         [Edit invoice]
Pembayar                           Teman
Pembagian                          Per item
Peserta dan bagian masing-masing
Bagian saya                        Rp…
Perlu saya ganti ke teman           Rp…
[Simpan hitungan]
Tidak mengubah saldo atau utang
[Catat ke keuangan…]                opsional
```

**Simpan hitungan** hanya menyimpan perhitungan lokal; tidak meminta dompet atau memaksa membuat orang pada modul utang. **Catat ke keuangan…** membuka preview terpisah: Saya membayar / Teman membayar. Jika teman membayar, pilih **Sudah saya ganti** atau **Bayar nanti**. Pilih dompet aset bila ada uang keluar, kategori bagian pribadi, tanggal dan orang terkait; tampilkan perubahan saldo, expense pribadi, serta utang/piutang sebelum konfirmasi.

Jika teman membayar lalu diganti, pengeluaran pribadi hanya dicatat sekali. Pembayaran sebagian memakai riwayat utang yang sudah ada. Sesi yang telah dibukukan menampilkan tautan transaksi/utang dan tidak menawarkan konfirmasi pembukuan kedua. Koreksi diarahkan ke pembatalan pembukuan; jelaskan pembayaran mana yang harus dibatalkan dahulu.

Nama peserta tetap lokal, tidak membutuhkan kontak/login, dan nama sama tidak otomatis digabung. Detail perilaku dan angka acceptance ada di [desain split bill](docs/superpowers/specs/2026-11-20-split-bill-design.md).


### Dompet

Daftar memisahkan aset dan kartu kredit. Detail menampilkan saldo berdasarkan catatan, tanggal saldo awal, dan riwayat. Arsip menyembunyikan dari pemilih baru tanpa menghilangkan histori. Dompet dengan tagihan/alokasi aktif harus dipindahkan referensinya atau dinonaktifkan dependensinya sebelum arsip. Edit saldo awal memiliki penjelasan dampak dan konfirmasi.

Kartu dengan saldo internal negatif menampilkan **Saldo kredit Rp…**, bukan angka tagihan negatif; Ringkasan menunjukkan total saldo kredit kartu terpisah dari kas dan kewajiban. Tombol pelunasan tidak aktif bila tidak ada kewajiban positif.

### Utang & piutang

Daftar orang menampilkan total **Utang saya** dan **Piutang saya** terpisah. Detail orang berisi beberapa catatan dan total sisa. Form memilih **Saya berutang / Orang ini berutang ke saya**, nama orang, **Transaksi baru / Catatan lama**, nominal/sisa pokok, tanggal, jatuh tempo opsional, dan catatan.

```text
Andi
Piutang tersisa Rp300.000

Pinjaman makan              Belum lunas
Pokok Rp500.000 · Dibayar Rp200.000
Sisa Rp300.000
[Catat pembayaran]

Riwayat pembayaran
18 November 2026  Rp200.000 → BCA
```

Pembayaran meminta pokok, dompet dan admin opsional. Preview menjelaskan perubahan saldo dan sisa utang. Catatan lama menampilkan teks **Tidak mengubah saldo dompet**. Nama orang tidak diambil dari kontak dan tidak mengirim pesan kepadanya.

### Anggaran dan ringkasan

Anggaran keseluruhan tampil di atas daftar kategori. Kategori tidak ditambahkan lagi ke total. Progres berisi angka dan label **Lewat Rp…**, tidak hanya warna merah. Ringkasan periode dapat dibuka ke daftar transaksi dengan filter aktif. Rentang tanggal inklusif di UI; komputasi memakai akhir eksklusif hari setelah tanggal akhir.

### Tagihan dan target

Tagihan dipisahkan menjadi akan datang, jatuh tempo, dan dibayar. **Catat pembayaran** membuka form pengeluaran terisi; pengingat bukan bukti pembayaran. Target menunjukkan dialokasikan/target, dompet pendukung, dan peringatan bila saldo tidak cukup menopang alokasi. **Alokasikan** dan **Tarik alokasi** tidak membuat pendapatan/pengeluaran.

### Pencarian dan pengaturan

Pencarian menyediakan kolom teks dan filter periode/dompet/kategori/jenis; chip filter bisa dihapus. Detail utang dapat dijangkau dari hasil. Pengaturan menyediakan kategori, keamanan, CSV, buat/pulihkan cadangan dan informasi data lokal. Export memberi preview cakupan; restore menampilkan versi, waktu, jumlah data dan peringatan penggantian seluruh data, lalu autentikasi dan konfirmasi.

## 3. Sistem visual usulan

| Bagian | Aturan |
| --- | --- |
| Warna | Gunakan semantic system background/label/separator; tint system teal; status system red/orange/green dengan label atau ikon. |
| Tipografi | SF melalui Dynamic Type: largeTitle untuk angka utama, headline untuk bagian, body untuk form, caption untuk metadata; nominal memakai monospaced digits. |
| Spasi | Skala 4, 8, 12, 16, 24, 32 pt; margin layar 16 pt, mengikuti safe area. |
| Komponen | Native List, Form, navigation stack, sheet, date picker, search, confirmation dialog; SF Symbols berlabel aksesibilitas. |
| Sentuhan | Minimum 44 x 44 pt; aksi destruktif dipisahkan dari simpan dan diberi konfirmasi dampak. |
| Gerak | Transisi sistem; hormati Reduce Motion; tidak ada animasi saldo yang menghalangi pembacaan angka. |
| Tampilan | Light/dark mengikuti sistem; informasi tidak bergantung pada warna; jangan memotong nominal besar. |

## 4. Keadaan yang wajib dirancang

| Keadaan | Perilaku |
| --- | --- |
| Belum ada data | Jelaskan kegunaan dan satu aksi utama, misalnya Buat dompet. |
| Tidak ada hasil filter | Tampilkan filter aktif dan Reset filter, bukan menyatakan semua data kosong. |
| Sedang menyimpan | Cegah double submit, input tetap dipertahankan. |
| Penyimpanan gagal/disk penuh | Pesan jelas, tombol coba lagi, tidak mengubah saldo sebagian. |
| Izin ditolak | Fitur manual tetap jalan; tautan Pengaturan jika diperlukan. |
| App terkunci/background | Konten keuangan tertutup; Share Extension tidak menampilkan data akun. |
| Backup rusak/key salah | Tidak mengganti store aktif; tawarkan pilih file lain. |
| Data panjang | Nama 120 karakter, catatan 2.000 karakter, angka maksimum PRD; wrap dan layout adaptif. |

## 5. Bukti desain yang diperlukan sebelum rilis

Screenshot terang/gelap untuk Ringkasan, form transfer+admin, utang lama, pembayaran sebagian, OCR ambigu, split invoice/item bersama/hitung saja/preview kedua arah pembayar, dan restore. Periksa pada iPhone terkecil yang didukung dan ukuran besar, Dynamic Type aksesibilitas, VoiceOver, serta keyboard terbuka. Belum ada bukti visual pada tahap dokumentasi ini.
