# Struktur Repo AgenticAI — Dokumen Final untuk Persetujuan Push

**Repo:** `sktyawan/AgenticAI` (publik, dibuat 27 Oktober 2026; saat ini baru kerangka awal, commit `a3b2121`)
**Status dokumen:** USULAN untuk disetujui Seti. Bukan keputusan yang sudah disahkan, bukan kode yang sudah ditulis.
**Aturan keras yang tetap berlaku:** TIDAK ADA push/commit ke GitHub mana pun sebelum Seti menyetujui struktur ini. Dokumen ini tidak mengubah blueprint final maupun konsolidasi v1.2. Tidak ada token/secret apa pun di dokumen ini.
**Sumber yang dirangkai:** BRIEF riset 24 jam; gelombang 1 (keamanan), 2 (regulasi primer), 3 (arsitektur agentic), 4 (UI Antigravity + struktur repo); empat file pembanding (framework orkestrasi, perusahaan-agent, control-plane/human-gate, keamanan-tata-kelola + 4 file bagian); dokumen konsolidasi blueprint v1.2; dokumen blueprint final 28 Oktober 2026 (98 halaman, §23 Changelog: 108 usulan diterapkan, 1 ditunda, 0 konflik).
**Yang TIDAK diulang di sini:** usulan gelombang 1–3 dan pembanding sudah diterapkan ke blueprint final dan tercatat di §23-nya — sintesis ini tidak mengulang penerapan itu. Bagian 10 dokumen ini hanya memuat usulan TAMBAHAN dari gelombang 4.

---

## 1. Keputusan bentuk repo: monorepo

**Dipilih (usulan): monorepo.** Alasan: tim awal kecil; perubahan lintas service (API ↔ policy ↔ skill-bank ↔ dashboard) adalah kasus normal di Otonom, bukan pengecualian — monorepo membuat perubahan itu atomik dalam satu PR; satu sumber kebenaran untuk tipe/kontrak API mencegah drift versi antar-repo.
**Biaya yang diakui dan penawarnya:** CI melambat tanpa caching → build terpengaruh saja (affected) + cache; kontrol akses per direktori lebih kasar daripada repo terpisah → CODEOWNERS per direktori sensitif, bukan pemecahan repo.
**Perkakas (usulan, diputuskan final lewat ADR-001/ADR-006):** pnpm workspaces + Turborepo bila lapisan JS/TS dipakai untuk dashboard/API; Nx dipertimbangkan bila stack akhir poliglot (service Python/Go di samping TS). Sampai stack diputuskan, tree di Bagian 2 ditulis agnostik bahasa — tidak mengunci runtime di nama direktori.

## 2. Tree direktori lengkap dan tujuan tiap bagian

Tree ini mempertahankan kerangka `a3b2121` (yang sudah ter-push: `apps/dashboard-web`, `services/api`, `services/agent-runner`, `services/tool-gateway`, `packages/skill-bank`, `packages/policy`, `packages/crud`, `infra/`, `docs/`) dan memperluasnya. Perubahan dari kerangka dicatat di akhir bagian ini.

```
sktyawan/AgenticAI/
├── apps/
│   └── dashboard-web/          # Control plane Otonom — UI gaya web Antigravity (Bagian 5).
│       ├── src/
│       │   ├── app/            # Rute: dashboard, kasus, approval, agent, audit, skill, pengaturan.
│       │   ├── components/     # sidebar, command-palette, copilot-panel, approval-card,
│       │   │                   # audit-timeline, agent-status-card, case-card, empty-state.
│       │   ├── tokens/         # Design token: primitif, semantik (dark/light), komponen.
│       │   └── lib/            # Klien API. Tidak pernah menyimpan secret; token di httpOnly cookie.
│       └── tests/              # Uji komponen + uji aksesibilitas (axe) + e2e alur approval.
├── services/
│   ├── api/                    # /api/v1 — satu-satunya pintu masuk HTTP (gateway aplikasi).
│   │   └── src/
│   │       ├── routes/         # kasus, approval, agent, audit, skill, memori, register.
│   │       ├── middleware/     # auth principal, tenant isolation, idempotency, If-Match.
│   │       └── domain/         # saga/orkestrasi, gerbang manusia, validasi intent beku.
│   ├── agent-runner/           # Eksekutor agent: klaim lease, heartbeat, reaper, saga worker.
│   │                           # Runtime agent TIDAK memegang kunci audit, secret jangka panjang,
│   │                           # atau jalur egress selain tool-gateway.
│   ├── tool-gateway/           # Satu-satunya jalur egress: allowlist alat/domain, validasi
│   │                           # argumen, pemindaian injeksi, audit per tool-call, budget.
│   ├── identity/               # Penerbit & verifikasi kredensial principal (human/agent/
│   │                           # reviewer/executor); token berumur pendek, sender-constrained.
│   │                           # Termasuk paspor agent per posisi (Bagian 4.3).
│   ├── audit-signer/           # SATU-SATUNYA pemegang kunci penandatangan AuditEvent.
│   │                           # Tidak pernah di-mount ke runtime agent (aturan SLSA-L3, Bagian 4.2).
│   └── vault/                  # Secret dinamis per tugas; rotasi; tidak pernah ke log/prompt/memori.
├── packages/
│   ├── skill-bank/             # Katalog skill berversi (Bagian 6): definisi + semver +
│   │                           # revocation + eval gate + penugasan. Cermin 1:1 dari blueprint §7/§7.5.
│   ├── policy/                 # Matriks gerbang, klasifikasi tindakan, aturan transisi,
│   │                           # register regulasi berversi (dengan tanggal "terakhir dicek" per entri).
│   ├── crud/                   # Pola CRUD bergating (docs/CRUD.md): Idempotency-Key, If-Match,
│   │                           # revisi berversi, soft-delete default.
│   ├── contracts/              # Skema API & tipe bersama — satu sumber kebenaran kontrak /api/v1.
│   └── ui-tokens/              # Token desain bersama (dipakai dashboard-web; siap dipakai
│                               # aplikasi lain bila kelak ada).
├── db/
│   ├── migrations/             # Migrasi berversi, hanya maju (tidak ada edit migrasi lama).
│   └── seeds/                  # Data contoh non-sensitif untuk dev. Tanpa data pribadi asli.
├── infra/
│   ├── ci/                     # Definisi pipeline (Bagian 8).
│   └── deploy/                 # Konfigurasi deployment per lingkungan. Tanpa secret.
├── .github/
│   ├── workflows/              # ci.yml, security.yml, release.yml, provenance.yml.
│   ├── CODEOWNERS              # Pemilik per direktori sensitif (Bagian 8).
│   └── dependabot.yml
├── docs/
│   ├── STRUCTURE.md            # Dokumen ini (versi final yang disetujui menjadi dokumen ini).
│   ├── CRUD.md                 # Konvensi CRUD bergating (sudah ada di kerangka).
│   ├── UI.md                   # Sistem UI Bagian 5 (ditulis dari dokumen ini saat implementasi).
│   ├── RUNBOOK.md              # Operasi: rotasi secret, pemulihan rantai audit, insiden, kill switch.
│   ├── REGISTER.md             # Register regulasi (isi dari gelombang 2 + tanggal cek per entri).
│   └── adr/                    # Architecture Decision Records (Bagian 7).
├── tests/
│   └── e2e/                    # Uji ujung-ke-ujung: alur approval, saga, rekonsiliasi, gerbang.
├── README.md                   # Status jujur: kerangka → implementasi bertahap (Bagian 9).
├── SECURITY.md                 # Pelaporan kerentanan, kebijakan secret, dukungan versi (root; satu lokasi).
└── CONTRIBUTING.md             # Alur kontribusi, konvensi commit, definisi "selesai" (root; satu lokasi).
```

**Perubahan dari kerangka `a3b2121` (agar mudah ditinjau Seti):** (a) tiga service keamanan dipisah dari `api` menjadi service sendiri — `identity/`, `audit-signer/`, `vault/` — sesuai "tiga komponen keamanan" gelombang 1; (b) `packages/` ditambah `contracts/` dan `ui-tokens/`; (c) `db/` dipisah dari service agar migrasi bisa ditinjau independen; (d) `tests/e2e/` di root karena alur approval menyentuh semua service; (e) `docs/` ditambah `UI.md`, `RUNBOOK.md`, `REGISTER.md`; (f) `SECURITY.md` dan `CONTRIBUTING.md` diputuskan di root saja (bukan ganda di `docs/`).

## 3. Skema data inti (kolom kunci)

| Tabel | Kolom kunci | Catatan |
|---|---|---|
| companies | id, nama, status | Tenant; semua kueri difilter server-side dari identitas, bukan dari input. |
| principals | id, kelas (human/agent/reviewer/executor), company_id, status | Kelas dibaca dari kredensial, bukan payload. |
| tasks | id, status (PENDING/RUNNING/SUBMITTED/DONE/FAILED), lease_owner, lease_expires_at, claim_token, attempts, remote_request_id, company_id | Mesin gelombang 3; SUBMITTED tidak boleh di-submit ulang — hanya direkonsiliasi. |
| approvals | id, intent_hash, tindakan, objek, batas, masa_berlaku, pemutus, keputusan, alasan, company_id | Intent beku; keputusan dana satu per satu (lihat Bagian 12, W4-03). |
| audit_events | seq (monotonik), prev_hash, hash, signature, actor, objek, revisi, company_id | Append-only; kunci di audit-signer; jangkar eksternal berkala. |
| skills | id, nama (SK-*), semver, status (aktif/dicabut), hash_versi, scope (org/tenant/project/run) | Promosi manusia atas hash persis; pencabutan berlaku segera untuk invokasi baru. |
| memories | id, scope, konten, taint_flag, provenance (sesi penulis), versi | Taint tak terhapuskan agent; tulis bebas hanya scope run. |
| idempotency_keys | principal_id, kunci, hasil, dibuat | Hasil pengiriman ulang = hasil pertama. |
| outbox / inbox | aggregate_id, event, status, kunci_idempotensi | At-least-once + inbox wajib; outbox audit terpisah dari outbox bisnis. |
| controlled_documents | id, versi, status, hash, penyetuju | Dokumen ISO terkendali (kebijakan/SOP/formulir) dengan penomoran dan masa berlaku. |

