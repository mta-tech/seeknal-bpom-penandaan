# 05 — Workflow State Machine

> Status codes, trx_steps (edge vs node), piramida user, dan **PANGAN dead-end**.

---

## Status Code Kanonik (19 nilai)

Di-encode di `mv_penandaan_log.status_code` dan `mv_penandaan_timeline.status`.

```
[0]   Operator - Draft Sampling
 ↓
[1]   Supervisor - Verifikasi
 ↓
[2]   Supervisor 2 - Verifikasi (opsional)
 ↓
[3]   TPS - Penerimaan SPU (Kepala Balai receives)
 ↓
[4]   MT - Pembuatan SPK (Central team creates Surat Perintah Kerja)
 ↓
[5]   Deputi MT - Pembuatan SPK
 ↓
[6]   Penyelia - Pembuatan SPP
 ↓
[7]   Penguji - Entri Hasil Pengujian
 ↓
[999] Sampel Rujukan Selesai

Reject branches (8 kode):
[990] (label kosong)             1 event
[991] ditolak_spv_1         36.261 event  ← reject terbesar
[992] ditolak_spv_2             78
[993] ditolak_kepala_balai     636
[994] ditolak_pusat          1.460
[995] ditolak_spv_1_pusat   1.415
[996] ditolak_spv_2_pusat       63
[997] ditolak_direktur          43

Status langka lain:
[12]  Kepala Balai                35 event
[14]  Operator - Perbaikan Sampel 758 event
```

---

## 🔴 PANGAN Dead-End — Workflow Jalan Buntu

**Perbandingan pipeline event-level** (OBAT/ROKOK/KOSMETIKA vs PRODUK PANGAN):

| Step | OBAT+ROKOK+KOSMETIKA | PRODUK PANGAN |
|---|---|---|
| 0 Draft | 163.740 sampel | 75.560 |
| 1 Supervisor | 163.425 | 75.524 |
| 3 TPS/Kabala | 163.740 | 75.560 |
| 4 MT SPK | 163.740 | 75.560 |
| 5 Deputi | 161.314 | **168** ⚠️ |
| 6 Penyelia | 79.806 | **0** ❌ |
| 7 Penguji | 160.124 | **0** ❌ |
| **999 Selesai** | **158.428** | **0** ❌ |

**PANGAN terputus total setelah status 4.** Hanya 168 dari 75.560 sampai Deputi, dan **NOL** sampai Penyelia/Penguji/Selesai.

**Anatomi PANGAN (75.560 sampel):**
- 75.392 (99,8%) mandek di status 4
- 168 (0,2%) di status 5
- **0 di status 999**
- TAPI kesimpulan_pusat ADA: VP=54.117, MK=16.840

**Paradoks:** PANGAN **sudah dinilai** (kesimpulan_pusat terisi) tapi workflow **tidak pernah ditutup** (status macet di 4). Keputusan dibuat, proses tidak diakhiri. Ini **diskonek antara assessment dan closure**.

Direktorat Pengawasan Peredaran Pangan Olahan: **0 completion** (2.519 event diproses, tak satu pun selesai).

> Lihat detail lengkap di [07_temuan_kritis.md](07_temuan_kritis.md) §temuan-1.

---

## trx_steps vs status_code — Dua Sistem yang Tidak Boleh Dicampur

Log punya 2 kolom yang sering dianggap sinkron tapi **mengukur hal berbeda**:

| Aspek | `status_code` | `trx_steps` |
|---|---|---|
| Makna | **Node/state** tujuan ("sedang diproses siapa") | **Edge/aksi** ("dari siapa dikirim") |
| Kanonik | ✅ Ya | ❌ Bayangan |
| Values | 19 bersih | 18 + typo |
| Rekomendasi | **GUNAKAN INI** untuk analisis state | Untuk analisis "aksi terakhir siapa" |

