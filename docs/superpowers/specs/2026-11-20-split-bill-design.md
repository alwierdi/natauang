# Desain split bill NataUang v1

Tanggal: 20 November 2026. F13 disetujui pengguna untuk masuk dokumen dan target v1. Dokumen ini merinci keputusan tersebut, bukan izin memulai aplikasi atau pengesahan D01–D05. Acuan: [PRD](../../PRD.md), [UX](../../../DESIGN.md), [desain teknis utama](2026-11-20-natauang-design.md), dan [rencana implementasi](../plans/2026-11-20-natauang.md).

## 1. Tujuan dan pendekatan

Bantu menghitung bagian tagihan bersama, termasuk ketika teman membayar dahulu dan pemilik aplikasi hanya ingin mengetahui nominal penggantian. Hasil perhitungan tidak wajib menjadi transaksi atau utang.

| Pendekatan | Manfaat | Trade-off dan keputusan |
| --- | --- | --- |
| Kalkulator terpisah tanpa pembukuan | Tidak mengubah keuangan tanpa sengaja | Tidak memenuhi kebutuhan pencatatan opsional |
| Perhitungan mandiri dengan pembukuan eksplisit | Bisa menghitung saja atau mencatat saat diperlukan | Perlu relasi atomik agar pengeluaran tidak terhitung dua kali; dipilih dalam diskusi pengguna |
| Grup online dengan transfer uang | Kolaborasi antarperangkat | Akun, server dan pembayaran nyata di luar scope |

Keputusan pengguna: pembagian rata/custom/per item, item bersama, foto atau screenshot invoice dengan pemeriksaan, pembagian pajak/service/diskon yang bisa dikoreksi, total pembulatan tepat, serta pembukuan opsional. Bukan transfer uang nyata, pengiriman pesan, atau permintaan bayar otomatis.

Keputusan builder untuk v1: satu pembayar asal per tagihan, yaitu diri sendiri atau satu teman. Tidak ada beberapa pembayar asal, netting lintas tagihan, akun peserta, atau sinkronisasi grup. Pembayaran penggantian tetap dapat dicatat sebagian melalui service utang. Peserta dapat memakai nama lokal yang sama tanpa dianggap orang yang sama.

## 2. Alur dan pilihan pencatatan

Mulai dari menu Split bill pada Rencana atau aksi dari Baca bukti. Pilih input manual atau gambar, koreksi invoice, tentukan peserta termasuk diri sendiri, tetapkan pembayar, lalu bagi tagihan. **Hitung saja** adalah default dan menyimpan hanya perhitungan lokal, bukan Transaction, Debt, saldo atau kontribusi laporan.

Setelah memeriksa hasil, pembukuan memerlukan aksi terpisah, preview dan konfirmasi:

- **Saya yang membayar:** pilih dompet aset; catat uang keluar sebesar seluruh invoice, pengeluaran hanya bagian sendiri, dan piutang per peserta lain yang bagiannya positif.
- **Teman yang membayar, sudah saya ganti:** pilih dompet aset dan tanggal penggantian; catat uang keluar dan pengeluaran sebesar bagian sendiri, tanpa utang terbuka.
- **Teman yang membayar, bayar nanti:** catat pengeluaran bagian sendiri dan utang kepada pembayar, tanpa perubahan saldo saat itu. Saat membayar, saldo turun dan utang berkurang, tanpa pengeluaran kedua.

Pilihan sudah mengganti berarti seluruh bagian sendiri telah dibayar. Penggantian sebagian memakai bayar nanti lalu Catat pembayaran. Aplikasi tidak menebak dompet, kategori atau status pembayaran dari invoice. Tanggal pengakuan pengeluaran dipilih saat pembukuan, default tanggal invoice yang telah diperiksa; tanggal penggantian boleh berbeda dan tidak mendahului tanggal tersebut. Tanggal posting dompet mengikuti batas tanggal saldo awal dan larangan transaksi masa depan. Invoice dapat dihitung tanpa tanggal untuk mode hitung saja.

Kategori pengeluaran wajib ketika bagian sendiri positif; seluruh bagian sendiri memakai satu kategori yang dipilih. Pajak/service invoice termasuk bagian pengeluaran, bukan biaya admin bank kedua. Pembukuan split bill mengikuti dompet aset yang didukung utang personal; mode hitung saja tidak membatasi cara pembayaran di dunia nyata.

## 3. Aturan pembagian uang

