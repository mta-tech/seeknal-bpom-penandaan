#PREDIKAT — RULES FOR ACCURATE COUNTING, FILTERING, AND INTERPRETATION IN BPOM PENANDAAN#

> **v1.0 — 2026-08-11.** Single source of truth for how numbers are derived.
> Every answer that carries a count must follow these rules. A number built
> outside these rules is wrong by construction, even if the SQL runs cleanly.

## §1 Counting Entity (CRITICAL)

The entity determines the `COUNT` expression. Using the wrong entity
over-counts or under-counts silently.

| Entity | Expression | Table | When to use |
|---|---|---|---|
| **Penandaan** (sample) | `COUNT(DISTINCT id)` | `mv_penandaan` | Default. "berapa penandaan", "berapa sampel", "berapa kasus". |
| **Surat** | `COUNT(DISTINCT nomor_surat)` | `mv_penandaan` | "berapa surat", "berapa pengajuan". 1 surat → avg 13 sampel. |
| **Produk** | `COUNT(DISTINCT nama_produk)` | `mv_penandaan` | "berapa produk", "berapa nama produk". |
| **Sampel unik** | `COUNT(DISTINCT nomorsampel)` | `mv_penandaan` | "berapa nomor sampel". 31.985 kosong — handle carefully. |
| **Event** | `COUNT(*)` | `mv_penandaan_log` | Workflow events, audit trail. |
| **Penandaan (agregat)** | `SUM(jumlah_penandaan)` | `mv_penandaan_agg` | Pre-aggregated rollup. |

> **BLOCK:** `COUNT(*)` over `mv_penandaan` is WRONG for penandaan count —
> there are no duplicate rows, but the entity is penandaan (id), not row.
> Use `COUNT(DISTINCT id)` unless the question explicitly asks for "jumlah
> baris/record".

## §2 Date Column Choice

| Context | Column | Table | Notes |
|---|---|---|---|
| Penandaan period | `tgl_start` | `mv_penandaan` | When the penandaan started. Primary date for fact. |
| Penandaan completion | `tgl_end` | `mv_penandaan` | When the penandaan ended. 93,5% same-day as tgl_start. |
| NIE expiry | `ed_nie` | `mv_penandaan` | When the product registration expires. 10% null, 31 outlier. |
| Workflow event | `tanggal_proses` | `mv_penandaan_log` | When the event happened. 8,4% null. |
| Aggregation period | `tanggal_periode` | `mv_penandaan_agg` | Day or month period. |

> **Never use `EXTRACT(YEAR FROM tgl_start)` to filter.** Use `tgl_start >= 'YYYY-MM-DD' AND tgl_start < 'YYYY-MM-DD'` for range filtering. Extracting year breaks index usage and produces different results for edge cases.

## §3 Kesimpulan Interpretation

### §3a — kesimpulan_penilaian_pusat (FINAL decision)

| Code | Meaning | Komoditi | Business interpretation |
|---|---|---|---|
| **MK** | Memenuhi Ketentuan | all 8 | Product label COMPLIES with regulations. Product may circulate. |
| **TMK** | Tidak Memenuhi Ketentuan | 6 (excl. PANGAN, KEMASAN) | Product label DOES NOT comply. Violations found. |
| **VP** | Verifikasi Produk | 6 (excl. OBAT, ROKOK) | Needs additional verification. NOT a rejection. See §3b. |
| **TMK MAYOR** | Tidak Memenuhi — Mayor | PANGAN only | Major non-compliance (wrong packaging, unauthorized claims, incorrect ING). |
| **TMK MINOR** | Tidak Memenuhi — Minor | PANGAN only | Minor non-compliance (missing batch code, wrong halal logo). |

### §3b — VP Semantics (varies by komoditi)

| Komoditi | VP meaning | Evidence |
|---|---|---|
| KOSMETIKA | Claims need supporting data (MKL.02). "dermatologically tested", "clinically tested", "SPF", "cruelty free". | Catatan contains "MKL.02" + claim text. |
| PRODUK PANGAN | Verification needed. **No catatan.** 82,9% had balai=MK. | Empty catatan, balai assessed as compliant. |
| KEMASAN PANGAN | Same as PANGAN — verification without rejection. | Empty catatan. |
| OBAT TRADISIONAL | Rare (1,5%). Similar to PANGAN pattern. | |
| SUPLEMEN KESEHATAN | Rare (1,3%). Similar to PANGAN pattern. | |
| OBAT KUASI | Rare (1,5%). Similar to PANGAN pattern. | |

> **VP is NOT a rejection.** When counting "non-compliant", do NOT include VP
> unless the question explicitly says "tidak memenuhi termasuk verifikasi".

### §3c — kesimpulan_penilaian_balai (assessment by regional office)

| Code | n | % | pusat override pattern |
|---|---|---|---|
| MK | 177.080 | 60,6% | 70,3% stay MK, 26,9% → VP, 2,8% → TMK |
| *(empty)* | 85.113 | 29,1% | 70,6% → TMK (pusat downgrades empty balai) |
| TMK | 14.113 | 4,8% | 79,9% stay TMK, 13,6% → MK (pusat upgrades) |
| TMK MINOR | 7.407 | 2,5% | 68,3% → VP, 29,1% stay TMK MINOR |
| TMK MAYOR | 6.592 | 2,3% | 64,5% → VP, 34,4% stay TMK MAYOR |
| TMK Minor | 1.864 | 0,6% | **CASING DUP** of TMK MINOR. Normalise first. |

