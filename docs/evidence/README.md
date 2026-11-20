# Format bukti pekerjaan

Buat file `<TASK-ID>.md` saat task dikerjakan dan tautkan dari sprint tracker. Bukti harus merujuk hasil yang benar-benar diamati. Jangan menyimpan data pribadi, recovery key, token, atau gambar rekening asli.

```markdown
# Bukti Sx-yy — Nama task

- Tanggal:
- Owner:
- Revision/commit (jika ada):
- Lingkungan: OS, Xcode, SDK, simulator/perangkat, versi iOS
- Cakupan requirement:

## Perubahan

Ringkas perilaku yang berubah dan file terkait.

## Verifikasi aktual

| Pemeriksaan/perintah | Expected | Actual | Hasil | Bukti |
| --- | --- | --- | --- | --- |
| Nama test/perintah yang dijalankan | Hasil yang diharapkan | Output/angka yang diamati | PASS/FAIL/NOT RUN | Path hasil/screenshot |

## Review dan acceptance

- Reviewer dan temuan:
- Penyelesaian temuan:
- Acceptance yang terpenuhi:
- Acceptance yang belum terpenuhi:

## Hambatan dan handoff

Langkah berikutnya. NOT RUN tidak sama dengan PASS; task belum DONE bila bukti wajib belum ada.
```

Untuk tugas dokumentasi, catat pemeriksaan link, ID cakupan, konsistensi dan review. Jangan menjalankan/mengarang test aplikasi untuk membuktikan perubahan Markdown.
