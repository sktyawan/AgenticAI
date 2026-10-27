# CRUD Bergating

CRUD adalah kemampuan inti AgenticAI — tapi **bukan CRUD mentah**. Setiap operasi
create, read, update, dan delete tunduk pada aturan yang sama dengan dashboard:
autentikasi, isolasi perusahaan, dan jejak audit.

## Objek yang punya CRUD

Company, OrganizationalUnit, Principal, AgentPosition, AuthorityGrant, Case,
StageExecution, Skill, SkillVersion, SkillAssignment, MemoryEntry, SoulVersion,
ControlledDocumentVersion, RegulationVersion, ApprovalRequest, ActionIntent,
AuditFinding. (Daftar mengikuti kontrak data blueprint.)

## Aturan

1. **Autentikasi & isolasi**: semua operasi lewat API terautentikasi. `company_id`
   diturunkan server dari identitas dan mandat — nilai dari pemanggil hanya divalidasi,
   bukan dipercaya. Read dibatasi lingkup mandat/tahap; tidak lintas perusahaan.
2. **Idempotensi & konkurensi**: mutasi (create/update/delete) wajib memakai
   Idempotency-Key. Update memakai If-Match terhadap revision; ketidakcocokan → 409.
3. **Jejak audit**: setiap mutasi menghasilkan AuditEvent (actor, objek, revisi,
   hash bukti). Tidak ada jalan untuk bermutasi tanpa jejak.
4. **Delete = soft-delete secara default**: status menjadi ditarik/diarsipkan, riwayat
   tetap tersedia. Hard delete / pemusnahan hanya lewat gerbang manusia dan aturan retensi.
5. **AuditEvent tidak punya delete**: endpoint audit hanya read-only. Tidak ada operasi
   untuk mengubah atau menghapus event audit lewat API.
6. **Bukan CRUD biasa**: operasi yang mengubah keadaan pihak luar (eksekusi, pembayaran,
   pengajuan resmi, pengikatan kontrak) bukan CRUD — lewat action intent + approval
   manusia sesuai matriks gerbang.
7. **Batas kelas principal agent**: agent boleh mengusulkan dan membaca dalam mandatnya;
   tidak boleh menyetujui atas nama manusia, menghapus, atau menaikkan haknya sendiri.

## Pola endpoint (`/api/v1`)

| Operasi | Pola | Catatan |
| --- | --- | --- |
| List | `GET /resources` | Filter per perusahaan & mandat |
| Detail | `GET /resources/{id}` | Termasuk versi & riwayat |
| Create | `POST /resources` | Idempotency-Key wajib |
| Update | `PATCH /resources/{id}` | If-Match wajib |
| Delete | `DELETE /resources/{id}` | Soft-delete / pengajuan penghapusan |
| Audit | `GET /audit-events` | Read-only, tanpa update/delete |

## Uji terkait (blueprint)

T12/T31/T35 (isolasi perusahaan), T26 (idempotensi), T10/T27 (konkurensi/lease),
T18 (integritas audit), T25 (kelas principal ditolak di gerbang manusia).