Gunakan full-width intermediate atau algoritme hasil-bagi/sisa setara untuk perkalian proporsi; jangan menolak invoice valid hanya karena produk antara melampaui Int64. Periksa batas pada hasil akhir dan total. Pada mode rata/custom, item tidak wajib: input total manual sudah cukup, sedangkan rincian penyesuaian hanya wajib cocok jika diisi.

Gunakan Int64 rupiah dan checked arithmetic, termasuk perkalian untuk proporsi. Nominal mengikuti maksimum input PRD. Jangan memakai Double untuk pembagian uang. Harga baris menyimpan total baris, bukan harga satuan yang dikalikan kembali; kuantitas adalah informasi dari invoice yang bisa diperiksa.

- Mode rata membagi total akhir ke seluruh peserta yang dipilih.
- Mode custom mengisi nominal akhir per peserta; jumlah harus tepat sama dengan total invoice. Jangan menambahkan pajak/service/diskon lagi.
- Mode item menetapkan seluruh item ke peserta; satu item dapat dibagi rata atau memakai bobot bilangan bulat positif untuk beberapa peserta. Semua item wajib mendapat pemilik.
- Total akhir = subtotal baris + pajak + service charge - diskon. Semua komponennya nonnegatif dan total akhir harus positif. Diskon tidak boleh melebihi subtotal; total invoice yang terbaca harus cocok dengan rincian sebelum konfirmasi.
- Mode item membagi diskon proporsional terhadap subtotal peserta, kemudian pajak dan service proporsional terhadap subtotal setelah diskon. Jika diskon menghabiskan subtotal, gunakan subtotal sebelum diskon sebagai bobot pajak/service. Subtotal item nol dengan penyesuaian positif harus dikoreksi atau dialihkan secara eksplisit ke mode rata/custom.
- Pembagian integer memakai hasil bagi bawah lalu sisa rupiah ke pecahan terbesar; nilai pecahan sama mengikuti urutan peserta tersimpan. Berlaku pada item bersama, setiap penyesuaian dan pembagian rata; hasil deterministik dan jumlah tepat sama dengan total.
- Koreksi pembagian penyesuaian diperbolehkan dengan nominal per peserta; jumlah tiap komponen wajib cocok dan hasil akhir peserta tidak negatif. Selisih harus terlihat; jangan diam-diam mengubah total atau menambah peserta penampung.

Contoh acceptance, seluruh nominal dalam rupiah:

| Kasus | Hasil |
| --- | --- |
| Total 100000 dibagi rata tiga orang berurutan | 33334, 33333, 33333; total 100000 |
| Item A 60000 untuk diri sendiri, item B 40000 untuk teman, diskon 10000, pajak 9000, service 4500 | Dasar setelah diskon 54000/36000; akhir 62100/41400; total 103500 |
| Item bersama 30000 dibagi dua | Masing-masing 15000 sebelum penyesuaian |
| Nominal custom atau hasil OCR berbeda dari total | Konfirmasi diblokir sampai dikoreksi; input tetap tersedia |

## 4. Baca gambar invoice dan privasi

Gunakan OCR lokal F02 untuk membaca baris item, kuantitas, harga, subtotal, diskon, pajak, service charge dan total. Ini perlu parser invoice berbaris, bukan menganggap parser total transaksi sudah memadai. Jangan menjanjikan seluruh format/bahasa terbaca benar sebelum pengujian.

- Sumber: picker foto/screenshot, kamera dan gambar dari Share Sheet melalui inbox F02 setelah unlock.
- Pilih tujuan **Transaksi** atau **Split bill** sebelum konfirmasi; membaca gambar untuk split tidak boleh memanggil konfirmasi transaksi biasa.
- Tampilkan gambar sementara berdampingan atau bergantian dengan hasil koreksi. Angka ambigu, item belum terbaca dan total berbeda diberi label yang jelas. Pengguna harus memeriksa sebelum menghitung hasil final.
- Input manual selalu tersedia, termasuk ketika izin ditolak atau OCR gagal. Menghitung dan membuang draf tidak mengubah saldo.
- Proses pada perangkat; kata upload dalam percakapan berarti memilih gambar, bukan mengirim ke server. Pakai batas format, ukuran 20 MB, decode dan TTL inbox yang sama dengan F02.
- Hapus gambar setelah ekstraksi/review selesai atau dibatalkan; jika sesi ditinggalkan, gunakan TTL dan pembersihan launch/resume yang sama dengan F02. Simpan hanya isian terstruktur yang telah dikoreksi dan hash opsional, tanpa gambar permanen, teks mentah atau OTP.
- Hash yang sama menampilkan peringatan, bukan larangan menghitung ulang. Duplikasi pembukuan sesi yang sama tetap dicegah melalui relasi dan command receipt.

