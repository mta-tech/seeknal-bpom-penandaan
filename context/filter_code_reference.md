#FILTER CODE REFERENCE - VERIFIED CODE ANCHORS FOR BPOM PENANDAAN#

> **v1.0 — 2026-08-11.** Every categorical filter must resolve to an exact string
> from this file. If a user's term does not match, use the resolution path in §0
> to find the correct code. Never guess or improvise a filter value.

## §0 Resolution Path

| Step | Action | When |
|---|---|---|
| **P1 — Anchor** | Exact match from §1-§6 tables below. | User uses the canonical term. |
| **P2 — Category listing** | `SELECT DISTINCT column FROM table WHERE ...` to enumerate available values. | User uses a vague/broad term. |
| **P3 — Scoped label** | `ILIKE` search on a code or label column. | User provides a partial match. |
| **P4 — Segment discovery** | Check if the concept maps to a known segment in §6. | User asks about a product segment. |
| **P5 — Ask user** | `request_clarification` when no code resolves. | Last resort. |

## §1 Komoditi (8 values, exact strings)

| Code | n | % | MK% | TMK% | VP% | TMK MAYOR% | TMK MINOR% |
|---|---|---|---|---|---|---|---|
| `KOSMETIKA` | 78.432 | 26,8% | 80,2% | 14,7% | 5,1% | 0% | 0% |
| `PRODUK PANGAN` | 75.367 | 25,8% | 22,3% | 0% | 71,5% | 3,1% | 3,0% |
| `OBAT` | 53.034 | 18,2% | 9,5% | 90,5% | 0% | 0% | 0% |
| `OBAT TRADISIONAL (OT)` | 39.138 | 13,4% | 88,2% | 10,3% | 1,5% | 0% | 0% |
| `ROKOK` | 32.078 | 11,0% | 62,3% | 37,7% | 0% | 0% | 0% |
| `SUPLEMEN KESEHATAN` | 10.745 | 3,7% | 93,0% | 5,8% | 1,3% | 0% | 0% |
| `OBAT KUASI` | 2.686 | 0,9% | 92,7% | 5,8% | 1,5% | 0% | 0% |
| `KEMASAN PANGAN` | 689 | 0,2% | 0,3% | 0% | 99,7% | 0% | 0% |

> **Case-sensitive.** Use exact UPPER strings. `komoditi = 'KOSMETIKA'` works.
> For joins with `target_balai`, use `LOWER(komoditi)` — target uses Title Case.

## §2 Kesimpulan Penilaian Pusat (5 values, FINAL decision)

| Code | Meaning | n | % | Allowed komoditi |
|---|---|---|---|---|
| `MK` | Memenuhi Ketentuan | 151.765 | 51,9% | all 8 |
| `TMK` | Tidak Memenuhi Ketentuan | 76.397 | 26,1% | 6 (excl. PANGAN, KEMASAN) |
| `VP` | Verifikasi Produk | 59.404 | 20,3% | 6 (excl. OBAT, ROKOK) |
| `TMK MAYOR` | Tidak Memenuhi — Mayor | 2.367 | 0,8% | PANGAN only |
| `TMK MINOR` | Tidak Memenuhi — Minor | 2.236 | 0,8% | PANGAN only |

> **Filter example:** `kesimpulan_penilaian_pusat = 'MK'` → compliant.
> `kesimpulan_penilaian_pusat IN ('TMK','TMK MAYOR','TMK MINOR')` → non-compliant.
> `kesimpulan_penilaian_pusat = 'VP'` → verification needed (NOT rejection).

## §3 Kesimpulan Penilaian Balai (6 values, NORMALISED)

| Code | n | Apply normalisation? |
|---|---|---|
| `MK` | 177.080 | No |
| *(empty)* | 85.113 | No — balai did not assess |
| `TMK` | 14.113 | No |
| `TMK MINOR` | 7.407 | No (after normalisation) |
| `TMK MAYOR` | 6.592 | No |
| `TMK Minor` | 1.864 | **YES → normalise to `TMK MINOR`** |