**Bukti offset (min-max status_code per trx_steps):**
- `trx_steps='draft'` → status 0-990 (edge draft menghasilkan state draft ATAU reject)
- `trx_steps='selesai'` → status 3-999 (edge selesai bisa menghasilkan state 999 ATAU state 3)
- `trx_steps='ditolak_spv_1'` → status 991

Satu trx_step bisa menghasilkan beberapa state. **Jangan campur** keduanya.

---

## Piramida Organisasi (dari jumlah user per status)

| status | label | #user unik | event/user | Karakter |
|---|---|---|---|---|
| 0 | Draft | 1.428 | 403 | Banyak operator daerah |
| 1 | Supervisor | 1.423 | 364 | Banyak supervisor daerah |
| 3 | Kepala Balai | 739 | 663 | Lebih sedikit |
| 4 | MT | 281 | 1.905 | Tim pusat |
| 5 | Deputi | 93 | 3.867 | Makin sempit |
| 6 | Penyelia | 30 | 8.380 ⚠ | Spesialis |
| 7 | Penguji | 40 | 8.839 ⚠ | Spesialis lab |
| **999** | **Selesai** | **22** | **15.931** ⚠⚠ | **Approver final — sangat sedikit** |

**Pola piramida:** user menyempit seiring workflow naik (1.428 → 22). Beban per orang meledak di puncak. **Bottleneck struktural di puncak.**

---

## Reject Loop — Bukti Rework

- Draft: 576.236 event untuk 500.717 sampel unik → **75.519 event ekstra** = draft ulang
- `ditolak_spv_1` (991): 36.261 event, 29.837 sampel unik → **5,96% sampel pernah ditolak spv_1**

**Titik gagal terbesar = gerbang spv_1** (supervisor daerah) — kualitas input operator adalah titik lemah. Detail di [08_proses_beban_kerja](08_proses_beban_kerja.md).

---

## Distribusi Status Akhir (timeline, 500.717 sampel)

| status | n | % | Keterangan |
|---|---|---|---|
| **999** | 350.036 | 69,9% | Tuntas |
| **4** | 119.637 | 23,9% | **LIMBO** (95,4% = PANGAN!) |
| 0 | 17.274 | 3,5% | Draft belum jalan |
| 5 | 3.818 | 0,8% | |
| 1 | 3.067 | 0,6% | |
| 7 | 2.831 | 0,6% | |
| 3 | 1.358 | 0,3% | |
| 994 | 869 | 0,2% | Ditolak pusat |
| 2 | 842 | 0,2% | |
| 991 | 525 | 0,1% | Ditolak spv_1 |
| 995 | 180 | | |
| 993 | 147 | | Ditolak kabalai |

**Komposisi status 4 (limbo) per komoditi:**

| komoditi | n_status4 | % dari total status4 |
|---|---|---|
| **PRODUK PANGAN** | **75.392** | **95,4%** |
| KOSMETIKA | 1.802 | 2,3% |
| KEMASAN PANGAN | 689 | 0,9% |
| ROKOK | 556 | 0,7% |
| OT | 359 | 0,5% |
| OBAT | 108 | 0,1% |
| SUPLEMEN | 72 | 0,1% |
| OBAT KUASI | 28 | 0,0% |

> **95,4% sampel yang "stuck" adalah PANGAN.** Ini bukan masalah acak — ini konsekuensi langsung dead-end PANGAN.

---

## Query Scope Convention

| Skenario | Pola SQL |
|---|---|
| Penandaan selesai | `FROM timeline WHERE status = 999` |
| Sedang berjalan | `FROM timeline WHERE status IN (0,1,2,3,4,5,6,7,12,14)` |
| Ditolak | `FROM timeline WHERE status IN (990,991,992,993,994,995,996,997)` |
| Stuck di MT | `FROM timeline WHERE status = 4` (95% = PANGAN) |
| Workflow events | `COUNT(*) FROM log` |

---

Lanjut ke [06_penilaian_keputusan.md](06_penilaian_keputusan.md).