## 5. Model dan kontrak teknis

Tambahkan value type domain dan model persistence berikut ke snapshot dan backup. Semua ID UUID dan timestamp audit UTC mengikuti desain utama.

| Entitas | Isi dan invariant |
| --- | --- |
| SplitBill | id, title, invoiceDay?, timeZoneID, payerParticipantID, mode(equal/custom/items), subtotal, tax, serviceCharge, discount, total, status(calculation/booked), captureHash?, bookingTransactionID?, createdAt, updatedAt |
| SplitParticipant | id, splitBillID, displayName, isMe, order, personID?; tepat satu isMe, minimal dua peserta, pembayar termasuk peserta; personID hanya diperlukan untuk hubungan utang/piutang |
| SplitItem | id, splitBillID, label, quantityLabel?, lineAmount; nominal total baris nonnegatif, bukan harga satuan |
| SplitItemShare | id, itemID, participantID, weight; peserta harus dari tagihan yang sama |
| SplitAdjustmentShare | id, splitBillID, participantID, component(tax/serviceCharge/discount), amount; koreksi opsional per komponen, bukan tambahan ke total |
| SplitCustomShare | id, splitBillID, participantID, amount; hanya mode custom |

Hasil `SplitResult` dihitung dari record: total dan daftar bagian akhir per participantID. Jangan menyimpan saldo atau sisa utang kedua. Menandai hitung saja selesai hanya memperbarui record split, bukan record ekonomi.

Kontrak di `Core/Domain/SplitBillEngine.swift` dan `Features/SplitBills/SplitBillService.swift`:

```swift
struct SplitResult { let total: Int64; let shares: [UUID: Int64] }
enum SplitBookingMode { case paidByMe, reimbursed, owePayer }
struct SplitBookingInput {
    let mode: SplitBookingMode
    let expenseDay: LocalDay
    let reimbursementDay: LocalDay?
    let walletID: UUID?
    let categoryID: UUID?
    let personIDs: [UUID: UUID] // participantID -> Person.id
}
// SplitBillInput berisi identitas/tagihan, peserta, item dan pembagian tabel di atas.
// SplitInvoiceProposal berisi kandidat baris/total dan unresolvedFields.
```

Antarmuka yang dihasilkan:

- `SplitBillEngine.calculate(_ input: SplitBillInput) throws -> SplitResult`: validasi total, kepemilikan item, pembulatan dan overflow.
- `SplitInvoiceParser.parse(lines: [RecognizedLine]) -> SplitInvoiceProposal`: tidak membuat record keuangan.
- `SplitBillService.save(id: UUID?, input: SplitBillInput) throws -> UUID`: simpan perhitungan lokal melalui FinanceStore.perform; hanya status calculation.
- `SplitBillService.book(id: UUID, input: SplitBookingInput, commandID: CommandID) throws -> UUID`: validasi ulang hasil dan buat semua relasi dalam satu commit; hasil UUID transaksi pembukuan.
- `SplitBillService.unbook(id: UUID, commandID: CommandID) throws`: balikkan pembukuan secara atomik hanya setelah pembayaran terkait dibatalkan; sesi kembali calculation.
- `SplitBillService.delete(id: UUID) throws`: hanya untuk status calculation; sesi booked harus dibatalkan pembukuannya terlebih dahulu.

Draf koreksi invoice memakai state form sampai valid. Save/hitung final menolak input yang belum lengkap; kegagalan tidak menghapus input. Perubahan sesi booked diarahkan ke pembatalan pembukuan dahulu, bukan mengubah jumlah piutang di belakang riwayat pembayaran.

### Posting pembukuan tanpa penggandaan

Tambahkan `Transaction.kind = splitExpense`, `relatedSplitBillID?`, dan `Debt.origin = splitBill` beserta `relatedSplitBillID?` serta `relatedSplitParticipantID?`. Debt split mengikat bagian satu peserta: peserta lain untuk piutang paidByMe, peserta diri sendiri untuk utang owePayer; personID utang menunjuk pembayar. Satu sesi booked memiliki tepat satu splitExpense dan referensi balik bookingTransactionID. Nilai `Transaction.amount` adalah bagian sendiri, `fee = 0`; khusus jenis ini amount boleh nol jika pemilik menalangi seluruh bagian orang lain. Total invoice tetap positif.

