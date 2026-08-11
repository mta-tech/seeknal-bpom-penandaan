#WORKFLOW GUIDE — BPOM PENANDAAN PROCESS, STATUS CODES, DURATION, AND BOTTLENECK ANALYSIS#

> **v1.0 — 2026-08-11.** Complete reference for the penandaan workflow lifecycle.
> Every penandaan (product labeling inspection) passes through a multi-stage
> approval process from the regional Balai POM to the central directorate in Jakarta.

## §1 Workflow Sequence

The canonical workflow is encoded in `status_code` (NOT `trx_steps` — see §2).

```
[0] Operator - Draft Sampling
 ↓
[1] Supervisor - Verifikasi
 ↓
[2] Supervisor 2 - Verifikasi (optional, 56.841 events)
 ↓
[3] TPS - Penerimaan SPU (Kepala Balai receives)
 ↓
[4] MT - Pembuatan SPK (Central team creates Surat Perintah Kerja)
 ↓
[5] Deputi MT - Pembuatan SPK (Deputy processes)
 ↓
[6] Penyelia - Pembuatan SPP (Supervisor creates Surat Permintaan Pengujian)
 ↓
[7] Penguji - Entri Hasil Pengujian (Tester enters results)
 ↓
[999] Sampel Rujukan Selesai (Completed)

Rejection branches:
[991] ditolak_spv_1 (36.261) → back to operator
[992] ditolak_spv_2 (78)
[993] ditolak_kepala_balai (636)
[994] ditolak_pusat (1.460)
[995] ditolak_spv_1_pusat (1.415)
[996] ditolak_spv_2_pusat (63)
[997] ditolak_direktur (43)
```

## §2 status_code vs trx_steps — Use status_code

| Aspect | `status_code` | `trx_steps` |
|---|---|---|
| Alignment with status_label | ✅ Perfect | ❌ Miss-aligned |
| Canonical | ✅ Yes | ❌ No |
| Values | 19 (clean) | 18 (includes typos: `spv1`) |
| Recommendation | **USE THIS** | **Do not use** |

> **Example of misalignment:** `trx_steps='direktur'` maps to
> `status_label='Penguji - Entri Hasil Pengujian'` (353.574 events).
> `trx_steps='kepala_balai'` maps to `status_label='TPS - Penerimaan SPU'` (488.957 events).
> The `trx_steps` column appears to represent the internal system state,
> while `status_label` represents the business workflow stage.

## §3 Status Distribution (from timeline)

| status | Label | n | % | keterangan |
|---|---|---|---|---|
| **999** | Selesai | **350.036** | **69,9%** | Completed successfully |
| **4** | MT - Pembuatan SPK | **119.445** | **23,9%** | **STUCK** — not yet processed by pusat |
| 0 | Draft | 17.384 | 3,5% | Still in draft |
| 5 | Deputi MT | 3.601 | 0,7% | At deputy stage |
| 1 | Supervisor | 3.246 | 0,6% | At supervisor stage |
| 7 | Penguji | 2.810 | 0,6% | At tester stage |
| 3 | TPS | 1.365 | 0,3% | At Kepala Balai |
| 2 | Supervisor 2 | 842 | 0,2% | At second supervisor |
| 994 | Ditolak pusat | 869 | 0,2% | Rejected by pusat |
| 991 | Ditolak spv_1 | 518 | 0,1% | Rejected by supervisor |
| 993 | Ditolak kabalai | 147 | 0,0% | Rejected by Kepala Balai |
| 995 | Ditolak spv_1_pusat | 180 | 0,0% | Rejected by pusat supervisor |
| 6 | Penyelia | 100 | 0,0% | At supervisor stage |
| 992 | Ditolak spv_2 | 4 | 0,0% | |
| 8 | — | 11 | 0,0% | |
| 12 | Kepala Balai | 10 | 0,0% | |
| 14 | Perbaikan | 6 | 0,0% | |
| 996 | Ditolak spv_2_pusat | 1 | 0,0% | |
| 997 | Ditolak direktur | 1 | 0,0% | |

### §3a — Status × Kesimpulan Pusat (joined with fact)

| status | kesimpulan | n | % dalam status |
|---|---|---|---|
| 4 | VP | 56.618 | **71,8%** |
| 4 | MK | 17.318 | 22,0% |
| 4 | TMK MAYOR | 2.357 | 3,0% |
| 4 | TMK MINOR | 2.235 | 2,8% |
| 4 | TMK | 286 | 0,4% |
| 999 | MK | 131.975 | **63,1%** |
| 999 | TMK | 75.076 | 35,9% |
| 999 | VP | 2.238 | 1,1% |
| 5 | TMK | 540 | 46,6% |
| 5 | MK | 465 | 40,2% |
| 7 | MK | 1.908 | 67,9% |
| 7 | TMK | 494 | 17,6% |
| 7 | VP | 406 | 14,5% |

> **Key insight:** Status 4 (stuck at MT) is **71,8% VP** — VP products are
> the ones that get stuck at the MT stage. Status 999 (completed) has
> 63,1% MK and 35,9% TMK — this is the final outcome distribution.

