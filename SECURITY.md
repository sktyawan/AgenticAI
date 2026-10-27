# Security Policy

## Prinsip (dari blueprint Otonom)

- **Gerbang manusia**: tindakan mengikat, seluruh pengeluaran dana, pengajuan resmi
  (OSS, pajak, regulator, HKI), PHK, dan penghapusan data wajib persetujuan manusia berwenang.
- **Kelas principal**: human, agent, reviewer/auditor, executor. Kelas dibaca dari kredensial,
  bukan dari payload. Kredensial agent ditolak di endpoint gerbang manusia.
- **Isolasi perusahaan**: `company_id` diturunkan server dari identitas dan mandat.
  Nilai dari pemanggil hanya divalidasi, bukan dipercaya.
- **Konkurensi & idempotensi**: update memakai If-Match (revision); semua perintah yang
  mengubah keadaan wajib memakai Idempotency-Key.
- **Audit**: setiap permintaan dan mutasi menghasilkan AuditEvent (append-only + salinan
  independen). Tidak ada endpoint untuk mengubah atau menghapus event audit.
- **Alat & egress**: runner agent hanya memakai alat dan domain yang diizinkan (daftar putih),
  dengan batas biaya, waktu, dan jumlah upaya.

## Secret

Tidak ada secret, token, password, atau key di repo ini, dalam bentuk apa pun
(termasuk history git). Secret hanya lewat environment / secret manager.
Secret scanning dan branch protection diaktifkan saat CI siap (rencana).

## Melaporkan kerentanan

Jangan mempublikasikan detail eksploitasi atau data sensitif di issue publik.
Hubungi pemilik repo secara pribadi dengan deskripsi, dampak, dan langkah reproduksi.
