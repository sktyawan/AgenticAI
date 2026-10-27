# Struktur Repo (Kerangka Awal)

> Status: **kerangka awal** — bentuk direktori untuk diisi hasil riset 28 Oktober 2026.
> Modul di bawah ini belum diimplementasikan. Stack teknologi belum ditetapkan.

```
apps/
  dashboard-web/      # UI gaya Antigravity: sidebar, command palette, panel AI Copilot,
                      # antrean approval, timeline audit, kartu kasus
services/
  api/                # /api/v1: runtime agent, pemeriksa, auditor, gerbang manusia,
                      # eksekusi, register, memori, skill/soul
  agent-runner/       # eksekutor agent, lease tugas, heartbeat
  tool-gateway/       # daftar putih alat, batas biaya/waktu/egress
packages/
  skill-bank/         # katalog skill berversi (SK-REG-*, SK-F*, SK-DASH-001/002, SK-SEC-002, ...)
  policy/             # matriks gerbang, klasifikasi tindakan, aturan transisi
  crud/               # pola CRUD bergating (lihat docs/CRUD.md)
infra/                # CI/CD, branch protection, secret scanning (rencana)
docs/
  STRUCTURE.md        # dokumen ini
  CRUD.md             # konvensi CRUD bergating
  adr/                # architecture decision records
```

## Peta ke blueprint

| Blueprint | Repo ini |
| --- | --- |
| Dashboard & gerbang (Bag. 16, 11) | `apps/dashboard-web`, `services/api` (gerbang manusia) |
| Alur F01–F10 (Bag. 14) | `services/api` + `packages/policy` |
| Bank Skill (Bag. 7) | `packages/skill-bank` |
| Memori, soul, update skill (Bag. 8) | `services/api` (endpoint memori & skill/soul) |
| Kontrak API (Bag. 19) | `services/api` |
| Uji T01–T45 (Bag. 20) | suite uji per service (rencana) |

## Keputusan terbuka

Stack, penyedia identitas, format token layanan, database, dan pola deployment —
diputuskan dari hasil riset 28 Oktober 2026, bukan diasumsikan di sini.