## §4 Duration per Tahap (hari, dari timeline)

| Tahap | n | Min | P25 | Median | P75 | P90 | P95 | Max | Avg |
|---|---|---|---|---|---|---|---|---|---|
| `mulai_kabalai` | 478.592 | 0 | 3 | **6** | 16 | 62 | 123 | **1.475** | 21,6 |
| `kabalai_direktur` | 352.830 | 0 | 11 | **21** | 44 | 76 | 102 | **996** | 33,2 |
| `direktur_pusat` | 352.804 | 0 | 0 | **0** | 0 | 0 | 0 | 209 | 0,0 |

### §4a — Duration per Komoditi (hanya status=999/selesai)

| Komoditi | avg_hari | avg_kb | avg_kb_dir | avg_dir |
|---|---|---|---|---|
| ROKOK | 0,5 | 17,9 | 27,5 | 0,0 |
| OBAT | 2,1 | 7,3 | 17,8 | 0,0 |
| KOSMETIKA | 1,6 | 5,4 | **41,0** | 0,0 |
| OT | 1,5 | 4,9 | 11,4 | 0,1 |
| SUPLEMEN | 1,3 | 4,9 | 11,4 | 0,1 |
| OBAT KUASI | 1,3 | 5,2 | 11,1 | 0,1 |

> **Bottleneck:** `kabalai_direktur` is the longest phase (median 21 days, P95 102 days).
> KOSMETIKA has the worst bottleneck at 41 days average — due to high volume (78k samples)
> at the cosmetics directorate. `direktur_pusat` is almost always instant (median 0).

## §5 Lifecycle Trace Example

### §5a — Completed penandaan (id=393371, OBAT TRADISIONAL, KAPSIDA)

| Step | timestamp | user | balai | catatan |
|---|---|---|---|---|
| draft (0) | 2025-04-17 12:51 | Suharni, S.Si.Apt | BALAI POM DI MANOKWARI | |
| spv_1 (1) | 2025-04-25 06:58 | Suharni, S.Si.Apt | BALAI POM DI MANOKWARI | |
| kepala_balai (3) | 2025-04-25 07:04 | Sienny, S.Si, Apt | BALAI POM DI MANOKWARI | "OBA 10 sampel dibagi menjadi 2 SPU" |
| pusat (4) | 2025-04-28 16:49 | Drs. Yoseph Nahak Klau | BALAI POM DI KUPANG | |
| spv_1_pusat (5) | 2025-05-02 14:25 | Mohammad Gama Ramadhan | Direktorat Pengawasan OTSK | "mohon koreksi dan arahannya" |
| direktur (7) | 2025-05-03 18:31 | Dra. Tita Nursjafrida | Direktorat Pengawasan OTSK | "ACC" |
| selesai (999) | 2025-05-03 18:33 | Irwan, S.Si., Apt., M.KM | BBPOM SEMARANG | "OK" |

**Timeline:** mulai_kabalai=7, kabalai_direktur=6, direktur_pusat=0. Total: 16 days.

### §5b — Workflow routing pattern

| From | To | Who handles | Duration |
|---|---|---|---|
| Balai operator | Balai supervisor | Same balai | ~1 hour |
| Balai supervisor | Kepala Balai | Same balai | 1-8 days |
| Kepala Balai | Pusat MT | Different entity (central) | 2-3 days |
| Pusat MT | Deputi MT | Direktorat level | 1-42 days (BOTTLENECK) |
| Deputi MT | Penyelia | Same directorate | 1-3 days |
| Penyelia | Penguji | Same directorate | 1-7 days |
| Penguji | Selesai | Final approval | hours |

## §6 Keputusan Balai → Pusat (Override Pattern)

