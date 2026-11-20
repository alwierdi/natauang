# Desain teknis NataUang v1

Status: baseline teknis masih usulan untuk ditinjau; penambahan F13 disetujui untuk dokumentasi, bukan persetujuan seluruh baseline atau izin implementasi. Tanggal: 20 November 2026.

## 1. Intent, cakupan, dan pendekatan

Bangun seluruh [F01–F13 PRD](../../PRD.md), dengan [UX](../../../DESIGN.md) sebagai acuan. Salah saldo, catatan ganda, restore kehilangan data, dan klaim akses notifikasi yang tidak terbukti adalah hasil yang harus dihindari. Kontrak tambahan F13 ada di [desain split bill](2026-11-20-split-bill-design.md), termasuk keputusan pengguna, pembagian invoice dan pembukuan opsional.

| Pendekatan | Keuntungan | Trade-off |
| --- | --- | --- |
| Native SwiftUI + SwiftData, lokal | Cocok untuk iPhone, API sistem, tanpa operasi server | Membutuhkan Mac untuk build; Android memerlukan implementasi lain |
| Flutter/React Native, lokal | Jalur Android terbuka | Integrasi iOS OCR, share, dan keamanan tetap perlu native; belum ada kebutuhan Android |
| Native dengan backend sejak awal | Bisa menyiapkan sinkronisasi | Akun, keamanan server, konflik sinkronisasi dan biaya belum dibutuhkan |

**Rekomendasi perencana:** opsi native lokal. Pengguna belum memilih stack; catat sebagai D01 dan validasi sebelum menulis kode. Penulisan dokumen ini memenuhi permintaan perencanaan, bukan menganggap pilihan teknis sudah disetujui.

## 2. Global constraints

- Platform: iPhone, minimum iOS 17.
- Bahasa UI: Indonesia; mata uang: IDR; nominal: Int64 rupiah tanpa pecahan.
- Penyimpanan: lokal, tanpa akun/server dan tanpa unggahan gambar.
- Draf OCR tidak mengubah saldo sebelum konfirmasi pengguna.
- Seluruh perubahan transaksi terkait harus atomik dan idempoten.
- Split bill hitung saja tidak membuat transaksi atau utang; pembukuan eksplisit hanya menghitung bagian pribadi sebagai pengeluaran.
- Build, XCTest, simulator, dan validasi perangkat iOS memerlukan Mac/Xcode.

Usulan stack: SwiftUI, SwiftData, Foundation, Vision, PhotosUI, UserNotifications, LocalAuthentication, CryptoKit, XCTest. Gunakan SDK Xcode yang stabil pada saat S0, catat versi eksaknya dalam README, CI dan bukti uji; jangan mengklaim build di Windows. Tidak perlu dependency pihak ketiga untuk v1.

## 3. Struktur dan batas tanggung jawab

```text
NataUang/App/                 entry, navigasi, dependency composition
NataUang/Core/Domain/         nilai, model domain, aturan uang murni
NataUang/Core/Persistence/    model SwiftData, mapper, transaksi store, migrasi
NataUang/Features/            Wallets, Transactions, Capture, Categories,
                             Budgets, Bills, Reports, Savings, Debts,
                             Search, Settings, SplitBills
NataUang/Services/            OCR, reminders, authentication, export, backup
NataUangShare/                penerimaan gambar ke inbox App Group
NataUangTests/                XCTest unit dan integration
NataUangUITests/              XCTest alur dan aksesibilitas
```

View memanggil service/use case. Aturan uang tidak berada di view atau observer saldo. Satu FinanceService memvalidasi command, mengubah record dalam satu konteks dan satu save, lalu menghasilkan snapshot; bila gagal, rollback. Saldo dan laporan diturunkan dari record, bukan angka cache yang diperbarui terpisah.

## 4. Model data

Seluruh ID UUID; timestamp audit UTC; tanggal transaksi memakai LocalDay(year, month, day, timeZoneID). Model domain value type dipisah dari @Model persistence agar aturan bisa diuji tanpa UI. Store memiliki schemaVersion dan migrasi eksplisit.