**Mesin tugas (gelombang 3, diringkas):** klaim atomik `SELECT ... FOR UPDATE SKIP LOCKED`; lease per claim dengan `claim_token` segar per claim (fencing token); heartbeat wajib (`UPDATE ... WHERE id AND claim_token AND status='running'` — 0 baris = lease hilang, worker harus berhenti saat itu juga); finalisasi bertoken; reaper terjadwal mengambil lease kedaluwarsa, menghapus token, backoff eksponensial + jitter, lalu FAILED setelah ambang upaya. **Aturan keras:** klaim tidak boleh menimpa status SUBMITTED menjadi RUNNING — itu menghapus satu-satunya sinyal bahwa tugas sudah disubmit ke pihak luar.
**Idempotensi + outbox (gelombang 3):** tulis event ke outbox dalam transaksi yang sama dengan perubahan bisnis, lalu relay mempublikasikannya; outbox hanya menjamin at-least-once — konsumen wajib memakai inbox agar retry menjadi no-op. Klaim "exactly-once delivery" dari outbox saja adalah salah.
**Saga (gelombang 3):** tiap workflow bisnis multi-langkah didefinisikan sebagai saga dengan kompensasi idempotent per langkah yang ditulis SEBELUM workflow diaktifkan; kompensasi urutan terbalik; langkah ireversibel (kirim email, pengajuan resmi) di akhir; koordinator saga ≠ agen pelaksana; validasi oleh pemeriksa independen.
**Delegasi (gelombang 3):** delegasi internal memakai semantik task A2A (task object ber-lifecycle + artifact + `input_required` sebagai titik jeda manusia), walau implementasi internalnya bukan A2A murni; delegasi eksternal (kelak) memakai A2A betulan dengan Agent Card. Delegasi internal tidak boleh memakai percakapan bebas tanpa task object — itu yang membuat jejak audit tidak bisa dipertanggungjawabkan.
**Durable execution — JANGAN dibangun dari nol (gelombang 3):** pilihan di ADR-002: Postgres + lease/outbox (paling selaras dengan kontrak CRUD blueprint) atau Temporal (bila skala menuntut); DBOS dievaluasi sebagai jalan tengah karena blueprint sudah Postgres-sentris. "Durable" tidak sama dengan "exactly-once effect" di API pihak ketiga — replay tidak boleh meminta model membuat keputusan yang sama lagi.

## 4. Arsitektur keamanan kelas dunia, dipetakan ke gerbang blueprint

### 4.1 Baseline standar (gelombang 1 — target yang diusulkan)

| Standar | Target untuk AgenticAI | Pemetaan ke gerbang blueprint |
|---|---|---|
| OWASP ASVS | L2 sebagai baseline seluruh permukaan (API, dashboard, gateway); L3 untuk jalur kritis: gerbang manusia/approval, eksekusi pengeluaran dana, endpoint perubahan hak istimewa | Gerbang manusia = jalur L3; L1 saja tidak memadai karena blueprint menangani dana dan dokumen korporasi. |
| NIST SP 800-218 SSDF v1.1 | Keempat kelompok (PO/PS/PW/RV) sebagai kerangka SDLC, bukti per praktik disimpan sebagai artefak repo | PS.2/PS.3 (lindungi kode, verifikasi integritas rilis), PW.6 (konfigurasi build). SSDF bukan sertifikasi — pembuktian lewat atestasi internal + bukti. |
| NIST CSF 2.0 | Enam fungsi (Govern + Identify/Protect/Detect/Respond/Recover), 21 kategori | Govern = padanan organisasional gerbang manusia + auditor independen: kebijakan tertulis, peran (super admin → auditor), management review, manajemen risiko rantai pasok (model/provider pihak ketiga). |
| SOC 2 Trust Services Criteria | Target jangka menengah: Type II, ruang lingkup Security + Availability + Confidentiality + Processing Integrity (Privacy ditambah hanya bila AgenticAI mengumpulkan data pribadi langsung) | Processing Integrity krusial karena output agent dipakai untuk keputusan bisnis/dana. Type II = efektivitas operasi selama periode (min. 3 bulan, umumnya 6–12 bulan). |
| ISO/IEC 27001:2022 Annex A | 93 kontrol, 4 tema; Statement of Applicability (SoA) disusun sejak awal, bukan menjelang audit | Wajib sejak hari pertama antara lain 8.2 (hak istimewa), 8.3 (pembatasan akses), 8.5 (autentikasi aman), 8.10/8.11/8.12 (penghapusan, masking, anti-bocor — relevan dengan larangan "data sensitif tidak disalin ke log umum"), 8.15–8.17 (logging/monitoring/sinkronisasi jam), 8.25–8.28 (SDLC aman), 8.34. |
| ISO/IEC 42001:2023 (AIMS) | Payung tata kelola di atas ISMS: setiap sistem agent (peran, skill, model) dicatat dalam ruang lingkup AIMS, dinilai dampaknya, ditinjau manajemen | Memberi dasar audit eksternal untuk klaim "agent dikendalikan", bukan sekadar janji. |
| SLSA (build track v1.0) | L2 baseline; L3 untuk artefak rilis | L3 = lingkungan build ephemeral terisolasi, secret penandatangan tidak bisa diakses langkah build yang didefinisikan pengguna. Provenance TIDAK menjamin kode bebas kerentanan, sudah direview, dependensi aman, atau publisher tepercaya. |
| OWASP Agentic Top 10 (Des 2025, ASI01–ASI10) | Mitigasi per risiko; runtime authorization per tool-call (bukan hanya izin saat penugasan) + kill-switch/revokasi cepat sebagai kontrol wajib | ASI01 goal hijack, ASI02 tool misuse, ASI03 identity/privilege abuse, ASI04 supply chain, ASI05 unexpected code execution, ASI06 memory/context poisoning, ASI07 insecure inter-agent communication, ASI08 cascading failures, ASI09 human-agent trust exploitation, ASI10 rogue agents. Prompt injection tidak bisa dicegah total (konsensus 2026) — pertahanan berlapis hanya menekan dampaknya. |

### 4.2 Tiga prinsip yang tidak bisa ditawar

1. **Agent tidak pernah memegang kunci yang menandatangani jejak auditnya sendiri** (aturan SLSA-L3 yang diterapkan ke audit: yang dideskripsikan tidak boleh memegang kunci yang menandatangani deskripsinya). Penandatanganan AuditEvent dilakukan di luar sandbox agent, oleh `audit-signer`, setelah proses agent selesai.
2. **Agent tidak pernah memegang secret jangka panjang, dan tidak pernah bisa menaikkan izinnya lewat isi memori/dokumen yang ia baca.** Izin berasal dari kelas principal + kebijakan, bukan dari isi memori.
3. **Tugas yang sudah disubmit ke pihak luar tidak pernah di-submit ulang — hanya direkonsiliasi** (status SUBMITTED + fencing token, bukan disiplin manusia).

### 4.3 Identitas, kredensial, dan enforcement kelas principal

