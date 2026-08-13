# 13 — Konsistensi & Integritas Antar-Tabel

> Drift timeline-log, agg validation, metodologi join target.

---

## 13.1 Konsistensi timeline ↔ log (drift tipis)

`mv_penandaan_timeline` dan `mv_penandaan_log` = dua sudut pandang atas event yang sama.

| Sumber | n status 999 |
|---|---|
| timeline `status=999` | 350.036 |
| log `status_code=999` (event) | 350.495 |
| **Drift** | **459 baris (0,13%)** |

### Interpretasi drift
- **459 baris**: timeline bilang "belum selesai" TAPI log sudah punya event selesai (atau sebaliknya)
- ETL hampir sinkron tapi tidak sempurna
- Tingkat drift rendah (0,13%) → **dapat dipercaya sebagai proxy** satu sama lain

### Query identifikasi baris drift
```sql
-- Sampel timeline status≠999 tapi log punya event 999
SELECT t.id_penandaan, t.status AS tl_status,
  (SELECT max(status_code) FROM mv_penandaan_log
   WHERE id_penandaan=t.id_penandaan AND status_code=999) AS log_999
FROM mv_penandaan_timeline t
WHERE t.status<>999
  AND EXISTS (SELECT 1 FROM mv_penandaan_log
              WHERE id_penandaan=t.id_penandaan AND status_code=999)
LIMIT 15;
```

---

## 13.2 Validasi agg vs fact (SEMPURNA di grand total)

| Sumber | Σ jumlah_penandaan |
|---|---|
| `mv_penandaan` (fact row count) | 292.598 |
| `mv_penandaan_agg` periode `day` | 292.598 ✅ |
| `mv_penandaan_agg` periode `month` | 292.598 ✅ |

**agg = satu-satunya tabel yang integritasnya terbukti sempurna vs fact.**

> ⚠️ TAPI validasi grand total bisa **menyembunyikan kesalahan yang saling meniadakan** di level detail (per balai × bulan). Konsistensi detail perlu verifikasi lanjut — lihat [17_rencana_investigasi](17_rencana_investigasi_lanjut.md) §K9.

### agg scope warning
agg hanya me-rollup era 2023+ (mewarisi bias scope fact). **Tidak mencakup 2020-2022** (orphan).

---

## 13.3 ⚠️ METODOLOGI JOIN target_balai (KOREKSI "KPI buta")

### Asumsi awal context v1.0
> "Tabel target secara fungsional terisolasi dari fact. Dashboard yang join naif akan melaporkan 0% pencapaian palsu."

### Temuan: JOIN BISA berhasil dengan metodologi benar

**❌ SALAH (cartesian explosion):**
```sql
-- Join langsung fact×target → target dihitung ganda
SELECT sum(t.target_penandaan), count(*) AS realisasi
FROM mv_penandaan p JOIN target_balai t
  ON t.nama_balai=p.nama_balai AND lower(t.komoditi)=lower(p.komoditi)
WHERE t.tahun=2024;
-- Hasil: target jadi jutaan (13.963.955) — salah total
```

**✅ BENAR (agregasi terpisah lalu join):**
```sql
WITH tgt AS (
  SELECT nama_balai, komoditi, sum(target_penandaan) AS target
  FROM target_balai WHERE tahun=2024 GROUP BY 1,2
),
rsl AS (
  SELECT nama_balai, komoditi, count(*) AS realisasi
  FROM mv_penandaan WHERE tgl_start BETWEEN '2024-01-01' AND '2024-12-31'
  GROUP BY 1,2
)
SELECT rsl.komoditi, sum(tgt.target) AS target, sum(rsl.realisasi) AS realisasi
FROM rsl LEFT JOIN tgt
  ON tgt.nama_balai=rsl.nama_balai AND lower(tgt.komoditi)=lower(rsl.komoditi)
GROUP BY 1;
```

**Kunci:** agregasi terpisah di tiap tabel LALU join hasil agregat. Hindari cartesian.

### Hasil benar — achievement 2024 (lihat [14_kpi_target_2024](14_kpi_target_2024.md))
| Komoditi | Target | Realisasi | % |
|---|---|---|---|
| ROKOK | 7.020 | 8.171 | 116,4% |
| OBAT | 15.455 | 16.999 | 110,0% |
| KOSMETIKA | 22.833 | 25.024 | 109,6% |
| PRODUK PANGAN | 23.488 | 23.762 | 101,2% |

> **Koreksi:** target_balai **TIDAK buta**. Bisa di-join dengan `lower()` + agregasi terpisah. Semua komoditi terlampaui 101-116%. Context awal salah metodologi → salah simpulkan "KPI rusak".

---

## 13.4 Konsistensi fact ↔ timeline (id matching)

| Metrik | Angka |
|---|---|
| fact distinct id | 292.598 |
| timeline distinct id_penandaan | 500.717 |
| timeline id YANG ADA di fact | 292.598 (semua fact id punya timeline) ✅ |
| timeline id orphan (tak di fact) | 208.119 |

> **Setiap baris fact punya pasangan timeline** (100% coverage fact→timeline). Tapi timeline punya 208.119 extra (orphan era 2020-2022). Lihat [11_data_quality_anomali](11_data_quality_anomali.md).

---

## 13.5 Konsistensi nama_balai fact ↔ coverage

| Metrik | Angka |
|---|---|
| nama_balai distinct di fact | 83 |
| Match ke coverage_balai | 83/83 (**100%**) ✅ |

> Setiap balai di fact terdaftar di coverage. 1 baris fact punya nama_balai kosong (anomali).

---

## 13.6 Peta Kepercayaan Antar-Tabel

| Relasi | Konsistensi | Catatan |
|---|---|---|
| fact ↔ agg | ✅ Sempurna (grand total) | Validasi detail belum |
| fact ↔ timeline | ✅ 100% fact covered | 208k orphan timeline |
| fact ↔ coverage | ✅ 100% balai match | |
| timeline ↔ log | 🟡 99,87% (drift 459) | Bisa saling proxy |
| fact ↔ target | 🟡 Butuh lower() + agg terpisah | Bisa, bukan "buta" |
| fact ↔ dimension | ❌ Dimension stale (74k tertinggal) | Jangan pakai dimension |

---

## Implikasi

1. **agg paling dapat dipercaya** untuk dashboard cepat (sudah pre-aggregated + valid)
2. **timeline ↔ log drift 0,13%** — dapat pakai salah sebagai proxy
3. **Metodologi join target penting** — agregasi terpisah hindari cartesian
4. **Dimension JANGAN dipakai** — stale, tanpa id
5. **Konsistensi detail agg** perlu verifikasi (bisa menyembunyikan offset)

---

Lanjut ke [14_kpi_target_2024.md](14_kpi_target_2024.md).