| Pilihan | Posting splitExpense | Relasi utang/piutang | Laporan |
| --- | --- | --- | --- |
| paidByMe | Aset sumber -total invoice pada expenseDay | Satu piutang per teman dengan bagian positif; tanpa debtIssue tambahan | Expense +bagian sendiri |
| reimbursed | Aset sumber -bagian sendiri pada reimbursementDay | Tidak membuat Debt | Expense +bagian sendiri pada expenseDay |
| owePayer | Tidak ada posting dompet saat pembukuan | Satu utang ke pembayar sebesar bagian sendiri, jika positif; tanpa debtIssue | Expense +bagian sendiri pada expenseDay |

Tanggal transaksi splitExpense merupakan expenseDay; khusus reimbursed, tanggal posting saldo memakai reimbursementDay dari data pembukuan yang disimpan bersama sesi. Simpan `bookingMode?`, `bookingWalletID?`, `bookingExpenseDay?`, `bookingReimbursementDay?` pada SplitBill saat booked; Ledger memeriksa tanggal posting dompet tersebut. Record tidak boleh memiliki dua sumber jumlah: posting memakai total invoice/bagian hasil engine dan memvalidasi amount sama dengan hasil.

Pembayaran utang split memakai recordPayment dan DebtPayment yang sudah ada. Pokok hanya memengaruhi saldo/sisa utang; admin pembayaran mengikuti aturan existing. Debt split menyimpan principal sama dengan bagian terkait; generik create/update/delete tidak boleh membuat atau mengubah relasi split, dan openDebt tidak boleh membuat origin splitBill secara bebas. Semua diarahkan ke SplitBillService dengan validasi FinanceService. Refund generik hanya menerima expense biasa; koreksi split memakai unbook setelah pembayaran dibatalkan. Menghapus sesi booked, peserta/Person yang direferensikan, atau transaksi sumber secara terpisah ditolak.

FinanceStore.perform menyimpan sesi, transaksi, semua Debt, Person baru yang dipilih untuk dibuat dan CommandReceipt dalam satu commit. Nama peserta tidak otomatis membuat Person; pemetaan atau pembuatan Person diminta hanya saat pembukuan membutuhkan hubungan utang. Gunakan ID Person yang sudah ada atau UUID baru dari input pemetaan beserta nama peserta; nama sama tidak menggabungkan orang. Retry commandID mengembalikan transactionID pertama, dan konfirmasi sesi booked dengan commandID baru ditolak agar tidak menggandakan uang.

## 6. Bukti yang harus tersedia

Pengujian domain harus diamati gagal sebelum implementasi, lalu lulus. Uji fokus, suite terdampak, review dan bukti aktual mengikuti aturan proyek.

- Hitung saja: buat/edit/hapus, OCR gagal, invoice duplikat dan koreksi tidak mengubah ledger/laporan/Debt.
- Engine: rata, custom, item bersama, koreksi penyesuaian, pembulatan deterministik, overflow, item belum dialokasikan, subtotal nol dan total berbeda.
- Saya bayar 300000, bagian sendiri 100000: aset -300000, expense 100000, piutang dua orang masing-masing 100000. Pembayaran keduanya memulihkan 200000 tanpa income.
- Teman bayar 300000, bagian sendiri 100000: owePayer menghasilkan expense 100000, utang 100000, aset tetap. Bayar 40000 lalu 60000 membuat utang lunas dan aset turun 100000, expense tetap 100000.
- Reimbursed pada bulan berikutnya: expense tetap pada expenseDay, saldo berkurang pada reimbursementDay, tanpa utang/income kedua; tanggal sebelum openingDay ditolak.
- Retry, double tap, save gagal di tengah pembuatan beberapa piutang, restart dan backup/restore mempertahankan total, relasi dan status; rollback meninggalkan snapshot awal identik.
- UI/device: review OCR ambigu, hitung saja, pemilihan item bersama, kedua arah pembayar, preview pembukuan, izin ditolak, light/dark, Dynamic Type dan VoiceOver. Belum ada hasil iOS yang diklaim.

## 7. Dampak dan handoff

Implementasi berada pada S6-04 (engine, kalkulator dan invoice OCR) serta S6-05 (pembukuan opsional). S7-03 harus memuat seluruh entitas split, relasi, status dan command receipts; S8 menguji F01–F13. Penambahan F13 tidak menyetujui D01–D05, tidak membuka gate Mac/Xcode, dan tidak memulai scaffolding.