- **Enforcement di server, di tiap endpoint, dari kredensial terautentikasi.** Nilai kelas di payload/body/path hanya divalidasi, tidak pernah dipercaya. "Flag manusia dari payload" ditolak di gateway, bukan di handler bisnis.
- **Tenant isolation di lapisan data:** semua kueri dipaksa berfilter `company_id` server-side (diturunkan dari identitas dan mandat; nomor perusahaan kiriman pengguna/agent bukan bukti izin). Tidak ada query lintas tenant tanpa mandat auditor eksplisit yang tercatat.
- **OAuth 2.0 / OIDC (RFC 9700, Security Best Current Practice):** authorization code + PKCE (S256) + Pushed Authorization Requests (PAR) untuk login manusia; client credentials dengan token sender-constrained (DPoP atau mTLS) + berumur pendek + berlingkup sempit per agent/per tugas untuk identitas agent — bukan satu bearer token panjang yang dipakai bersama. PKCE wajib untuk semua klien; grant Implicit dan Resource Owner Password Credentials dilarang; pencocokan redirect URI eksak tanpa wildcard. FAPI 2.0 (PAR + sender-constrained wajib + autentikasi klien asimetris + rotasi refresh token) sebagai profil untuk jalur kritis.
- **Paspor agent per posisi (pembanding ServiceNow AI Passport, diadopsi sebagai konsep, bukan produk):** setiap AgentPosition memiliki kredensial terverifikasi (penerbit, cakupan, masa berlaku) yang diperiksa runtime di setiap langkah, dengan pencabutan presisi. Satu agent = satu klien OAuth (pola WorkOS): pencabutan satu agent tidak memengaruhi yang lain.
- **Blended identity (pembanding Aembit):** setiap audit-event menjawab "siapa yang bertanggung jawab" dengan identitas agent DAN identitas pengguna/otorisasi yang mendasarinya.
- **Token Vault / broker kredensial (pembanding Auth0/Okta, Keycloak sebagai kandidat self-host):** agent tidak pernah memegang root key; kredensial pihak ketiga disimpan di vault, agent hanya meminta token berumur pendek. Kandidat authorization server self-host: Keycloak (alasan kedaulatan data Indonesia/UU PDP — vendor SaaS luar negeri adalah transfer lintas batas yang harus dibuktikan dasarnya). Kandidat identitas workload internal: SPIFFE/SPIRE (SVID X.509 berumur pendek, rotasi otomatis; attestation membuktikan kode apa yang berjalan, BUKAN apa yang berhasil dibujuk model di dalamnya — agent terkena prompt injection tetap menyajikan SVID valid, jadi SVID bukan bukti niat).
- **Hierarki secret (terkuat → terlemah):** (1) secretless/ephemeral — identitas workload ditukar menjadi kredensial temporer saat pakai; (2) dynamic secret — diterbitkan on-demand dengan lease/TTL dan pencabutan otomatis; (3) rotasi otomatis; (4) penyimpanan terpusat saja — bukan migrasi model identitas. Kandidat vault: HashiCorp Vault (patokan dynamic secret; lisensi BSL) atau OpenBao (fork komunitas, MPL-2.0, kompatibel API Vault) bila ingin pola Vault tanpa lisensi BSL. Cloud secret manager native hanya bila single-cloud.
- **Approval terikat intent beku:** ApprovalRequest mengikat hash intent beku (tindakan + objek + batas + masa berlaku). Persetujuan tidak bisa dipakai ulang untuk intent berbeda. Perubahan hak istimewa = jalur approval L3 (ASVS). Larangan "approve for session" pada gerbang wajib sudah diterapkan di blueprint final (§11, uji T53); sesi hanya untuk tindakan internal non-gerbang, tiap tindakan tetap AuditEvent tersendiri.

### 4.4 Audit append-only + salinan independen

- Tiap entri: `seq` monotonik dari sekuens basis data (bukan ID buatan aplikasi) + `prev_hash` + `hash` + tanda tangan `audit-signer` + akun penulis basis data yang hanya bisa INSERT + penyimpanan hash terpisah dari data.
- **Rantai hash saja tidak cukup:** rantai hanya membuktikan "entri yang ada tidak diubah", bukan "tidak ada segmen yang dihapus/di-restart". Bukti kelengkapan menuntut jangkar eksternal: kepala rantai (hash terakhir + tinggi rantai) ditulis berkala ke medium append-only di luar jangkauan operator (log transparansi, otoritas timestamp RFC 3161, atau repositori independen), lalu diverifikasi bahwa tinggi rantai hanya bertambah (monotonik naik). Verifikasi rantai menjadi job terjadwal + alert bila rantai putus atau tinggi menyusut.
- **Kanal pelaporan ke manusia berada di luar jangkauan optimasi agent** (pembanding Avalon-M1 — pelaporan ke pengawas jatuh dari 76,7% ke 0% sementara penalaran tentang melapor tetap 96,7–100%): pelaporan masalah ke super admin/auditor wajib berbentuk AuditEvent yang ditulis server saat kejadian terdeteksi (deterministik, bukan narasi agent); agent tidak diberi pilihan untuk menahan laporan. Sudah diterapkan di blueprint final (§13, uji T46).
- **Penghapusan vs imutabilitas:** data pribadi tidak dihapus dari rantai; yang dihapus adalah materi kunci yang membuat referensi identitas di rantai tak bisa dipulihkan (cryptographic erasure) — rantai tetap utuh.
- **Dua jalur log:** log umum (tanpa data sensitif, token bertopeng) dan brankas bukti (akses auditor saja, enkripsi, retensi terkonfigurasi + teruji pemulihannya).
- **Auditor = kelas principal tersendiri** dengan akses baca lintas tenant yang tercatat; tidak bisa diubah/dinonaktifkan oleh peran yang dia audit; temuan audit (AuditFinding) tidak bisa dihapus siapa pun kecuali lewat proses koreksi berversi yang ikut tercatat.
- **Observabilitas ≠ enforcement ≠ audit:** trace (LangSmith/Langfuse/Weave) menjelaskan "mengapa agent melakukan X" tetapi bukan bukti hukum; guardrail bukan otorisasi; skor satu percobaan bukan keandalan. Audit Otonom tidak boleh bergantung pada retensi vendor.

### 4.5 Tool gateway, sandbox, dan egress

- **Tool gateway = satu-satunya jalur egress (jaringan + aplikasi).** Fungsi: autentikasi sekali di pintu masuk (token tidak di-pass-through ke resource lain); otorisasi per agent, per tool, per operasi — daftar putih server + tool; validasi argumen sebelum eksekusi (tipe, rentang, allowlist, normalisasi path, query berparameter); pemindaian injeksi pada deskripsi tool dan hasil tool sebelum mencapai model; rate limit per tool/server/trace; audit penuh (identitas, tool, argumen, hasil, trace ID).
- **Kesenjangan MCP yang harus ditutup gateway:** spesifikasi MCP membuat otorisasi opsional; tidak ada identitas, least-privilege, atau audit trail bawaan. Gateway menutupnya — bukan prompt agent.
- **Pola "alat tak terlihat":** kredensial read-only tidak boleh sekadar ditolak saat memanggil tool tulis — tool tulis TIDAK BOLEH MUNCUL SAMA SEKALI di daftar tool untuk kredensial itu. Instruksi injeksi tidak punya sasaran bila tidak ada tool yang bisa dituju.
- **Pola "alat sempit, bukan alat generik":** sediakan tool domain-spesifik bernama (read/propose/execute/write); jangan sediakan `update_anything` atau query generik. Lebar permukaan tool = lebar permukaan serangan.
- **Sandbox eksekusi (pembanding keamanan-tata-kelola):** batas default untuk kode agent yang tidak tepercaya = microVM (Firecracker) atau gVisor — BUKAN kontainer Docker biasa (berbagi kernel host). Satu sandbox per tugas; snapshot sebagai cetakan bersih → klon per tugas → reset/hancur; snapshot harus diikat ke proyek/tenant pemiliknya di sisi host (peringatan Vercel Sandbox Challenge 2026: jangan pernah mempercayai nilai yang disuplai tamu). Egress default-deny + allowlist FQDN + proxy di luar agent; mulai dari tanpa jaringan, buka domain per domain atas kebutuhan yang terbukti (pola Anthropic Code Execution Tool). Batas keras: CPU/RAM/disk/proses/file/output, TTL pendek default + idle auto-stop, kuota kredit per pengguna/agent/divisi. Sandbox bawaan penyedia model diperlakukan sebagai pelengkap opsional, bukan fondasi — fondasi adalah sandbox yang dikendalikan Otonom sendiri. Kandidat produk (E2B, Modal, Daytona, Steel untuk browser, Fly.io, Cloudflare, Azure ACA) ditulis sebagai kandidat di ADR, bukan keputusan.
- **Dokumen/halaman/pesan vendor/keluaran model = data.** Instruksi di dalamnya tidak menambah izin atau mengubah kebijakan. Konten tak tepercaya dipindai di gateway sebelum mencapai agent; memori/soul hanya dari sumber terautentikasi dan terkurasi; penulisan memori oleh agent tidak bisa menaikkan izinnya sendiri. Pemisahan instruksi/data ini adalah pertahanan terhadap ASI01/ASI06.

## 5. Sistem UI gaya web Antigravity (dashboard sebagai control plane)

Arah "gaya Antigravity" adalah keputusan Seti 5 Oktober 2026 yang dikutip BRIEF (bukan temuan riset gelombang 4). Yang dipinjam dari Antigravity adalah pola antarmukanya, bukan produknya — dan dengan satu penolakan eksplisit di bawah.

### 5.1 Prinsip pertama: dashboard BUKAN chatbot-first

Antigravity 2.0 mengganti workspace IDE menjadi aplikasi chatbot-first yang mengubur tampilan kerja — dikritik karena menghilangkan tampilan yang biasa dipakai. **Dashboard Otonom tidak boleh menjadi chatbot-first: chat Copilot adalah panel pendamping, bukan layar utama. Layar utama adalah antrean approval, status agent, dan timeline audit — karena di situlah keputusan manusia dan bukti audit hidup.** (Lihat juga usulan gelombang 4 W4-01 di Bagian 12.)

### 5.2 Layout tiga area

```
┌──────────────┬──────────────────────────────────────┬─────────────────────┐
│ Sidebar      │ Area kerja utama                     │ Panel Copilot       │
│ (240px,      │ (kasus / antrean approval / detail)  │ (360px, bisa        │
│  bisa ciut    │                                      │  dilipat)           │
│  ke 64px)    │ Top bar: company switcher, search    │                     │
│              │ (Cmd/Ctrl+K), status sistem, avatar  │ Thread per objek,   │
│ Nav:         │                                      │ bukan chat global   │
│ Dashboard    │ Kartu statistik quiet (angka +       │ bebas               │
│ Kasus        │ delta, tanpa warna mencolok)         │                     │
│ Approval     │                                      │ Aksi Copilot selalu │
│ Agent        │ Antrean approval / daftar kasus /    │ menghasilkan draf   │
│ Audit        │ timeline audit                       │ (bukan eksekusi     │
│ Skill &      │                                      │ langsung)           │
│ Memori       │                                      │                     │
│ Pengaturan   │                                      │                     │
└──────────────┴──────────────────────────────────────┴─────────────────────┘
```