> **Always normalise before use:**
> ```sql
> CASE WHEN kesimpulan_penilaian_balai = 'TMK Minor'
>      THEN 'TMK MINOR'
>      ELSE kesimpulan_penilaian_balai
> END AS kesimpulan_penilaian_balai_norm
> ```

## §4 Status Code (workflow) — 19 values

### §4a — Operational codes (0-7, 12, 14)

| Code | Label | n | % |
|---|---|---|---|
| `0` | Operator - Draft Sampling | 575.785 | 16,3% |
| `1` | Supervisor - Verifikasi | 517.981 | 14,7% |
| `2` | Supervisor 2 - Verifikasi | 56.841 | 1,6% |
| `3` | TPS - Penerimaan SPU | 489.763 | 13,9% |
| `4` | MT - Pembuatan SPK | 535.392 | 15,2% |
| `5` | Deputi MT - Pembuatan SPK | 359.654 | 10,2% |
| `6` | Penyelia - Pembuatan SPP | 251.411 | 7,1% |
| `7` | Penguji - Entri Hasil Pengujian | 353.574 | 10,0% |
| `12` | Kepala Balai | 35 | 0,0% |
| `14` | Operator - Perbaikan Sampel | 758 | 0,0% |

### §4b — Rejection codes (990-997)

| Code | Label (trx_steps) | n | % |
|---|---|---|---|
| `990` | *(unknown)* | 1 | 0,0% |
| `991` | ditolak_spv_1 | 36.261 | 1,0% |
| `992` | ditolak_spv_2 | 78 | 0,0% |
| `993` | ditolak_kepala_balai | 636 | 0,0% |
| `994` | ditolak_pusat | 1.460 | 0,0% |
| `995` | ditolak_spv_1_pusat | 1.415 | 0,0% |
| `996` | ditolak_spv_2_pusat | 63 | 0,0% |
| `997` | ditolak_direktur | 43 | 0,0% |

### §4c — Completion

| Code | Label | n | % |
|---|---|---|---|
| `999` | Sampel Rujukan Selesai | 350.495 | 9,9% |

> **Use `status_code`, NOT `trx_steps`.** `trx_steps` labels are misaligned with
> the actual workflow stage — e.g., `trx_steps='direktur'` maps to
> `status_label='Penguji - Entri Hasil Pengujian'`.

## §5 Balai POM (83 active entities)

### §5a — Tipe balai

| Tipe | Count | Volume range | Example |
|---|---|---|---|
| BALAI BESAR POM | ~45 | 1.000-11.384 | BBPOM SURABAYA (11.384), BBPOM BANDUNG (11.348) |
| BALAI POM | ~25 | 62-6.298 | BALAI POM AMBON (6.298), BALAI POM JAMBI (6.083) |
| LOKA POM | ~13 | 54-2.116 | LOKA POM ACEH SELATAN (2.116), LOKA POM GUNUNGSITOLI (54) |

### §5b — Top 10 balai (volume)

| nama_balai | n | % |
|---|---|---|
| BALAI BESAR POM DI SURABAYA | 11.384 | 3,9% |
| BALAI BESAR POM DI BANDUNG | 11.348 | 3,9% |
| BALAI BESAR POM DI JAKARTA | 11.191 | 3,8% |
| BALAI BESAR POM DI YOGYAKARTA | 10.226 | 3,5% |
| BALAI BESAR POM DI SEMARANG | 9.937 | 3,4% |
| BALAI BESAR POM DI DENPASAR | 9.929 | 3,4% |
| BALAI BESAR POM DI PEKANBARU | 8.221 | 2,8% |
| BALAI BESAR POM DI PADANG | 7.916 | 2,7% |
| BALAI BESAR POM DI MAKASSAR | 7.823 | 2,7% |
| BALAI BESAR POM DI BANJARBARU | 7.697 | 2,6% |

> **1 unnamed balai** (1 row) — `nama_balai = ''`. Exclude from balai analysis.

### §5c — Coverage mapping

| Relationship | Count | Join condition |
|---|---|---|
| mv_penandaan → coverage_balai | 83/83 (100%) | `mv_penandaan.nama_balai = coverage_balai.nama_balai` |
| mv_penandaan → target_balai | 54/83 (65%) | Same + `LOWER(komoditi)` for komoditi match |
| coverage_balai balai count | 88 | 5 more than fact (inactive balai) |