| Entitas | Field utama dan aturan |
| --- | --- |
| Wallet | id, name, kind(bank/eWallet/cash/creditCard), openingAmount, openingDay, archived; saldo kartu bertanda: positif kewajiban, negatif saldo kredit |
| Category | id, name, kind(income/expense), archived, systemKey?; kategori admin sistem tidak dapat dihapus |
| Transaction | id, commandID unik, kind(income/expense/transfer/creditPayment/refund/debtIssue/debtPayment/splitExpense), amount, fee, sourceWalletID?, destinationWalletID?, categoryID?, merchant?, note?, day, relatedDebtID?, billOccurrenceID?, refundOfID?, captureHash?, relatedSplitBillID? |
| Person | id, name; nama bukan unique key |
| Debt | id, personID, direction(iOwe/owedToMe), origin(new/legacy/splitBill), principal, openingDay, dueDay?, note?, relatedSplitBillID?, relatedSplitParticipantID?; transaksi pencairan baru menunjuk id ini, origin split tidak membuat debtIssue |
| DebtPayment | id, debtID, transactionID; nilai/tanggal/dompet berasal dari Transaction agar tidak punya dua sumber nominal |
| MerchantRule | normalizedMerchant unik, categoryID; aturan disimpan setelah konfirmasi/koreksi pengguna |
| CaptureDraft | id, proposedFields, unresolvedFields, imageHash?, createdAt; tidak membuat transaksi sampai commit |
| Budget | id, month, categoryID?; amount; unik per bulan+scope; categoryID nil = keseluruhan |
| Bill | id, name, estimatedAmount, categoryID, preferredWalletID?, cadence, anchorDay, startDay, reminderEnabled, archived |
| BillOccurrence | id, billID, dueDay; unik billID+dueDay; paid diturunkan dari transaksi terhubung |
| SavingsGoal | id, name, targetAmount, targetDay?, archived |
| GoalAllocation | id, goalID, walletID, signedAmount, day; net allocation tidak negatif |
| AppSettings | schemaVersion, lockEnabled; rahasia tidak disimpan dalam model ini |
| SplitBill dan record pembagian | Perhitungan lokal, peserta, item, pembagian, penyesuaian dan metadata pembukuan; kontrak lengkap pada desain split bill |

Representasikan operasi ekonomi hanya sekali melalui Transaction; biaya merupakan field terkait, bukan record bebas. Fungsi posting menerjemahkan record menjadi perubahan saldo dan kontribusi laporan. Jangan menyimpan Debt.remaining atau Wallet.balance sebagai sumber kebenaran kedua. Snapshot dapat dihitung ulang; optimasi hanya setelah pengukuran.

### Posting per jenis

| Jenis | Posting aset/kewajiban | Kontribusi laporan |
| --- | --- | --- |
| income | aset tujuan +amount | income +amount |
| expense aset | aset sumber -(amount+fee) | expense kategori +amount; admin +fee |
| expense kartu | kewajiban sumber +(amount+fee) | expense kategori +amount; admin +fee |
| transfer aset | sumber -(amount+fee), tujuan +amount | admin +fee |
| creditPayment | aset sumber -(amount+fee), kewajiban tujuan -amount | admin +fee |
| refund aset/kartu | aset asal +amount / kewajiban asal -amount | expense kategori asal -amount |
| debtIssue iOwe | aset tujuan +(amount-fee) | admin +fee; fee harus < amount |
| debtIssue owedToMe | aset sumber -(amount+fee) | admin +fee |
| debtPayment iOwe | aset sumber -(amount+fee) | admin +fee |
| debtPayment owedToMe | aset tujuan +(amount-fee) | admin +fee; fee harus < amount |

Untuk penerimaan dengan fee, preview selalu menampilkan pokok, admin dan bersih diterima; admin nol default. Catatan lama hanya membuat Debt, tidak debtIssue. Koreksi transaksi merupakan replace record atomik dengan validasi seluruh dependensi/refund/pembayaran; delete menghapus posting turunan melalui perhitungan ulang. Menonaktifkan autosave pada konteks mutasi service mencegah commit setengah jalan.

Kartu dengan balance negatif didukung setelah refund/pembukaan saldo kredit. Tampilkan tagihan = max(balance,0) dan saldo kredit = max(-balance,0) secara terpisah; saldo kredit tidak menambah kas atau income. Pelunasan harus <= max(balance,0). Belanja berikutnya menambah balance sehingga memakai saldo kredit lebih dulu secara alami.

F13 menambah `splitExpense`, `relatedSplitBillID` pada Transaction dan `origin = splitBill` beserta relasi split pada Debt. Posting splitExpense mengikuti tabel desain split bill: nominal transaksi adalah bagian pribadi, uang keluar dapat sebesar invoice penuh atau bagian pribadi, dan mode utang tidak memposting dompet dahulu. Relasi split bukan alasan membuat debtIssue tambahan. Khusus splitExpense, amount nol sah ketika seluruh tagihan milik peserta lain. Semua pembukuan/pembatalan melalui SplitBillService dan boundary FinanceStore; generik FinanceService mengarahkan koreksi ke sesi asal.