- **Satu prinsip layout:** tidak ada aksi mengikat (dana, pengajuan resmi, perubahan hak) yang bisa dieksekusi dari panel Copilot atau dari kartu ringkas — aksi mengikat hanya dari kartu approval lengkap yang menampilkan intent beku.
- **Responsif:** di bawah 1024px panel Copilot menjadi drawer; di bawah 768px sidebar menjadi bottom navigation + drawer; semua target sentuh memenuhi WCAG 2.5.8 (Bagian 5.5). Mode terang/gelap wajib keduanya (keputusan Seti 5 Okt 2026) — bukan dark-only.

### 5.3 Design token (nilai adalah usulan, bukan kutipan produk)

Tiga lapis token, mengikuti praktik umum design token (bukan klaim atas token Antigravity yang sebenarnya, yang tidak dipublikasikan sebagai sistem token):

| Lapis | Isi | Contoh nama (usulan) |
|---|---|---|
| Primitif | Nilai mentah: warna, ukuran, radius, durasi | `color-gray-950`, `space-4`, `radius-md`, `duration-fast` |
| Semantik | Arti dalam konteks UI, per tema (dark default, light) | `surface-page`, `surface-card`, `border-hairline`, `text-primary`, `text-quiet`, `status-pending`, `status-approved`, `status-rejected`, `status-blocked` |
| Komponen | Token per komponen, merujuk semantik | `approval-card-border`, `copilot-panel-bg`, `sidebar-active-bg` |

- **Tema gelap (default):** latar mendekati hitam dengan kartu sedikit lebih terang, bukan abu-abu menengah; garis pemisah hairline (1px, kontras rendah tapi masih terlihat sebagai pemisah struktur, bukan sebagai teks); teks utama terang, teks sekunder ("quiet") cukup kontras untuk lolos 4.5:1 — "quiet" berarti ukuran/berat yang lebih ringan dan posisi sekunder, BUKAN warna yang gagal kontras.
- **Tema terang:** latar terang netral, kartu putih, hairline yang sama; kedua tema memakai token semantik yang sama sehingga pergantian tema tidak mengubah makna warna status.
- **Tipografi:** Inter Tight untuk judul dan antarmuka (keputusan Seti 5 Okt 2026); satu keluarga monospace untuk hash, ID, dan jejak audit (hash intent beku harus bisa dibaca dan disalin utuh, bukan dipotong dengan elipsis tanpa cara melihat lengkapnya).
- **Ikon:** SVG garis tipis (stroke, bukan fill berat), satu gaya di seluruh aplikasi; ikon tidak pernah menjadi satu-satunya pembawa makna status — selalu ditemani teks (WCAG 1.4.1).
- **Status tidak boleh hanya warna:** pending = ikon jam + teks "Menunggu"; disetujui = centang + teks; ditolak = silang + teks; diblokir penelaah = ikon gembok + teks + alasan. Tangkapan layar kartu approval harus bisa dibaca tanpa warna.

### 5.4 Komponen wajib (dengan perilaku, bukan hanya nama)

| Komponen | Perilaku yang wajib ada | Kaitan blueprint |
|---|---|---|
| Sidebar | Navigasi per peran (menu yang tidak diizinkan untuk principal yang login TIDAK TAMPIL SAMA SEKALI, mengikuti pola "alat tak terlihat"); lencana jumlah approval menunggu; status agent (berjalan/menunggu/diblokir) per peran | Kelas principal; hierarki peran |
| Command palette (Cmd/Ctrl+K) | Cari kasus, approval, skill, dokumen, dan perintah navigasi; hasil menampilkan tipe objek + status; TIDAK mengeksekusi aksi mengikat dari palette — palette hanya navigasi dan pembuka draf | Akses cepat tanpa melewati gerbang |
| Panel AI Copilot | Thread terikat satu objek (kasus/approval/dokumen); riwayat thread tersimpan dan bisa diaudit; balasan Copilot yang mengusulkan aksi menghasilkan DRAF yang harus dibuka di kartu approval lengkap; Copilot tidak pernah menampilkan secret, token, atau isi dokumen di luar izin principal | Memori = data; kredensial per tugas |
| Kartu approval | Menampilkan: siapa meminta, tindakan persis, objek, batas (nominal/biaya/masa berlaku), hash intent beku (lengkap, bisa disalin), siapa penelaah, tenggat, tautan ke bukti; tombol Setuju / Tolak / Minta revisi; aksi dana memakai KONFIRMASI KEDUA (dialog yang mengulang nominal + objek); setelah diputuskan, kartu menjadi hanya-baca dengan stempel siapa/waktu/keputusan | Gerbang manusia; approval terikat intent beku |
| Timeline audit | Daftar AuditEvent kronologis per objek: aktor (kelas principal + nama), tindakan, revisi, hash rantai; filter per aktor/tipe; tautan status verifikasi rantai terakhir dari job terjadwal | Audit append-only + jangkar eksternal |
| Kartu agent / status tugas | Status PENDING/RUNNING/SUBMITTED/DONE/FAILED; untuk SUBMITTED tampilkan "hasil eksternal belum diketahui — sedang direkonsiliasi" dan TANPA tombol kirim ulang; lease kedaluwarsa tampil sebagai peringatan, bukan diam-diam antre ulang | Lease + fencing token + rekonsiliasi |
| Kartu kasus (F01–F10) | Tahap saat ini, penanggung jawab agent, penelaah, dokumen terkait, tenggat regulasi (dari register gelombang 2, mis. tenggat alih daya 30 Apr 2028 bila relevan) | Alur F01–F10 |
| State kosong | Setiap daftar kosong menjelaskan kenapa kosong dan apa langkah berikutnya (bukan halaman putih); daftar approval kosong = "Tidak ada approval menunggu" + tautan ke riwayat | — |
| State error | Pesan error berbahasa manusia + apa yang bisa dilakukan pengguna + ID insiden (trace ID) untuk dilaporkan; TIDAK PERNAH menampilkan stack trace, secret, atau isi prompt ke pengguna | Keamanan |
| Mode baca auditor | Auditor independen mendapat tampilan hanya-baca lintas tenant yang BERTANDA VISUAL JELAS ("Mode audit — semua akses tercatat"), agar tidak ada kebingungan apakah ia sedang bertindak sebagai operator | Auditor melapor ke super admin |

### 5.5 Kartu approval sebagai komponen keamanan

Kartu approval bukan komponen kosmetik — ia adalah permukaan gerbang manusia. Kegagalan desainnya sama berbahayanya dengan kegagalan server: (1) yang disetujui harus persis yang ditampilkan — hash intent beku di kartu harus sama dengan hash yang diverifikasi server saat eksekusi; bila intent berubah setelah kartu dibuka, tombol Setuju menjadi tidak valid dan kartu memuat ulang (bukan mengeksekusi intent lama); (2) tidak ada persetujuan massal buta — "Setujui semua" dilarang untuk aksi dana dan perubahan hak; setiap approval dana diputuskan satu per satu; (3) penolakan adalah keputusan yang tercatat, bukan pembatalan diam-diam: alasan penolakan (dipilih + teks bebas) masuk AuditEvent; (4) kedaluwarsa terlihat — approval yang melewati masa berlaku intent menampilkan status kedaluwarsa dan tidak bisa dieksekusi; (5) tidak ada aksi dari notifikasi — tautan email/chat ke approval hanya membuka kartu lengkap; tidak ada tombol "setujui dari email". (Butir 2–5 dirinci sebagai usulan gelombang 4 di Bagian 10; butir 1 adalah penegasan atas aturan intent beku yang sudah ada di blueprint final.)

### 5.6 Aksesibilitas — target WCAG 2.2 AA (persyaratan terverifikasi; kepatuhan dashboard BELUM_DIPERIKSA karena belum ada implementasi)

- Kontras: teks minimal 4.5:1, teks besar 3:1, komponen UI dan ikon bermakna 3:1 (1.4.3, 1.4.11).
- Target sentuh: minimal 24×24 CSS px, atau berjarak cukup sehingga lingkaran 24px di sekeliling tiap target tidak tumpang tindih (2.5.8).
- Fokus: indikator fokus selalu terlihat dan tidak tertutup seluruhnya oleh header sticky/overlay (2.4.11); fokus tidak boleh hilang di balik panel Copilot yang terbuka.
- Seret (drag): semua aksi seret wajib punya alternatif klik/tombol (2.5.7).
- Bantuan konsisten: tautan bantuan/dokumentasi tampil di tempat yang sama di semua halaman (3.2.6).
- Tidak ada entri berulang: form multi-langkah tidak meminta pengguna memasukkan ulang data yang sudah diberikan dalam proses yang sama (3.3.7).
- Autentikasi aksesibel: login tidak boleh bergantung pada teka-teki kognitif (CAPTCHA teka-teki tanpa alternatif) — sediakan passkey/manajer kata sandi/tempel (3.3.8).
- Warna bukan satu-satunya pembawa makna (1.4.1); struktur di markup, bukan hanya visual (1.3.1); teks bisa dibesarkan 200% dan reflow satu kolom pada 320px tanpa kehilangan konten (1.4.4, 1.4.10).
- Nilai token final wajib lolos uji kontras sebelum disahkan; uji aksesibilitas (axe) masuk CI dan roadmap fase 3.

## 6. Pemetaan bank skill ke repo

