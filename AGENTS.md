# Aturan agent — NataUang

## 1. Sumber keputusan

Baca [README](README.md), [PRD](docs/PRD.md), [desain UX](DESIGN.md), [desain teknis](docs/superpowers/specs/2026-11-20-natauang-design.md), [rencana](docs/superpowers/plans/2026-11-20-natauang.md), dan [tracker](docs/SPRINTS.md) sebelum implementasi. Instruksi terbaru pengguna mengungguli dokumen; rekonsiliasi dokumen bila berubah.

PRD menentukan perilaku, desain teknis menentukan kontrak data, DESIGN menentukan pengalaman, SPRINTS menentukan status. Jangan diam-diam memilih salah satu bila bertentangan; selesaikan konflik dan dokumentasikan keputusan. Pilihan berstatus usulan belum merupakan persetujuan pengguna. Permintaan membuat dokumen mengizinkan penyusunan/revisi dokumen, bukan otomatis memulai aplikasi.

## 2. Cara bekerja

- Terapkan Superpowers yang relevan untuk brainstorming, perencanaan, debugging dan verifikasi; terapkan Karpathy: nyatakan asumsi, gunakan solusi minimum, ubah hanya yang relevan, tentukan bukti sukses.
- Mulai dari task READY dengan dependensi selesai; satu task implementasi aktif per agent. Jangan melewati gate desain/lingkungan.
- Delegasi hanya bila diizinkan pengguna atau instruksi skill yang berlaku. Tetapkan pemilik file agar tidak ada penulisan bersamaan.
- Jangan menambah fitur, backend, library pihak ketiga, analytics, API berbayar, atau cloud upload sebagai inisiatif tersembunyi.
- Jika ada git, periksa status dan pertahankan perubahan pengguna. Jangan reset, clean, hapus massal, commit atau publish otomatis karena task dokumen ini. Commit kecil dapat dilakukan ketika workflow implementasi memang mengizinkannya.
- Jelaskan kemajuan dengan bahasa Indonesia yang mudah dipahami. Jangan menyatakan simulator, build atau pengujian iPhone lulus bila belum dijalankan.

## 3. Aturan implementasi

- Domain keuangan tidak berada di view. Gunakan Int64 rupiah dengan pemeriksaan overflow, UUID, tanggal bisnis lokal dan timestamp audit UTC.
- Saldo, sisa utang, dan kontribusi laporan diturunkan dari satu sumber data. Setiap perubahan transaksi/admin/pembayaran atomik dan idempoten.
- Jangan memperlakukan transfer sendiri, pokok pinjaman, pelunasan kartu atau alokasi tabungan sebagai penghasilan/belanja.
- Semua mutasi utang dan tagihan melewati service terkait. Record generik tidak boleh melewati validasi hubungan.
- Gambar bukti dan isi bank tidak diunggah, tidak dimasukkan log/fixture nyata. Tidak ada kredensial bank, private API, jailbreak atau klaim pembaca notifikasi bank tanpa bukti resmi.
- Gunakan kontrol native, Dynamic Type, VoiceOver, light/dark, target sentuh >=44 pt, dan label selain warna. Jangan membuat layout web untuk aplikasi native.

## 4. Pengujian dan bukti

- Untuk aturan uang/bug: tulis test perilaku, amati gagal karena alasan yang tepat, implementasikan perubahan minimum, lalu jalankan test sampai lulus.
- Test harus membuktikan hasil pengguna: saldo, laporan, relasi utang, rollback, restore, izin dan lock. Jangan menguji sekadar detail implementasi.
- Uji fokus terlebih dahulu; setelah lulus jalankan suite yang terdampak. Suite penuh dan UI/device check wajib pada gate sprint/rilis sesuai rencana.
- Perubahan dokumen saja cukup dicek link lokal, cakupan ID dan konsistensi status; tidak perlu test aplikasi palsu.
- Lingkungan Windows tidak membuktikan build iOS. Jika Mac/Xcode belum tersedia, tandai task terkait BLOCKED dengan alasan, lanjutkan pekerjaan independen, dan laporkan batasan secara tepat.
- Sebelum DONE, rekam perintah/lingkungan, hasil aktual, tanggal, dan lokasi bukti di docs/evidence/. Bukti tidak boleh berisi data keuangan pribadi.

## 5. Memperbarui progress

Sumber status tunggal: tabel task di [SPRINTS](docs/SPRINTS.md). Checklist rencana adalah panduan langkah; jangan menggunakannya sebagai tracker terpisah yang bertentangan.

1. Saat mulai, ubah status task ke IN_PROGRESS, isi owner dan tanggal update.
2. Jika menunggu dependency/keputusan, gunakan BLOCKED, alasan, dan tindakan pembuka hambatan; task yang belum dimulai tetap BACKLOG.
3. Setelah implementasi, gunakan REVIEW; kaitkan bukti serta commit/PR bila tersedia.
4. DONE hanya jika acceptance task, pengujian dan review terpenuhi. Jangan memasukkan seluruh sprint sebagai DONE hanya karena sebagian task selesai.
5. Catat scope change dan perubahan keputusan di log tracker. Jangan mengubah tanggal estimasi menjadi janji selesai.
6. Jika tugas dilanjutkan agent lain, tinggalkan handoff: task ID, perubahan, bukti, kendala, langkah selanjutnya.

## 6. Batas rilis

Jangan publish ke App Store/TestFlight, mengirim pesan ke pihak lain, atau memindahkan data ke layanan eksternal hanya berdasarkan rencana ini. Penyiapan dokumen dan pengujian lokal berada dalam scope; distribusi mengikuti instruksi pengguna berikutnya. Semua F01–F13 harus punya bukti sebelum rilis v1 dinyatakan lengkap.
