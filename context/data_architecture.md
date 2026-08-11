#DATA ARCHITECTURE - BPOM PENANDAAN DATABASE INVENTORY, JOIN RULES, AND TOPOLOGY#

> **v1.0 — 2026-08-11.** Single schema `public` on PostgreSQL (`penandaan` database).
> All tables are ETL-loaded physical tables with a `sync` timestamp — the `mv_*` prefix
> denotes **materialised analytics tables**, not live materialized views. No PK, FK,
> index, or view is enforced — referential integrity depends entirely on the ETL pipeline.

## §1 Table Inventory

| Table | Rows | Size | Grain | Time Range | Notes |
|---|---|---|---|---|---|
| `mv_penandaan` | 292.169 | 67 MB | 1 row = 1 sampel penandaan | 2023-01 → 2026-08 | **Fact core.** PK candidate: `id` (unique). |
| `mv_penandaan_log` | 3.531.653 | 582 MB | 1 row = 1 workflow event | 2020-03 → 2026-08 | Audit trail. 500.576 distinct `id_penandaan`. **208.407 orphan** (2020-2022, not in fact). |
| `mv_penandaan_timeline` | 500.576 | 43 MB | 1 row per `id_penandaan` | same as log | Lifecycle summary. PK candidate: `id_penandaan`. Same 208.407 orphan. |
| `mv_penandaan_agg` | 94.517 | 13 MB | Composite key: periode × komoditi × balai × kesimpulan | 2023-01 → 2026-08 | Pre-aggregated rollup. **sum(jumlah_penandaan) = 292.169 = fact total** ✅ |
| `coverage_balai` | 668 | 104 kB | 1 row = 1 balai × 1 kabupaten | static | Dimension. id_balai ↔ nama_balai **1:1**. 88 balai, 514 kabupaten. |
| `target_balai` | 532 | 112 kB | 1 row = 1 balai × 1 komoditi × 1 tahun | 2024 only | Dimension. 76 balai × 7 komoditi. **KEMASAN PANGAN absent.** |

### Schema `dimension` (proyeksi tereduksi, bukan mirror)

| Table | Rows | vs public | Kolom di-dim |
|---|---|---|---|
| `dimension.mv_penandaan` | 218.472 | 217.212 overlap | 10 kolom (tanpa id, tanggal) |
| `dimension.mv_penandaan_log` | 33.137 | subset | 5 kolom (tanpa id_penandaan, status_code) |
| `dimension.coverage_balai` | 513 | 81 balai (vs 88) | 2 kolom |
| `dimension.target_balai` | 76 | pivot | 2 kolom |

> **Use `public` as the single source of truth.** `dimension` is a stale BI export —
> never query both in the same answer.

## §2 Column Type Summary

### mv_penandaan (fact core) — all columns are `text` except dates and sync

| Column | Type | Distinct | Null/Empty | Notes |
|---|---|---|---|---|
| `id` | bigint | 292.169 | 0 | **Unique — PK candidate.** |
| `komoditi` | text | 8 | 0 | Categorical. Exact strings in `filter_code_reference.md` §1. |
| `nomor_surat` | text | 22.443 | 0 | Surat pengantar. 1 surat → avg 13 sampel. Prefix =PW.{DD}.{BB}.{SS}. |
| `nomorsampel` | text | 260.112 | 31.985 | ID sampel. Contains balai code in prefix (YY.BBB.XXX). |
| `tgl_start` | date | 1.281 | 0 | Tanggal mulai penandaan. Range: 2023-01-01 → 2026-08-10. |
| `tgl_end` | date | 1.286 | 0 | Tanggal selesai. **93,5% same-day** (median = 0). |
| `nama_produk` | text | 84.693 | 200 | Nama produk pada label. |
| `ed_nie` | date | 3.498 | 29.270 | Tanggal kedaluwarsa NIE. **31 outlier** (1026-2929). |
| `pendaftar` | text | 4.314 | 116.232 | Pihak pendaftar. 40% kosong. |
| `produsen` | text | 20.480 | 80.202 | Produsen/manufaktur. 27% kosong. Double-spasi issues. |
| `nama_balai` | text | 83 | 1 | Balai POM. **100% match** ke coverage_balai. |
| `kesimpulan_penilaian_balai` | text | 6 | 85.113 | Keputusan balai. **Casing dup**: "TMK Minor" vs "TMK MINOR". |
| `kesimpulan_penilaian_pusat` | text | 5 | 0 | Keputusan pusat (FINAL). 0 null. |
| `catatan` | text | 27.087 | 179.841 | Catatan alasan. 62% kosong. Contains regulation codes (MKL.02, TME.09). |
| `sync` | timestamp | 1 | 0 | Last ETL sync. All rows = 2026-08-10. |