## §6 Direktorat Pusat (5 entities, from log)

| nama_balai | n | Sampel | Komoditi coverage |
|---|---|---|---|
| Direktorat Pengawasan KMEI ONPPZA | 589.831 | 148.396 | Obat, Pangan, Rokok, Kosmetik |
| Direktorat Pengawasan Kosmetik | 372.709 | 124.983 | Kosmetik |
| Direktorat Pengawasan OTSK | 243.923 | 89.307 | Obat Tradisional, Suplemen |
| Direktorat Pengawasan Peredaran Pangan Olahan | 2.519 | 2.519 | Pangan |
| Direktorat Pengawasan Distribusi dan Pelayanan ONPP | 1.928 | 1.350 | Pangan |

> These appear in `mv_penandaan_log.nama_balai` but NOT in `mv_penandaan.nama_balai`.
> They represent the pusat-level directorates that process penandaan.

## §7 Nomor Surat Prefix Patterns

| Prefix | n | Balai | Komoditi | Notes |
|---|---|---|---|---|
| `-` | 7.825 | 69 | 8 | Draft/unassigned surat |
| `PW.01.01.3A.` | 4.558 | 67 | 8 | Directorate 01, Bidang 01 |
| `PW.03.12.20A` | 4.195 | 60 | 8 | Directorate 03, Bidang 12 |
| `PW.04.03.*` | ~15.000 | 2-3 | 3-6 | **VP batch** — specific to VP products |

> Pattern: `PW.{DD}.{BB}.{SS}{A/B}.{MM}.{YY}.{SEQ}` where DD=directorate, BB=bidang, MM=month, YY=year.

## §8 Nomorsampel Prefix Patterns

| Prefix | n | Balai | Pattern |
|---|---|---|---|
| `YY.BBB.` | varies | 1 | First 2 digits = year, next 3 = balai code |

> The nomorsampel encodes the balai code in its structure. Each prefix maps
> to exactly 1 balai. Used for cross-validation against `nama_balai`.

## §9 Regulation Codes (in catatan)

| Code | Meaning | n | Context |
|---|---|---|---|
| `MKL.02` | Memenuhi Ketentuan Label Pasal 2 | 2.948 | VP kosmetik — claims need supporting data |
| `TME.09` | Tidak Memenuhi Edar Pasal 9 | ~14 | Expiry date format issues |
| `TME.11` | Tidak Memenuhi Edar Pasal 11 | ~23 | Missing 2D barcode |
| `NIK.01` | Nomor Izin Kemasan Pasal 1 | 61 | Packaging issues |
| `TIE` | Trade in Export | 834 | Product not registered |

## §10 Keyword → Code Mapping

| User keyword | Code | Column |
|---|---|---|
| "memenuhi", "sesuai", "lulus" | `MK` | kesimpulan_penilaian_pusat |
| "tidak memenuhi", "tidak sesuai", "gagal" | `TMK` | kesimpulan_penilaian_pusat |
| "verifikasi", "perlu cek" | `VP` | kesimpulan_penilaian_pusat |
| "pelanggaran berat" | `TMK MAYOR` | kesimpulan_penilaian_pusat |
| "pelanggaran ringan" | `TMK MINOR` | kesimpulan_penilaian_pusat |
| "kosmetik", "krim", "serum" | `KOSMETIKA` | komoditi |
| "pangan", "makanan", "minuman" | `PRODUK PANGAN` | komoditi |
| "obat", "farmasi" | `OBAT` | komoditi |
| "jamu", "herbal" | `OBAT TRADISIONAL (OT)` | komoditi |
| "rokok", "tembakau" | `ROKOK` | komoditi |
| "suplemen", "vitamin" | `SUPLEMEN KESEHATAN` | komoditi |
| "kemasan" | `KEMASAN PANGAN` | komoditi |
| "selesai", "finished" | `status = 999` | timeline |
| "draft", "baru" | `status = 0` | timeline |