| Balai says | Pusat says | n | % | Interpretation |
|---|---|---|---|---|
| MK | MK | 124.467 | 70,3% | Consensus: compliant |
| MK | VP | 47.611 | 26,9% | Pusat changes MK → VP (mostly PANGAN) |
| MK | TMK | 4.893 | 2,8% | Pusat downgrades (found issues balai missed) |
| TMK | TMK | 11.276 | 79,9% | Consensus: non-compliant |
| TMK | MK | 1.919 | 13,6% | Pusat upgrades (disagreed with rejection) |
| TMK | VP | 918 | 6,5% | Changed to VP |
| TMK MAYOR | VP | 4.250 | 64,5% | **Most Mayor → VP** (verify, don't reject) |
| TMK MAYOR | TMK MAYOR | 2.266 | 34,4% | Consensus: major violation |
| TMK MINOR | VP | 5.062 | 68,3% | **Most Minor → VP** |
| TMK MINOR | TMK MINOR | 2.156 | 29,1% | Consensus: minor violation |

> **VP is the middle ground.** When balai says TMK Mayor/Minor, pusat often
> changes it to VP (64-68%) rather than outright rejection. VP means
> "needs additional verification" — NOT a rejection.

## §7 Catatan Patterns

### §7a — Top catatan in fact (mv_penandaan)

| catatan | n | Context |
|---|---|---|
| `-` | 13.522 | Placeholder |
| `ok` | 13.412 | Simple approval |
| `Penandaan memenuhi ketentuan` | 12.819 | KOSMETIKA, MK |
| `MK` | 9.325 | Just the code |
| `Penandaan MK` | 6.078 | KOSMETIKA, MK |
| `sudah sesuai` | 5.158 | Approval |
| `TIE` | 398 | 5 komoditi — Trade in Export |
| `Tidak mencantumkan 2D barcode` | 311 | 4 komoditi |
| `temuan berulang` | 107 | Repeat findings |

### §7b — Top catatan in log (mv_penandaan_log)

| catatan | n | Langkah |
|---|---|---|
| `-` | 285.567 | 10 langkah |
| `ok` | 272.468 | 11 langkah |
| `MK` | 114.085 | 12 langkah |
| `OK` | 56.749 | 9 langkah |
| `Memenuhi Ketentuan` | 47.457 | 7 langkah |
| `Kirim Perbaikan` | 33.294 | 12 langkah |
| `MEMENUHI KETENTUAN, SELESAI` | 27.006 | selesai |
| `ACC` | 24.655 | 3 langkah |
| `HASIL EVALUASI MEMENUHI KETENTUAN` | 19.605 | direktur+spv_2 |
| `Yth. Pak Gama, mohon arahan...` | 10.443 | pusat+deputi+penyelia |

### §7c — VP catatan by komoditi

| Komoditi | VP catatan pattern |
|---|---|
| KOSMETIKA | `MKL.02` + claim text (dermatologically tested, clinically tested, SPF, cruelty free) |
| PRODUK PANGAN | **No catatan** — empty. 82,9% had balai=MK. |
| KEMASAN PANGAN | **No catatan** — empty. |

## §8 Rejection Analysis

| Rejection code | n | Unique samples | Users |
|---|---|---|---|
| 991 (ditolak_spv_1) | 36.261 | 29.837 | 650 |
| 994 (ditolak_pusat) | 1.460 | 1.386 | 38 |
| 995 (ditolak_spv_1_pusat) | 1.415 | 1.393 | 15 |
| 993 (ditolak_kepala_balai) | 636 | 634 | 75 |
| 992 (ditolak_spv_2) | 78 | 72 | 12 |
| 997 (ditolak_direktur) | 43 | 43 | 6 |
| 996 (ditolak_spv_2_pusat) | 63 | 63 | 4 |
| 990 | 1 | 1 | 1 |

> **Rejection rate:** 39.957 rejections out of 3.531.653 events = **1,13%**.
> Most rejections (91%) are at spv_1 level (supervisor 1 at balai).

## §9 Timeline Distribution (tanggal_proses per tahun)

| Tahun | n | % | Sampel unik |
|---|---|---|---|
| 2020 | 63.221 | 1,8% | 17.548 |
| 2021 | 482.634 | 13,7% | 126.724 |
| 2022 | 752.600 | **21,3%** | 261.240 |
| 2023 | 636.851 | 18,0% | 192.114 |
| 2024 | 608.631 | 17,2% | 186.254 |
| 2025 | 424.023 | 12,0% | 123.399 |
| 2026 | 265.571 | 7,5% | 95.887 |
| *(null)* | 298.122 | 8,4% | — |

> **Peak activity:** 2022 (21,3%). Log starts 2020-03, fact starts 2023-01.
> This gap produces the 208.407 orphan records.

## §10 Top Users (by event count)

| fullname | n | balai |
|---|---|---|
| Mohammad Gama Ramadhan | 136.319 | Direktorat Pengawasan Kosmetik |
| Irwan, S.Si., Apt., M.KM | 106.705 | BALAI BESAR POM DI SEMARANG |
| Nova Emelda, S.Si, MS, Apt | 83.751 | BALAI BESAR POM DI MEDAN |
| Dra. Yusmeiliza, Apt | 65.483 | BALAI BESAR POM DI BANDA ACEH |
| Dra. Tri Asti Isnariani, Apt., M.Pharm | 62.855 | BALAI BESAR POM DI BANJARBARU |

> Each user belongs to exactly 1 balai (consistent assignment).

## §11 Workflow Query Patterns

| Scenario | SQL pattern |
|---|---|
| Completed penandaan only | `FROM mv_penandaan p JOIN mv_penandaan_timeline t ON t.id_penandaan = p.id WHERE t.status = 999` |
| Still in progress | `FROM mv_penandaan_timeline WHERE status IN (0,1,2,3,4,5,6,7,12,14)` |
| Rejected | `FROM mv_penandaan_timeline WHERE status IN (990,991,992,993,994,995,996,997)` |
| Stuck at MT (status=4) | `FROM mv_penandaan_timeline WHERE status = 4` |
| Average duration | `SELECT avg(mulai_kabalai), avg(kabalai_direktur), avg(direktur_pusat) FROM mv_penandaan_timeline WHERE status = 999` |
| Duration by komoditi | Join timeline with fact, GROUP BY komoditi |
| Override analysis | Join fact (balai kesimpulan) with fact (pusat kesimpulan) on id |