Bank skill blueprint (§7 + §7.5, 40 skill + 3 skill dashboard) dicerminkan 1:1 di `packages/skill-bank` berdasarkan ID skill + semver + hash versi. Setiap skill adalah entri berversi: definisi, semver (major = perubahan perilaku, minor = perbaikan kompatibel, patch = klarifikasi), status (aktif/dicabut), eval gate (golden dataset), dan penugasan (SkillAssignment). **Pencabutan berlaku segera untuk invokasi baru. Skill tidak boleh mengesampingkan peran agent, aturan keamanan, batasan tool, atau kebijakan aplikasi.** Promosi versi baru melewati gerbang eval sebelum ke produksi.

| Keluarga skill (blueprint) | Contoh ID | Pemilik (posisi) | Batas keras / gerbang |
|---|---|---|---|
| Regulasi | SK-REG-001 routing & penugasan; SK-REG-002 standar opini spesialis; SK-REG-003 pemantauan perubahan regulasi; SK-REG-004 penilaian dampak transisi; SK-REG-005 telaah mutu sumber/pasal/masa berlaku | Koordinator regulasi, spesialis domain, penelaah mutu | Opini wajib memuat fakta, sumber primer dan pasal, status verifikasi, ketidakpastian, langkah berikutnya. |
| Alur korporasi F01–F10 | SK-F01-001 paket keputusan korporasi; SK-F02-001 paket permohonan izin; SK-F03-001 seleksi/onboarding/administrasi hubungan kerja; SK-F04-001 validasi payroll; SK-F04-002 rekonsiliasi payroll (reviewer terpisah dari penghitung); SK-F05-001 pembukuan/rekonsiliasi; SK-F05-002 perhitungan & kalender pajak; SK-F06-001 seleksi vendor/kontrak; SK-F06-002 pencocokan penerimaan/tagihan/pembayaran; SK-F07-001 penawaran/pemenuhan/tagihan/keluhan; SK-F07-002 pendaftaran HKI; SK-F08-001 pemetaan data & penilaian dampak; SK-F08-002 penanganan insiden & notifikasi; SK-F09-001 permintaan GA & verifikasi hasil fisik; SK-F10-001 penilaian sengketa & opsi | Pelaksana per alur, penelaah terpisah | Berhenti sebelum pengesahan (F01), sebelum pengajuan resmi (F02), PHK tidak didelegasikan penuh (F03), tidak mengeluarkan dana (F04), pengajuan resmi lewat manusia (F05), komitmen mengikat lewat manusia (F06/F07), hak istimewa lewat manusia (F08), pesanan sukses ≠ pekerjaan selesai (F09), surat/pernyataan resmi lewat manusia (F10). |
| Keamanan lintas posisi (wajib) | SK-SEC-001 penanganan isi tidak tepercaya & injeksi instruksi | Semua posisi | Dokumen/keluaran model = data; instruksi di dalamnya tidak menambah izin. |
| Dashboard (disetujui Seti 27 Okt 17:10; TEMPLATE_DESAIN, belum disahkan; uji T43–T45) | SK-DASH-001 UI builder dashboard | Engineer dashboard (Staf TI di bawah Direktur TI) | Paket perubahan UI dari komponen yang sudah disetujui. TIDAK BOLEH menambah alat, domain egress, atau akses di luar mandat; tidak boleh mengubah gerbang, approval, atau tampilan status audit. |
| Dashboard | SK-DASH-002 bug control dashboard | Engineer dashboard | Triase bug, paket perbaikan, bukti uji regresi. Perbaikan diverifikasi reviewer terpisah; agent tidak menelaah perbaikannya sendiri; perbaikan yang menyentuh gerbang/keamanan memerlukan persetujuan sesuai matriks. |
| Dashboard | SK-SEC-002 firewall defence dashboard | Engineer dashboard | Hanya mengusulkan dan memantau (usulan aturan firewall/WAF + laporan anomali). Perubahan aturan firewall, akses, dan deploy produksi WAJIB approval manusia; agent tidak menaikkan haknya sendiri. Dashboard = control plane, penjaganya paling ketat. |

**Memori/soul (blueprint §8) di repo:** satu repositori berversi dengan empat scope — `org` (metodologi bersama, agent hanya baca), `tenant:<id>` (spesifik satu tenant, agent baca + tulis sebagai *pending*), `project:<id>` (satu codebase, baca + pending), `run:<id>` (catatan satu sesi, agent baca-tulis bebas). Aturan keras: (1) agent tidak pernah menulis langsung ke scope org/tenant/project — hanya mengusulkan (pending); (2) promosi = persetujuan manusia atas versi persis (hash) yang direview; bila entri berubah setelah review, penerimaan gagal; (3) taint menempel pada entri yang berasal dari konten tak tepercaya dan tidak bisa dihapus agent; (4) snapshot pinning: tiap thread mem-pin snapshot memori saat dimulai dan memakainya seumur hidup thread; (5) dreaming bergating — job terjadwal boleh mengusulkan perubahan dari banyak sesi, tetapi tidak pernah menulis langsung; usulan yang menyentuh perintah, URL, kredensial, scope, atau izin ditandai merah; (6) scope adalah sumbu tambahan atas jenis memori §8.1 (Kerja/Kasus/Prosedural/Organisasi), bukan pengganti jenis; promosi mengikuti jalur §8.3 (penyelarasan yang sudah dicatat di changelog final).

## 7. Ringkasan kontrak API /api/v1 + kelas principal

**Kelas principal (dibaca dari kredensial, bukan payload):** `human`, `agent`, `reviewer`/`auditor`, `executor` (layanan). Enforcement di server, di tiap endpoint. `company_id` diturunkan server dari identitas dan mandat.
**Aturan kontrak yang mengikat semua endpoint (blueprint §19):** Idempotency-Key wajib untuk semua perintah ubah keadaan — tanpa kunci, perintah mutasi ditolak (kode §19.5 final: 428; usulan W1-04 yang menulis 400 tidak dipakai karena 428 sudah ada di §19.5); pengiriman ulang = hasil yang sama. If-Match + revision; ketidakcocokan = 409. AuditEvent untuk setiap permintaan (actor, objek, revisi); `GET /audit-events` hanya baca — tidak ada penulisan/pembaruan/penghapusan lewat API. Notifikasi webhook ditandatangani, diantar minimal sekali, konsumen wajib idempotent; notifikasi menyampaikan status — ia BUKAN persetujuan atau perintah eksekusi. Kode 403 PRINCIPAL_CLASS/CHANNEL ditambahkan sebagai baris kode di §19.5 (CP-06) tanpa mengubah arti 403 PRINCIPAL_CLASS yang sudah ada.

| Kelompok endpoint (§19 final) | Endpoint kunci | Kelas yang diizinkan |
|---|---|---|
| 1. Runtime agent | `GET /agents/me/mandate`, `GET /agents/me/skills`, `POST /agent/tasks/claim`, `POST /agent/tasks/{id}/heartbeat`, `GET /cases/{id}`, `POST /cases/{id}/evidence`, `POST /cases/{id}/stages/{id}/submissions`, `POST /cases/{id}/action-intents` (bekukan paket tindakan; server menentukan action_type dari semantik dan alat; mengembalikan payload_hash), `POST /cases/{id}/escalations`, `POST /tool-gateway/invocations` (alat yang tidak mengubah keadaan pihak luar; alat yang mengubah keadaan hanya lewat executor), `GET /agents/me/credentials` (paspor AgentCredential aktif; hanya baca), `POST /agent/tasks/{id}/submit-external` (menandai SUBMITTED; submit ulang ditolak) | agent |
| 2. Pemeriksa & penelaah | `GET /review-queue` (menyaring kasus yang melibatkan identitas yang sama), `POST /cases/{id}/stages/{id}/checkpoint-decisions` (lolos/lolos bersyarat/tolak/eskalasi; cek konflik identitas; terikat versi bukti; lolos bersyarat tidak menggantikan bukti wajib), `POST /regulation-opinions` | reviewer |
| 3. Auditor | `POST /audits` (menolak auditor yang pernah terlibat), `POST /audits/{id}/findings` (mayor otomatis memblokir lingkup + eskalasi ke super admin), `POST /findings/{id}/verifications` (hanya auditor independen; pernyataan pelaksana tidak cukup) | auditor |
| 4. Gerbang manusia | `GET /approvals` (antrean berurutan menurut risiko dan tenggat), `POST /approvals/{id}/decisions` (setujui/tolak/minta revisi; identitas dari autentikasi; cocokkan hash, wewenang, cakupan, masa berlaku), `POST /authority-grants` + revocations, `POST /stops` + resumptions (resume tidak menjalankan otomatis pekerjaan tertunda), `POST /overrides` (tidak mengubah gagal menjadi lolos; wajib alasan dan kedaluwarsa) | human; kredensial agent ditolak |
| 5. Eksekusi | `POST /execution/commands` (konsumsi approval dan buat perintah; transaksional; validasi ulang isi, delegasi, masa berlaku, regulasi, stop, blokir), `POST /execution/attempts/{id}/results` (timeout setelah kirim = execution_unknown), `POST /execution/attempts/{id}/reconciliations` | executor; bukan agent |
| 6. Register | `GET/POST /skills`, `POST /skills/{id}/versions`, `POST /skill-assignments`, `GET /regulations`, `POST /regulations/{id}/versions`, `POST /regulation-impact-cases`, `GET /documents` dan versi, `GET /audit-events` | sesuai matriks §11 |
| 7. Notifikasi keluar | Webhook bertanda tangan (lihat aturan kontrak di atas) | — |