> **Casing normalisation:** `CASE WHEN kesimpulan_penilaian_balai = 'TMK Minor' THEN 'TMK MINOR' ELSE kesimpulan_penilaian_balai END`. Apply this BEFORE any grouping or filtering on kesimpulan_balai.

### §3d — Cross-tab: komoditi → kesimpulan_pusat (dominant pattern)

| Komoditi | Dominant | Pattern |
|---|---|---|
| KOSMETIKA | MK (80,2%) | MK dominant. TMK (14,7%), VP (5,1%). |
| PRODUK PANGAN | VP (71,5%) | VP dominant. MK (22,3%). TMK MAYOR (3,1%) + TMK MINOR (3,0%). |
| OBAT | TMK (90,5%) | TMK dominant. MK (9,5%). VP = 0. |
| OT | MK (88,2%) | MK dominant. TMK (10,3%). |
| ROKOK | MK (62,3%) | Mixed. TMK (37,7%). VP = 0. |
| SUPLEMEN | MK (93,0%) | MK dominant. |
| OBAT KUASI | MK (92,7%) | MK dominant. |
| KEMASAN PANGAN | VP (99,7%) | VP near-total. MK (0,3%). |

## §4 Default Scope

| Parameter | Default |
|---|---|
| Year | ALL-TIME (no year filter unless asked). Fact starts 2023-01. |
| Komoditi | NO DEFAULT — must clarify if question is komoditi-specific. |
| Balai | NO DEFAULT — must clarify if question is balai-specific. |
| Kesimpulan | NO DEFAULT — must clarify if question filters by kesimpulan. |
| Result limit | Top 10 unless the question specifies otherwise. |
| Time grain | Monthly unless the question says daily/weekly/yearly. |

## §5 Orphan Handling

The log and timeline tables contain 208.407 `id_penandaan` values that do not exist in `mv_penandaan` (2020-2022 data). Decision rules:

| Scenario | SQL pattern |
|---|---|
| **Fact-scoped analysis** (default) | `FROM mv_penandaan p JOIN mv_penandaan_log l ON l.id_penandaan = p.id` — excludes orphan |
| **Full history analysis** | `FROM mv_penandaan_log l` (no join to fact) — includes orphan |
| **Orphan investigation** | `FROM mv_penandaan_log l WHERE NOT EXISTS (SELECT 1 FROM mv_penandaan p WHERE p.id = l.id_penandaan)` |

> Always state which scope was used in the answer. "Termasuk data 2020-2022"
> or "Terbatas pada data 2023+".

## §6 Normalisation Rules

| Column | Issue | Normalisation |
|---|---|---|
| `kesimpulan_penilaian_balai` | "TMK Minor" vs "TMK MINOR" | `CASE WHEN ... = 'TMK Minor' THEN 'TMK MINOR' ELSE ... END` |
| `produsen` | Double-spaces | `TRIM(REGEXP_REPLACE(produsen, '\s{2,}', ' ', 'g'))` |
| `komoditi` (in target_balai) | Title case vs UPPER | `LOWER(komoditi)` for joins, or normalise target to UPPER |
| `trx_steps` (in log) | Miss-aligned with status_label | **Do not use.** Use `status_code` instead. |

## §7 Exclusions

| Exclusion | Rule |
|---|---|
| Unnamed balai | `nama_balai <> ''` — 1 row with empty balai name. |
| ed_nie outliers | `ed_nie BETWEEN '2000-01-01' AND '2100-01-01'` — excludes 31 corrupt dates. |
| Empty nomorsampel | 31.985 rows have empty `nomorsampel` — handle with `WHERE nomorsampel <> ''` when counting unique samples. |

## §8 Cast Rules

All columns in `mv_penandaan` are native PostgreSQL types (bigint, date, text, timestamp). No cast needed — this is NOT like the ERBA system where all columns are TEXT.

| Table | Cast needed? |
|---|---|
| `mv_penandaan` | No — native types |
| `mv_penandaan_log` | No — native types |
| `mv_penandaan_timeline` | No — native types |
| `mv_penandaan_agg` | No — native types |
| `coverage_balai` | No — native types |
| `target_balai` | No — native types |

## §9 execute_sql Rules

- One statement per call. No `;` at the end.
- Never use `EXTRACT(YEAR FROM ...)` to filter — use date range comparison.
- Never use `generate_series`, recursive CTE, or JOINs in forecast SQL.
- Use `GROUP BY` ordinal position for readability.
- Always alias computed columns: `count(*) AS jumlah`.

## §10 Answer Contract

Every data answer must include:

1. **Entity statement** — which counting entity was used (penandaan/surat/produk).
2. **Scope statement** — which filters applied (komoditi, balai, time range, kesimpulan).
3. **Code + description** — every number labelled with its code meaning (e.g., "MK (Memenuhi Ketentuan)").
4. **Proportionality** — percentage alongside absolute count.
5. **Period breakdown** — when multi-period, show per-period table from ONE `GROUP BY`.
6. **Orphan scope** — state whether 2020-2022 data is included or excluded.
7. **Casing note** — if kesimpulan_balai was normalised, mention it.
8. **Balai coverage** — when counting by balai, note if target_balai data is available (54/83).
9. **Data freshness** — "Data per {sync timestamp}".