### mv_penandaan_log (audit fact)

| Column | Type | Notes |
|---|---|---|
| `id_penandaan` | bigint | FK to `mv_penandaan.id` (implicit). 208.407 orphan. |
| `trx_steps` | text | **Miss-aligned** with status_label — do NOT use for analysis. |
| `status_code` | bigint | **Canonical.** 19 values (11 operational + 8 reject). Use this. |
| `status_label` | text | Aligned with status_code. 39.964 empty. |
| `fullname` | text | Petugas. 1.803 unik. Each → 1 balai. |
| `nama_balai` | text | 91 entities (83 balai + 5 direktorat pusat + 3 lainnya). |
| `catatan` | text | 45% non-empty. Workflow notes. |
| `tanggal_proses` | timestamp | 8,4% null. Range: 2020-03 → 2026-08. |

### mv_penandaan_timeline (lifecycle summary)

| Column | Type | Null % | Notes |
|---|---|---|---|
| `id_penandaan` | bigint | 0 | **Unique — PK candidate.** |
| `tgl_start` / `tgl_end` | date | 0,003% | Matches fact dates. |
| `tanggal_kirim_kabalai` | date | 4,4% | |
| `tanggal_kirim_direktur` | date | 29,5% | Many null = not reached this stage. |
| `tanggal_kirim_pusat` | date | 4,7% | |
| `status` | bigint | 0 | Same code space as log `status_code`. |
| `mulai_kabalai` | integer | 4,4% | Days: start → kirim kabalai. Median 6, max 1475. |
| `kabalai_direktur` | integer | 29,5% | Days: kabalai → direktur. **BOTTLENECK.** Median 21, max 996. |
| `direktur_pusat` | integer | 29,5% | Days: direktur → pusat. Median 0 (instant). |

### mv_penandaan_agg (pre-aggregated rollup)

| Column | Type | Notes |
|---|---|---|
| `periode_type` | text | `day` (63.830) or `month` (30.687). |
| `tanggal_periode` | date | Period date. |
| `komoditi` | text | Same 8 values as fact. |
| `nama_balai` | text | 83 balai. |
| `kesimpulan_penilaian_balai` | text | 6 values, 26% null. |
| `kesimpulan_penilaian_pusat` | text | 5 values. |
| `jumlah_penandaan` | bigint | Count of penandaan. NOT NULL. |
| `jumlah_surat_unik` | bigint | Distinct nomor_surat. |
| `jumlah_sampel_unik` | bigint | Distinct nomorsampel. |
| `jumlah_produk_unik` | bigint | Distinct nama_produk. |
| `avg_durasi_hari` | double precision | Average tgl_end − tgl_start. |
| `min_durasi_hari` / `max_durasi_hari` | integer | Min/max duration. |
| `last_updated` | timestamp | ETL timestamp. |

## §3 Join Rules (Implicit — No FK Enforced)

```
mv_penandaan.id  ──1:N──▶  mv_penandaan_log.id_penandaan
                            (208.407 orphan from 2020-2022 not in fact)

mv_penandaan.id  ──1:1──▶  mv_penandaan_timeline.id_penandaan
                            (same 208.407 orphan)

mv_penandaan.nama_balai  ──N:1──▶  coverage_balai.nama_balai
                                   (83/83 = 100% match)

mv_penandaan.nama_balai  ──N:1──▶  target_balai.nama_balai
                                   (54/83 = 65% match — 29 balai have no 2024 target)

mv_penandaan.komoditi  ──N:1──▶  target_balai.komoditi
                                  (case mismatch: fact=UPPER, target=Title — use LOWER())
```