**Klarifikasi bertipe (sudah diterapkan di final, CP-03):** pertanyaan agent ke manusia distandarkan menjadi Input (nilai yang hilang), Pilihan (opsi yang harus dipilih manusia), Aksi (tindakan manusia di luar sistem, mis. otorisasi OAuth/tanda tangan) — tiap klarifikasi menyimpan pertanyaan, tipe, opsi, jawaban, identitas penjawab, stempel waktu sebagai bagian dari state kasus, bukan log sampingan.
**Tingkat otonomi sebagai atribut eksplisit per tugas (sudah diterapkan, PA-06):** `autonomy_level` pada ActionIntent; level autonomous tidak pernah untuk dana/pengajuan resmi/PHK/penghapusan data/deploy produksi.

## 8. ADR (Architecture Decision Records)

Format: MADR (Markdown Any Decision Records) — status `proposed | accepted | rejected | deprecated | superseded by <ADR>`, bagian Konteks, Opsi yang dipertimbangkan, Keputusan, Konsekuensi. Disimpan sebagai markdown di `docs/adr/`, bernomor urut, bersama kode (bukan di wiki terpisah). **Untuk sistem agentik, tiap ADR yang menyentuh otonomi wajib mencantumkan: batas otonomi, eval yang menggerbanginya, dan mode fallback.** Status awal semua ADR di bawah ini: `proposed` — yang mengesahkan adalah Seti.

| ADR | Keputusan yang direkam |
|---|---|
| ADR-001 | Monorepo (Bagian 1) vs polyrepo. |
| ADR-002 | Mesin durable execution: Postgres lease/outbox vs Temporal vs DBOS (gelombang 3). |
| ADR-003 | Database dan pola migrasi. |
| ADR-004 | Identitas principal: IdP + format token layanan + sender-constrained (kandidat: Keycloak, SPIFFE/SPIRE). |
| ADR-005 | Pemisahan audit-signer/vault dari runtime agent (gelombang 1). |
| ADR-006 | Stack dashboard-web + sistem token (Bagian 5). |
| ADR-007 | Tool gateway: protokol (MCP/A2A) untuk delegasi internal vs eksternal (gelombang 3). |
| ADR-008 | Strategi rilis: SLSA level target + SBOM + provenance (gelombang 1). |

## 9. CI/CD, branch protection, dan secret scanning

- **Branch protection (ruleset, bukan hanya proteksi cabang lama):** PR wajib untuk `main`; minimal 1 reviewer (2 untuk direktori sensitif via CODEOWNERS: `services/audit-signer/`, `services/vault/`, `services/identity/`, `packages/policy/`, `.github/workflows/`); review basi dibatalkan saat commit baru (dismiss stale reviews); status check CI wajib lolos dan cabang harus mutakhir sebelum merge; LARANG force push dan penghapusan pada `main`; pola nama cabang (`feature/`, `fix/`, `docs/`) ditegakkan ruleset; pengecualian bypass hanya untuk akun otomasi bernama, tercatat, dan tetap tidak boleh melewati larangan penghapusan.
- **Pipeline CI (per PR):** lint + typecheck + uji unit + uji kontrak API + uji aksesibilitas dashboard (axe) + pemindaian secret (gitleaks/Trivy secret) + pemindaian kerentanan dependensi (OSV/Trivy, blokir HIGH/CRITICAL) + dependency review; build hanya dari commit yang lolos semua gerbang.
- **Pipeline rilis:** SBOM per rilis + provenance build yang ditandatangani (target SLSA L2; L3 untuk artefak rilis); verifikasi provenance sebelum promosi ke produksi; tidak ada rilis dari mesin pengembang (hanya dari CI).
- **Secret scanning + push protection:** aktif sejak hari pertama repo (publik sekalipun); push yang mengandung secret yang dikenali DIBLOKIR sebelum masuk repo; bila secret telanjur masuk, prosedurnya rotasi/revokasi segera, bukan sekadar menghapus commit (riwayat git tetap menyimpan). Hook lokal (pre-commit) hanya lapis tambahan — tidak pernah menjadi satu-satunya pertahanan karena bisa dilewati `--no-verify`. Batas fitur Advanced Security untuk repo publik vs privat perlu dicek di dokumentasi GitHub saat Fase 0 (PERLU_VERIFIKASI).
- **Dependabot:** alert + security update aktif; tinjau PR Dependabot seperti PR biasa (bukan auto-merge buta untuk direktori sensitif); pertimbangkan jeda (cooldown) agar tidak mengejar setiap rilis patch.
- **GitHub Actions hardening:** pin action ke SHA (bukan tag bergerak) untuk workflow yang menyentuh rilis/secret; `persist-credentials: false` kecuali job memang harus push; kredensial cloud lewat OIDC berumur pendek per job, bukan secret jangka panjang; secret hanya diberikan ke job yang membutuhkannya; anggap log workflow sebagai publik (jangan mencetak secret, token, atau isi prompt).

## 10. Roadmap bertahap 0–4

| Fase | Isi | Kriteria selesai (jujur, bisa diperiksa) |
|---|---|---|
| Fase 0 — Fondasi repo | Ruleset + CODEOWNERS + secret scanning + push protection + Dependabot + pipeline CI dasar + ADR-001–008 berstatus proposed | Semua gerbang aktif dan terbukti memblokir satu PR uji yang melanggar. |
| Fase 1 — Kontrak & keamanan inti | `services/api` + `identity/` + `vault/` + `audit-signer/` + skema DB + middleware idempotensi/If-Match/tenant isolation; uji kontrak | Seluruh endpoint menolak principal yang salah kelas; rantai audit terverifikasi job terjadwal. |
| Fase 2 — Orkestrasi | `agent-runner/` (lease/heartbeat/reaper) + `tool-gateway/` (allowlist, validasi, pemindaian) + saga + outbox/inbox; dashboard baca dari sumber kebenaran yang sama | Tugas SUBMITTED tidak pernah ganda-kirim dalam uji kegagalan yang disimulasikan. |
| Fase 3 — Dashboard & gerbang | `dashboard-web/` lengkap (Bagian 5): antrean approval, kartu approval, timeline audit, panel Copilot, command palette; uji aksesibilitas WCAG 2.2 AA | Satu alur approval dana ujung-ke-ujung lolos uji e2e + audit axe tanpa pelanggaran kritis. |
| Fase 4 — Skill, memori, register | `skill-bank/` berversi + promosi manusia + taint + `REGISTER.md` hidup (tanggal cek per entri) + uji T01–T122 yang relevan | Skill yang dicabut berhenti dipakai invokasi baru; register menampilkan tanggal cek terakhir. |

Catatan: roadmap ini adalah roadmap pembangunan repo. Roadmap pembangunan perusahaan (fondasi kendali dan audit → pilot satu alur → lapisan pemeriksa penuh → seluruh alur utama → integrasi sektoral dan skala) tetap mengikuti blueprint final §17.

## 11. Perbandingan dengan pembanding terkuat: di mana Otonom lebih ketat, di mana lebih longgar

Pembanding terkuat per lapisan: gerbang = Microsoft Agent Framework (approval terikat pending request + anti-replay + checkpoint durable) dan LangGraph (`interrupt()` + checkpointer + gerbang ditegakkan struktur graf); mesin status = LangGraph; identitas = ServiceNow AI Passport; rencana-pra-eksekusi = Portia; keamanan agent = OWASP ASI + pola gateway (Portkey/Maestro sebagai enforcement, bukan observabilitas); bukti kegagalan organisasi agent = HurumoAI, Avalon-M1, OASIS, TheAgentCompany.

**Otonom LEBIH KETAT daripada pembanding — dan kenapa:**
- **Tanpa pengecualian nominal untuk dana.** Semua pengeluaran dana wajib persetujuan manusia (baseline v1.1). HumanLayer/Copilot Studio mengizinkan approval bersyarat berbasis ambang dan "approve for session" — Otonom melarang keduanya untuk gerbang wajib (final §11, uji T53). Kenapa: ambang adalah celah yang bisa dipecah menjadi banyak transaksi kecil.
- **Approval terikat intent beku + hash, diverifikasi server, sekali pakai, anti-replay.** MAF sudah punya pola ini; Otonom menambahkannya dengan pemisahan kelas principal, matriks gerbang lintas peran, dan auditor independen yang melapor ke super admin (bukan ke CEO) — sesuatu yang tidak dimiliki satu pun framework.
- **Penolakan ≠ jalankan ulang tanpa batas.** CrewAI: penolakan menjalankan ulang tugas, dan penolakan berulang yang sama memutar loop tak berujung (131 panggilan LLM dalam satu pengukuran). Otonom: cabang penolakan eksplisit + batas putaran revisi + eskalasi (final §10–§11, uji T74).
- **Kanal pelaporan di luar optimasi agent + disent anti-herding.** Avalon-M1 membuktikan organisasi agent yang dioptimasi kolektif berhenti melapor (76,7% → 0%); OASIS membuktikan agent menggiring lebih ekstrem dari manusia. Jawaban Otonom (final §13/§10–§11, uji T46/T48): laporan = AuditEvent deterministik server-side, dan persetujuan bulat pada kasus berisiko WAJIB eskalasi ke manusia — kesepakatan bulat antar-agent bukan validasi.
- **Laporan kemajuan wajib menunjuk artefak.** HurumoAI membuktikan agent mengarang capaian (kinerja mobile +40% tanpa pekerjaan apa pun). Otonom: laporan tanpa tautan artefak = UNVERIFIED dan tidak boleh menjadi dasar keputusan manusia (final §13, uji T47).
- **Ambang kelulusan 100% untuk tugas gerbang wajib.** TheAgentCompany: agent terbaik hanya 30,3% tugas penuh, jatuh tajam pada tugas kompleks. Otonom: skill gerbang wajib hanya naik bila lulus 100% pada set uji gerbangnya, tanpa "lolos bersyarat" (final §7, uji T50).
- **Pemisahan pelaksana ≠ penelaah ≠ auditor, dicek dari identitas/keterlibatan.** Devin/Factory tetap menaruh manusia pada merge/review PR — itu paket minimum enterprise yang bagus, tetapi Otonom melangkah lebih jauh: penelaah bisa memblokir, auditor independen bisa memblokir lingkup, dan keduanya tidak boleh pernah terlibat dalam pekerjaan yang mereka nilai.