Untuk split yang langsung diganti ke teman, expenseDay dan reimbursementDay boleh berbeda; laporan memakai tanggal pengeluaran, posting dompet memakai tanggal penggantian. Validasi openingDay mengikuti tanggal posting dompet, bukan menolak expenseDay tanpa arus uang. Ledger dan restore wajib memakai kontrak tanggal ini.


## 5. Antarmuka domain usulan

Semua tipe di Core/Domain kecuali implementasi service. Money = Int64 dengan checked arithmetic; rentang input di PRD. FinanceError: invalidAmount, overflow, invalidWallet, archivedReference, beforeOpeningDay, futureDay, exceedsOutstanding, referencedRecord, invalidRefund, duplicateCommand, storeFailure.

```swift
struct CommandID: Hashable, Codable { let rawValue: UUID }
struct LocalDay: Hashable, Codable { let year: Int; let month: Int; let day: Int; let timeZoneID: String }
struct DateRange { let start: LocalDay; let endExclusive: LocalDay }
// TransactionInput memuat field Transaction selain id/timestamp/commandID.
// DebtInput memuat personID, direction, origin, principal, openingDay, dueDay, note.
// DebtPaymentInput memuat debtID, walletID, amount, fee, day, note.
@MainActor protocol FinanceStore {
    func snapshot() throws -> FinanceSnapshot
    func perform(_ mutation: (inout FinanceSnapshot) throws -> Void) throws
}
@MainActor final class FinanceService {
    func create(_ input: TransactionInput, commandID: CommandID) throws -> UUID
    func update(id: UUID, input: TransactionInput, commandID: CommandID) throws
    func delete(id: UUID, commandID: CommandID) throws
    func openDebt(_ input: DebtInput, walletID: UUID?, fee: Int64, commandID: CommandID) throws -> UUID
    func recordPayment(_ input: DebtPaymentInput, commandID: CommandID) throws -> UUID
}
```

FinanceSnapshot memuat array seluruh entitas domain tabel data dan set processedCommandIDs. perform menjalankan mutasi pada salinan, validasi invariant, lalu persist diff dalam satu save; kegagalan tidak memublikasikan snapshot baru. Retry commandID yang sudah berhasil mengembalikan hasil pertama/no-op, bukan transaksi kedua; simpan commandID beserta resultID dalam CommandReceipt, termasuk edit/delete. Nilai baru untuk aksi baru, ID sama untuk retry aksi yang sama.

Domain tambahan: Ledger.balance(walletID:snapshot:) throws -> Int64; DebtRules.remaining(debtID:snapshot:) throws -> Int64; ReportEngine.summary(range:snapshot:) throws -> ReportSummary; semua fungsi murni. ReportSummary memuat income, expense, expenseByCategory; filter mencocokkan tanggal lokal, bukan timestamp UTC. FinanceSnapshot juga memuat seluruh record SplitBill; SplitBillEngine dan SplitBillService mengikuti kontrak F13 pada desain split bill.

## 6. Capture, pengingat, dan keamanan