> **Orphan policy:** When joining fact→log/timeline, decide explicitly whether to
> include orphan events (2020-2022 history) or restrict to fact-matched rows.
> A `LEFT JOIN FROM fact` excludes orphans; a `FULL JOIN` or `FROM log` includes them.
> State the choice in the answer.

## §4 ERD (Logical)

```
┌──────────────────────────────────────────────────────────────────────┐
│                        SCHEMA PUBLIC                                  │
│                                                                      │
│  ┌──────────────┐   nama_balai (83/83)   ┌──────────────────────┐  │
│  │ coverage_balai│◄──────────────────────│   mv_penandaan       │  │
│  │ (668 rows)   │   1 balai → N kab      │   (292.169 rows)     │  │
│  └──────────────┘                         │   FACT CORE          │  │
│                                           └──────┬───────────────┘  │
│  ┌──────────────┐   nama_balai+komoditi          │                   │
│  │ target_balai │◄────(54/83 match)──────────────┘                   │
│  │ (532 rows)   │   case mismatch: Title vs UPPER                    │
│  └──────────────┘                                                     │
│                                            id (PK)                   │
│                                            │                         │
│                          ┌─────────────────┼──────────────────┐      │
│                          │                 │                  │      │
│                          ▼                 ▼                  │      │
│  ┌────────────────────┐ ┌────────────────────┐ ┌────────────┐ │      │
│  │ mv_penandaan_log   │ │ mv_penandaan_      │ │ mv_penda-  │ │      │
│  │ (3.531.653 rows)   │ │ timeline           │ │ nanan_agg  │ │      │
│  │ id_penandaan→id    │ │ (500.576 rows)     │ │ (94.517)   │ │      │
│  │ 1:N (avg 7 events) │ │ id_penandaan→id    │ │ rollup     │ │      │
│  │ 208.407 orphan     │ │ 1:1                │ │ konsisten  │ │      │
│  └────────────────────┘ └────────────────────┘ └────────────┘ │      │
│                                                                │      │
└──────────────────────────────────────────────────────────────────────┘
```

## §5 Data Quality Flags

| Flag | Table | Impact |
|---|---|---|
| 208.407 orphan id_penandaan | log, timeline | 2020-2022 events not in fact. Use `FROM log` to include, `LEFT JOIN FROM fact` to exclude. |
| Casing dup "TMK Minor" vs "TMK MINOR" | fact kesimpulan_balai | Normalise: `CASE WHEN kpb = 'TMK Minor' THEN 'TMK MINOR' ELSE kpb END`. |
| Komoditi case mismatch | target vs fact | Join with `LOWER(komoditi)` or normalise target to UPPER. |
| 31 outlier ed_nie | fact | Years 1026-1747, 2102-2929. Filter: `ed_nie BETWEEN '2000-01-01' AND '2100-01-01'`. |
| trx_steps misaligned | log | **Do not use** `trx_steps` for analysis — labels do not match `status_label`. Use `status_code`. |
| Produsen double-spaces | fact | `TRIM(REGEXP_REPLACE(produsen, '\s{2,}', ' ', 'g'))`. |
| 119.445 samples stuck status=4 | timeline | Samples at MT stage — may be in-progress or abandoned. |

## §6 Query Scope Conventions

| Scope | SQL pattern |
|---|---|
| Penandaan count (samples) | `COUNT(DISTINCT id)` from `mv_penandaan` |
| Surat count | `COUNT(DISTINCT nomor_surat)` from `mv_penandaan` |
| Produk count | `COUNT(DISTINCT nama_produk)` from `mv_penandaan` |
| Sampel count | `COUNT(DISTINCT nomorsampel)` from `mv_penandaan` |
| Workflow events | `COUNT(*)` from `mv_penandaan_log` |
| Completed penandaan | `FROM mv_penandaan WHERE id IN (SELECT id_penandaan FROM mv_penandaan_timeline WHERE status=999)` |
| Active balai | `FROM mv_penandaan GROUP BY nama_balai` (83 distinct) |
| Coverage | `FROM coverage_balai` (88 balai, 514 kabupaten) |
| Target | `FROM target_balai WHERE tahun = <year>` (76 balai × 7 komoditi) |