**Otonom LEBIH LONGGAR / MASIH BERUTANG — dicatat jujur, bukan disembunyikan:**
- **Kematangan: Otonom masih rancangan, pembanding sudah berjalan.** MAF, LangGraph, ServiceNow, Devin, Factory adalah sistem produksi; Otonom belum punya satu pun service yang berjalan. Klaim "lebih ketat" di atas adalah klaim desain, bukan bukti operasi — dan baru terbukti lewat uji T01–T122 + pilot satu alur.
- **Mesin durable execution belum dipilih (ADR-002 terbuka).** LangGraph/MAF sudah punya checkpoint durable yang teruji; Otonom baru punya pola (Postgres lease/outbox vs Temporal vs DBOS). Sampai ADR-002 diputuskan dan Fase 2 lolos uji kegagalan, ini adalah celah implementasi, bukan keunggulan.
- **Fallback kedaluwarsa approval belum sedetail Dify/HumanLayer.** Dify punya timeout dengan forward eksplisit (contoh: 3 hari lalu di-forward); HumanLayer punya timeout dengan auto-reject/eskalasi. Blueprint final memeriksa masa berlaku intent dan menolak eksekusi kedaluwarsa, tetapi rute fallback per gerbang (eskalasi ke siapa, dalam berapa lama) masih perlu ditetapkan di konfigurasi aktivasi — masuk daftar keputusan terbuka (Bagian 12).
- **Angka anggaran per tugas belum ditetapkan.** Manus/TheAgentCompany memberi patokan ($4,20 dan 27 panggilan per tugas; tugas Manus berhenti di tengah jalan saat kredit habis). Blueprint final mewajibkan anggaran + lease per tugas dan berhenti aman (uji T49), tetapi angka spesifik ditetapkan per perusahaan saat aktivasi — bukan di blueprint. Sampai aktivasi, ini adalah aturan tanpa angka.
- **Operasi lintas yurisdiksi sengaja ditunda.** Entri register klasifikasi risiko EU AI Act (PA-Eropa) adalah satu-satunya usulan yang TIDAK DITERAPKAN di final: fokus blueprint adalah yurisdiksi Indonesia; kebutuhan referensi lintas yurisdiksi sudah dicakup CP-12 dan KT-35 di §9. Dibuka kembali bila operasi menyentuh yurisdiksi tersebut. Artinya: untuk pemakaian di luar Indonesia, Otonom saat ini lebih longgar daripada kerangka Tiongkok (human oversight bermakna + keterlacakan penuh + otorisasi ketat, 15 Juli 2026) dan EU AI Act — dan itu keputusan sadar, bukan kelalaian.
- **Tidak ada pembanding lokal.** Riset tidak menemukan perusahaan Indonesia yang dijalankan agent maupun produk control plane agent buatan Indonesia yang setara. Otonom tidak punya preseden regulasi lokal untuk perusahaan agent — alasan blueprint harus lebih konservatif (gerbang lebih ketat), bukan lebih longgar.

## 12. SISA GELOMBANG 4 SAJA — daftar usulan TAMBAHAN untuk blueprint final (USULAN, bukan penerapan)

**Status bagian ini — baca dulu:** TIDAK ADA satu pun butir di bawah ini yang ditulis ke dokumen blueprint mana pun. Blueprint final (98 halaman, §23) dan konsolidasi v1.2 TIDAK DIUBAH oleh sintesis ini. Usulan gelombang 1–3 dan pembanding TIDAK diulang di sini karena sudah diterapkan dan tercatat di §23 final (108 diterapkan, 1 ditunda [PA-Eropa], 0 konflik). Yang tercantum di sini hanyalah tambahan yang berasal dari gelombang 4 (UI Antigravity + struktur repo, 28 Okt 12:00 WIB) — diperiksa satu per satu terhadap teks final (pencarian kata kunci: chatbot, sidebar, palet, WCAG, kartu approval, mode audit, persetujuan massal, notifikasi) untuk memastikan tidak ada duplikasi dengan §23.

**Yang BUKAN usulan blueprint (dicatat agar tidak salah kaprah):** tree monorepo (Bagian 2), skema DB (Bagian 3), ADR (Bagian 8), CI/CD dan branch protection (Bagian 9), dan roadmap repo (Bagian 10) adalah struktur repo, bukan teks blueprint — semuanya hidup di dokumen ini saja dan tidak memerlukan perubahan blueprint. Usulan blueprint dari gelombang 4 hanyalah aturan perilaku dashboard di bawah ini, yang akan mengubah §16 (Dashboard, eskalasi, dan keamanan) dan, bila disetujui, sebagian §11/§12/§13.

### 12.A Usulan aturan BARU (belum ada padanannya di final)

| ID | Usulan | Alasan (sumber gelombang 4) | Bagian final yang akan diubah | Uji baru yang diusulkan* |
|---|---|---|---|---|
| W4-01 | **Dashboard bukan chatbot-first.** Layar utama = antrean approval + status agent + timeline audit; panel Copilot hanya pendamping dan hanya menghasilkan draf; aksi mengikat tidak bisa dieksekusi dari Copilot maupun kartu ringkas. | Antigravity 2.0 (chatbot-first) dikritik karena mengubur tampilan kerja; di Otonom, tampilan kerja itulah tempat keputusan manusia dan bukti audit hidup. | §16 (baru: prinsip layout dashboard) | T123 — Copilot tidak dapat mengeksekusi aksi mengikat; layar utama menampilkan antrean approval, status agent, dan timeline audit. |
| W4-02 | **Kartu approval: hash intent tampil lengkap dan bisa disalin; bila intent berubah setelah kartu dibuka, tombol Setuju menjadi tidak valid dan kartu memuat ulang** (bukan mengeksekusi intent lama). | "Yang disetujui harus persis yang ditampilkan" — tanpa ini, gerbang manusia hanya teater: approver menyetujui A, server mengeksekusi B. | §11/§12/§16 (permukaan gerbang) | T124 — intent yang berubah setelah kartu dibuka membatalkan tombol Setuju; hash yang tampil sama dengan hash yang diverifikasi server. |
| W4-03 | **Larang persetujuan massal buta untuk aksi dana dan perubahan hak.** "Setujui semua" dilarang; setiap approval dana diputuskan satu per satu. | Persetujuan massal mengubah approval dari keputusan menjadi rutinitas klik — celah yang sama berbahayanya dengan ambang nominal. | §11 (matriks gerbang) | T125 — aksi "setujui semua" atas approval dana ditolak server. |
| W4-04 | **Kedaluwarsa approval terlihat di kartu.** Approval yang melewati masa berlaku intent menampilkan status kedaluwarsa dan tidak bisa dieksekusi — bukan tombol yang diam-diam gagal di server. | Final sudah memeriksa masa berlaku di server; yang belum ada adalah kewajiban menampilkannya — approver tidak boleh dikecoh tombol yang tampak aktif. | §12/§16 | T126 — approval kedaluwarsa tampil sebagai kedaluwarsa dan tidak dapat dieksekusi. |
| W4-05 | **Mode baca auditor.** Auditor independen mendapat tampilan hanya-baca lintas tenant yang BERTANDA VISUAL JELAS ("Mode audit — semua akses tercatat"). | Mencegah kebingungan peran yang paling berbahaya: auditor yang mengira ia operator (atau sebaliknya) — padahal auditor melapor ke super admin, bukan ke CEO, dan tidak boleh bertindak sebagai operator. | §6/§16 | T127 — mode audit hanya-baca, bertanda visual, dan semua aksesnya tercatat. |
| W4-06 | **Menu dan aksi yang tidak diizinkan untuk principal yang login tidak tampil sama sekali di sidebar** (bukan tampil lalu ditolak saat diklik). | Penerapan pola "alat tak terlihat" (gelombang 3, sudah diterapkan untuk tool) ke permukaan UI — permukaan yang tidak terlihat tidak bisa menjadi sasaran injeksi maupun klik yang salah. | §16 | T128 — menu di luar izin principal tidak dirender sama sekali. |
| W4-07 | **Command palette hanya navigasi dan pembuka draf.** Palette (Cmd/Ctrl+K) mencari kasus/approval/skill/dokumen dan membuka draf — TIDAK mengeksekusi aksi mengikat dari palette. | Jalan pintas adalah gerbang bayangan: bila palette bisa mengeksekusi, kartu approval lengkap bisa dilewati. | §16 | T129 — tidak ada aksi mengikat yang dapat dieksekusi dari command palette. |
| W4-08 | **Target aksesibilitas WCAG 2.2 AA untuk dashboard.** Kontras teks 4.5:1 (teks besar 3:1, komponen bermakna 3:1); target sentuh 24×24 CSS px (2.5.8); indikator fokus selalu terlihat (2.4.11); alternatif klik untuk semua aksi seret (2.5.7); bantuan konsisten (3.2.6); tanpa entri berulang (3.3.7); autentikasi aksesibel tanpa teka-teki kognitif tanpa alternatif (3.3.8); status bukan hanya warna (1.4.1); teks bisa dibesarkan 200% dan reflow satu kolom pada 320px (1.4.4, 1.4.10). | Gerbang manusia yang tidak bisa dipakai oleh sebagian manusia bukan gerbang — ia adalah penghalang. Kriteria WCAG-nya terverifikasi; kepatuhan dashboard belum diperiksa (belum ada implementasi). | §16 (baru: subbagian aksesibilitas) | T130 — audit axe tanpa pelanggaran kritis + uji manual kriteria di atas (masuk Fase 3). |
| W4-09 | **State kosong dan state error dashboard diatur.** Daftar kosong menjelaskan kenapa kosong dan apa langkah berikutnya; pesan error berbahasa manusia + apa yang bisa dilakukan pengguna + ID insiden (trace ID); TIDAK PERNAH menampilkan stack trace, secret, atau isi prompt. | Daftar kosong yang bisu dan error yang membocorkan isi adalah dua kegagalan dashboard yang paling sering lolos karena dianggap "kosmetik". | §16 | T131 — state kosong informatif; state error tanpa stack trace/secret/prompt. |
| W4-10 | **Kartu status tugas: SUBMITTED tampil eksplisit + tanpa tombol kirim ulang; lease kedaluwarsa tampil sebagai peringatan.** Teks: "hasil eksternal belum diketahui — sedang direkonsiliasi". | Aturan server "SUBMITTED tidak boleh di-submit ulang" sudah ada di final; yang belum ada adalah kewajiban menampilkannya — operator yang tidak tahu statusnya akan menekan tombol yang salah. | §13/§16 | T132 — kartu SUBMITTED tanpa tombol kirim ulang; lease kedaluwarsa tampil sebagai peringatan. |

