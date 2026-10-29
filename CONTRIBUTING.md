# Berkontribusi ke AgenticAI

Repo ini adalah struktur Otonom yang disetujui (lihat `docs/STRUCTURE.md`).
Kontribusi mengikuti gerbang yang sama dengan kode produksi, karena
repo ini adalah control plane: kegagalan gerbang di sini sama
berbahayanya dengan kegagalan server.

## Alur

1. Buat cabang dari `main` dengan pola nama `feature/…`, `fix/…`,
   atau `docs/…`.
2. Buka pull request ke `main`. Direct push ke `main` dilarang oleh
   ruleset (lihat `docs/STRUCTURE.md` §9).
3. PR wajib lolos CI (pemindaian secret, review dependensi, higiene
   repo) dan mendapat review dari pemilik direktori (`CODEOWNERS`).
   Direktori sensitif (`services/audit-signer/`, `services/vault/`,
   `services/identity/`, `packages/policy/`, `.github/workflows/`)
   wajib 2 reviewer.
4. Review yang sudah basi dibatalkan saat commit baru masuk; cabang
   harus mutakhir sebelum merge. Force push dan penghapusan `main`
   dilarang.

## Pesan commit

Singkat, imperatif, dan menyebut area yang diubah, misalnya
`ci: tambah pemindaian trivy mingguan`. Satu commit = satu perubahan
yang bisa ditinjau.

## Definisi "selesai"

- CI hijau (semua gerbang lolos).
- Review yang disyaratkan sudah didapat.
- Tidak ada secret, token, atau kredensial di commit mana pun
  (push protection memblokir push yang mengandung secret yang dikenali).
- Dokumen diperbarui bila perilaku berubah (termasuk ADR baru bila
  keputusan arsitektur berubah — lihat `docs/adr/`).

## Yang tidak boleh

- Menulis secret ke repo, log, atau pesan commit — bila secret
  telanjur masuk, prosedurnya rotasi/revokasi segera, bukan sekadar
  menghapus commit (riwayat git tetap menyimpan).
- Melewati gerbang dengan `--no-verify` atau bypass ruleset di luar
  akun otomasi bernama yang tercatat.
- Mengubah gerbang, approval, atau tampilan status audit lewat
  perubahan UI tanpa persetujuan sesuai matriks (lihat
  `docs/STRUCTURE.md` §6, SK-DASH-001/002, SK-SEC-002).