- **OCR:** OCRService.recognize(imageData:) async throws -> [RecognizedLine]; ReceiptParser.parse(lines:) -> DraftProposal. Gunakan Vision VNRecognizeTextRequest dan deteksi bahasa yang tersedia di runtime; jangan mengklaim dukungan OCR Indonesia sebelum diuji. Parser format Rupiah harus membedakan 35.000 dari saldo/nomor rekening; ambiguous tidak terisi final otomatis.
- **Share:** Extension menulis satu gambar JPEG/PNG/HEIC <= 20 MB dan manifest atomik ke App Group inbox berproteksi; tidak membuka SwiftData. Main app mengonsumsi setelah unlock. TTL gambar 24 jam, pembersihan saat launch/resume dan sesudah konsumsi; tidak menjamin penghapusan saat aplikasi tidak berjalan. Batas pixel/decode diuji agar gambar besar tidak kehabisan memori.
- **Draf:** setelah OCR simpan isian dan hash tanpa teks mentah/OTP. Gambar sementara hanya bertahan selama ekstraksi/review, dihapus saat selesai/batal atau melalui TTL/pembersihan inbox bila ditinggalkan; tidak menjadi lampiran permanen. Hash gambar untuk indikasi duplikat saja; tidak menentukan identitas ekonomi secara mutlak.
- **Invoice split:** gunakan OCR yang sama, lalu SplitInvoiceParser untuk item/kuantitas/penyesuaian/total; koreksi wajib, total harus cocok. Gambar untuk hitung saja tidak dikirim ke konfirmasi transaksi. Aturan gambar sementara, batas/decode, hash dan unlock sama dengan capture biasa.
- **Pengingat:** ReminderService.reconcile(snapshot:today:) async menyusun paling banyak 32 pengingat terdekat untuk tagihan/utang, membatalkan jadwal usang, dan menjadwalkan ulang saat launch/resume/perubahan. Tidak menjanjikan semua pengingat masa depan tetap aktif jika aplikasi lama tidak dibuka. Teks notifikasi default generik tanpa nama/nominal.
- **App lock:** LocalAuthentication deviceOwnerAuthentication, fallback kode perangkat. Lock setiap masuk background, tutup snapshot app switcher. Ekspor, backup dan restore memerlukan autentikasi ketika lock aktif; app yang terkunci tidak memproses isi inbox. Bila perangkat belum punya passcode, jelaskan prasyarat dan jangan mengaku lock sudah aktif.
- **File lokal:** Data Protection pada store/inbox/file sementara; Keychain untuk rahasia lokal yang diperlukan. Jangan mengklaim Face ID mengenkripsi seluruh database sendiri.
- **Ekspor:** CSV transaksi dengan kolom tetap dan ID relasi, UTF-8, ISO date, integer nominal, escaping delimiter/newline serta proteksi formula injection untuk teks berawalan =,+,-,@. Sertakan related_split_bill_id, wallet_posting_day dan wallet_debit_idr agar bagian pribadi tidak disalahartikan sebagai seluruh uang keluar; header dan semantik di S7-02. CSV hanya untuk dibaca, bukan restore.
- **Backup:** file .natauangbak: header version=1, cipher=AES-256-GCM, nonce, ciphertext dan tag; header diautentikasi sebagai AAD. Payload JSON lengkap semua entitas, termasuk sesi split, peserta/item/pembagian, metadata pembukuan dan relasi transaksi/utang, schemaVersion, createdAt, jumlah record dan command receipts; tanpa gambar atau status lock perangkat. Schema awal v1 harus mencakup F13 karena belum ada aplikasi dirilis.
- **Recovery:** kunci acak 32 byte per backup, ditampilkan sebagai kode pemulihan base64 terpisah. Pengguna harus mengonfirmasi penyimpanan kode sebelum menyelesaikan export; jangan menaruh kode di file backup atau mengandalkan Keychain perangkat asal. Tidak ada fitur memulihkan kode yang hilang; jelaskan konsekuensinya sebelum export.
- **Restore:** batas file 50 MB, validasi versi/schema/rentang/UUID/relasi/invariant sebelum preview; key salah atau tag rusak gagal tanpa mutasi. Mode hanya replace, bukan merge. Setelah konfirmasi, ganti seluruh record dalam satu transaksi persistent store; bila save gagal rollback. Lock perangkat saat ini tetap, pengingat direkonsiliasi sesudah sukses, kegagalan pengingat tidak membatalkan data yang sudah berhasil dipulihkan.

## 7. Verifikasi dan risiko

Unit test memeriksa posting, debt remaining, tanggal, overflow, parsing dan aturan bisnis; integration test memeriksa rollback, persistence restart, idempotency, migrasi dan backup round-trip; UI/device test memeriksa izin, lock, Share Sheet dan keterbacaan. Nilai expected konkret ada di rencana implementasi.

Hambatan saat ini: belum ada Mac/Xcode terverifikasi, belum ada fixture bank pengguna, belum ada keputusan distribusi. Jalur manual tetap wajib bila OCR gagal. Jangan mengeksekusi pembayaran atau mengakses akun bank untuk memperoleh fixture; gunakan data sintetis yang formatnya realistis.

## 8. Keputusan builder dan non-goals

Builder boleh memilih nama view, pemecahan file kecil, query optimasi setelah profiling, dan patch SDK yang kompatibel. Mengubah minimum OS, model uang, cakupan F01–F13, semantics utang/admin/split, server/cloud, enkripsi backup, atau mengirim data keluar memerlukan perubahan dokumen dan persetujuan pengguna. Dokumen ini belum memerintahkan instalasi dependency atau scaffolding aplikasi.