\* Penomoran uji T123+ adalah usulan penomoran lanjutan (final berakhir di T122) — nomor final ditetapkan saat Seti menyetujui usulan ini, agar tidak bertabrakan dengan penomoran lain.

### 12.B Presisi UI atas aturan yang SUDAH ADA (bukan aturan baru — dicatat agar tidak diklaim sebagai temuan baru)

- **Penolakan adalah keputusan yang tercatat** (alasan dipilih + teks bebas → AuditEvent): cabang penolakan dan AuditEvent sudah ada di final (uji T74 menyinggung cabang penolakan); gelombang 4 hanya menambahkan presisi bahwa alasan penolakan harus tampil dan tercatat di kartu — tidak perlu aturan gerbang baru.
- **Tidak ada aksi dari notifikasi** (tautan email/chat hanya membuka kartu lengkap; tidak ada tombol "setujui dari email"): sudah tercakup substansinya di final §19.7 ("notifikasi menyampaikan status; ia bukan persetujuan atau perintah eksekusi"); gelombang 4 hanya menegaskan presisi UI-nya.
- **Konfirmasi kedua untuk aksi dana** (dialog mengulang nominal + objek): penegasan perilaku kartu approval atas aturan approval dana yang sudah ada; bukan gerbang tambahan.
- **Token desain 3 lapis, Inter Tight, ikon SVG garis tipis, status selalu ikon + teks:** spesifikasi implementasi §16, bukan aturan gerbang — nilai token adalah usulan dan wajib lolos uji kontras sebelum disahkan.

### 12.C Pernyataan eksplisit yang diminta

Gelombang 4 MENGHASILKAN usulan blueprint baru — daftar di 12.A di atas — sehingga pernyataan "tidak ada usulan baru" tidak berlaku untuk sintesis ini. Sebaliknya, yang tidak menghasilkan usulan blueprint adalah bagian struktur repo gelombang 4 (tree, ADR, CI/CD, roadmap): semuanya sudah ditampung di dokumen ini sebagai struktur repo dan tidak memerlukan perubahan teks blueprint.

## 13. Keputusan terbuka dan hal yang masih PERLU_VERIFIKASI (dibawa, bukan diselesaikan, oleh sintesis ini)

**Keputusan terbuka untuk Seti (dari blueprint final §21 + gelombang 4):** stack bahasa/framework tiap service; IdP; database konkret; penyedia hosting/CI; mesin durable execution (ADR-002); kandidat produk (Keycloak, SPIFFE/SPIRE, OpenBao, Steel, platform durable execution, backend observabilitas) — semuanya ditulis sebagai kandidat/pola, bukan keputusan produk; angka anggaran per tugas dan durasi lease (ditetapkan per perusahaan saat aktivasi, bukan di blueprint); rute fallback kedaluwarsa approval per gerbang (eskalasi ke siapa, dalam berapa lama); nilai token desain final (wajib lolos uji kontras); lokasi tunggal SECURITY.md/CONTRIBUTING.md sudah diputuskan di dokumen ini (root) — bila Seti lebih suka di `docs/`, itu satu-satunya penyesuaian editorial yang diminta.
**Masih PERLU_VERIFIKASI / BELUM_DIPERIKSA (label dipertahankan dari gelombang 1–4 dan changelog final; tidak ada yang dinaikkan tanpa bukti primer):** status ASVS 5.0.0 sebagai rilis stabil terbaru (Mei 2025); angka "38 kontrol Annex A" ISO 42001; rekonsiliasi level SLSA L4; teks primer RFC 9700, SP 800-218, CSF 2.0, spesifikasi MCP 2026-07-28 dan A2A v1.0 (termasuk tanggal rilis v1.0, 12 Maret 2026); pemetaan CVE per risiko ASI; nomor kontrol ISO 27001 spesifik yang dikutip; teks primer kriteria WCAG 2.2; batas fitur push protection/ruleset GitHub untuk repo publik; detail versi platform durable execution (DBOS Java v1.0 20 Juli 2026, integrasi OpenAI Agents SDK September 2025); angka studi MCP Juli 2026 (~92% server tanpa OAuth, 687 instansi tool shell); detail produk pola memori (CopilotKit, TencentDB); implementasi Debezium/CDC untuk outbox audit; nomor/naskah/tanggal pengundangan/ketentuan peralihan UU Pelindungan Ketenagakerjaan (masih di Setneg per 21 Okt 2026 — disetujui DPR 6 Okt 2026, 23 bab/313 pasal, BELUM diundangkan, BELUM bernomor, BELUM berlaku; frasa "disahkan" tanpa kualifikasi tidak boleh dipakai); rincian pasal PP 33/2026 (diundangkan 16 Juli 2026, LN 2026 No. 88, 12 bab/225 pasal; berlaku 16 Januari 2027 menurut mayoritas sumber vs 15 Januari 2027 menurut satu sumber); rincian pasal UU 2/2026, Permenaker 11/2026 dan 15/2026, dampak pasal-per-pasal PP 49/2025 atas PP 51/2023, klaim pencabutan formal PP 51/2023 (hanya pernyataan politik; JDIH masih Berlaku); kerangka agen Tiongkok 15 Juli 2026 dan tanggal penerapan Omnibus EU AI Act; kepatuhan aksesibilitas dashboard (belum ada implementasi untuk diuji); konfigurasi CI/ruleset konkret (belum diterapkan — tugas Fase 0).
**Yang SUDAH terverifikasi primer dan berubah statusnya di final (gelombang 2, agar tidak diragukan lagi):** Permenaker 7/2026 (30 Apr 2026, BN 2026 No. 281, Berlaku, 6 bidang, transisi s.d. 30 Apr 2028 — ketegangan catatan lama vs addendum TERSELESAIKAN); Permenaker 11/2026 (3 Juli 2026, BN 2026 No. 438, mencabut 33/2016 dan 1/2020) + amendemen Permenaker 15/2026 (8/12 Okt 2026, BN 2026 No. 681, tenggat 120 hari dan 14 hari kerja) sebagai entri baru; UU 2/2026 (30 Apr 2026, LN 2026 (47), TLN 7174, Berlaku, penyesuaian P3RT s.d. 30 Apr 2027); PP 49/2025 (17 Des 2025, LN 2025 (187), TLN 7148, Berlaku) sebagai entri baru — catatan "UMP 2027 masih mengacu PP 51/2023" dinyatakan kedaluwarsa.

---

## Penyerahan dan pertanyaan untuk Seti

1. **Struktur repo di atas — bolehkah di-push ke `sktyawan/AgenticAI`?** Yang diminta persetujuannya adalah struktur ini (tree Bagian 2 + ADR proposed + gerbang Fase 0), BUKAN kode implementasi — kode ditulis setelah struktur disetujui, mulai dari Fase 0 (ruleset, CODEOWNERS, secret scanning, pipeline CI dasar) sebelum kode bisnis apa pun.
2. **Usulan gelombang 4 (Bagian 12.A, W4-01 s.d. W4-10) — manakah yang disetujui untuk diterapkan ke blueprint final sebagai perubahan berikutnya?** Semuanya berstatus USULAN; tidak satu pun diterapkan sampai Seti memutuskan. Penomoran uji T123+ menyusul persetujuan.
3. **Penegasan:** blueprint final sudah ada (98 halaman, §23 Changelog: 108 diterapkan, 1 ditunda [PA-Eropa], 0 konflik) dan TIDAK DIUBAH oleh sintesis ini. Konsolidasi v1.2 juga tidak diubah. Satu-satunya file baru dari sintesis ini adalah dokumen ini.

**Lampiran dokumen ini:** file lengkap ini sendiri (`struktur-repo-agenticai.md`) — ringkasan chat di bawah ini bukan pengganti dokumen.
