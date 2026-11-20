# Riset capture iPhone: S0-03

Tanggal pemeriksaan sumber: 20 November 2026. Sumber berikut mendeskripsikan kemampuan platform, bukan hasil implementasi atau uji perangkat NataUang.

| Area | Temuan dan dampak rancangan | Sumber Apple |
| --- | --- | --- |
| OCR | Vision mengenali teks di perangkat, mengembalikan kandidat, confidence dan posisi. Bahasa bergantung pada jalur dan revisi; periksa pada runtime, jangan menjanjikan akurasi atau dukungan bahasa tertentu. | [Recognizing Text in Images](https://developer.apple.com/documentation/vision/recognizing-text-in-images) |
| Pemilih foto | PhotosPicker memilih foto; mengambil data bisa gagal, termasuk ketika aset perlu diunduh dari iCloud tanpa jaringan. Sediakan input manual. | [Bringing Photos picker to your SwiftUI app](https://developer.apple.com/documentation/photokit/bringing-photos-picker-to-your-swiftui-app) |
| Kamera | Akses kamera memerlukan otorisasi dan `NSCameraUsageDescription`; minta saat kamera dipakai. Penolakan tidak menghalangi input manual. | [Requesting authorization to capture and save media](https://developer.apple.com/documentation/avfoundation/requesting-authorization-to-capture-and-save-media) |
| Share Sheet | Share Extension menerima attachment dari extension context; validasi tipe dan ukuran sebelum memproses. Dokumen panduan ini diarsipkan, jadi periksa SDK terkini saat implementasi. | [Share Extension guide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/Share.html) |
| App Group | App dan extension dapat berbagi container melalui App Group, yang memerlukan capability dan konfigurasi identitas/signing. Tidak memberi extension akses otomatis ke container privat aplikasi. | [Configuring app groups](https://developer.apple.com/documentation/xcode/configuring-app-groups) |
| Notifikasi | UNUserNotificationCenter mengelola notifikasi aplikasi atau extension sendiri. Dokumentasi ini tidak menetapkan kemampuan membaca notifikasi bank/aplikasi lain; fitur itu bukan acceptance v1. | [UNUserNotificationCenter](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter) |

## Kontrak fixture

[Kasus sintetis](../../NataUangTests/Fixtures/receipts-cases.json) memakai teks rekaan, bukan gambar atau data rekening pengguna. `lines` adalah hasil OCR sintetis, bukan bukti akurasi Vision. `expected` berisi usulan yang harus diperiksa pengguna; `unresolved` adalah hal yang tidak boleh diputuskan diam-diam. Nilai nominal dalam rupiah integer. Split invoice menggunakan baris total item, bukan harga satuan yang dikalikan dua kali.

Fixture bukan parser atau test yang telah berjalan. S6-01/S6-04 harus mengadaptasi fixture ke kontrak implementasi dan menguji gambar sintetis pada runtime nyata. Duplicate hash pada fixture adalah simbol sintetis; peringatan duplikat boleh dilewati, bukan larangan pembukuan dua transaksi berbeda.

## Batas dan tindak lanjut

Tidak ada Mac/Xcode, perangkat, OCR gambar, izin, Share Sheet atau App Group yang diuji dalam task ini. Pengambilan gambar, masa hidup file, proteksi data, batas ukuran/pixel, penggunaan memori dan bahasa Vision tetap memerlukan pengujian implementasi. Jangan merekam teks OCR mentah, OTP, bukti atau nominal pengguna dalam log.
