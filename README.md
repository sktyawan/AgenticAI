# AgenticAI

Sistem korporasi yang dijalankan agent AI — **"Otonom"**.

## Status

**Kerangka awal** (27 Oktober 2026). Riset 24 jam atas blueprint berjalan 27–28 Oktober 2026;
struktur final menyusul hasil riset tersebut. Dokumen ini bukan klaim bahwa sistem sudah
dibangun atau sudah lulus audit apa pun.

## Acuan

Blueprint: [sktyawan/blueprint-sistem-korporasi](https://github.com/sktyawan/blueprint-sistem-korporasi)
(v1.2 konsolidasi — rancangan, belum disahkan). Satu dashboard, 12 peran pemeriksa,
10 alur utama (F01–F10), gerbang manusia, dan pemisahan pelaksana / pemeriksa / auditor.

## Kemampuan inti

- **CRUD bergating** untuk semua objek bisnis (lihat `docs/CRUD.md`): create, read, update,
  dan delete selalu lewat autentikasi, isolasi perusahaan, dan jejak audit. Bukan CRUD mentah.
- **Skill bank**: katalog skill berversi (pemeriksa regulasi, pelaksana F01–F10, jenjang,
  dan 3 skill dashboard: SK-DASH-001 UI builder, SK-DASH-002 bug control, SK-SEC-002 firewall defence).
- **Memori dan soul berversi**: memori adalah data (bukan instruksi/izin); soul membentuk cara
  bekerja (bukan wewenang).
- **API `/api/v1`** dengan kelas principal terpisah: human, agent, reviewer/auditor, executor.
- **Dashboard web** bergaya Antigravity: dark premium, sidebar, command palette, panel AI Copilot,
  antrean approval, dan timeline audit.

## Keamanan

Lihat [`SECURITY.md`](SECURITY.md). Ringkasnya: gerbang manusia untuk tindakan mengikat,
dana, dan pengajuan resmi; enforcement di server (bukan di prompt); audit append-only;
tidak ada secret di repo ini.

## Struktur

Lihat [`docs/STRUCTURE.md`](docs/STRUCTURE.md). Stack teknologi belum ditetapkan —
keputusan riset 28 Oktober 2026.

## Lisensi

Belum ditetapkan (keputusan terbuka). Kontribusi belum dibuka sampai struktur final disahkan.
