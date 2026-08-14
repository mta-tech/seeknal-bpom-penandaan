# 14 — KPI Target 2024 (Koreksi "KPI Buta")

> Achievement sebenarnya 101-116%. Koreksi hipotesis context awal.

---

## ⚠️ KOREKSI UTAMA

**Asumsi context v1.0:**
> "Tabel target secara fungsional terisolasi dari fact. Dashboard yang join naif akan melaporkan 0% pencapaian palsu. Tabel target ini secara praktis tidak terhubung ke fact tanpa transformasi."

**Temuan:** **SALAH.** Target 2024 **dapat di-join** dan **semua komoditi terlampaui 101-116%**. Hipotesis "KPI buta" berasal dari **metodologi join yang salah** (cartesian explosion), bukan dari data yang rusak.

---

## Achievement 2024 per Komoditi (TERVERIFIKASI)

| Komoditi | Target | Realisasi 2024 | % | Status |
|---|---|---|---|---|
| ROKOK | 7.020 | 8.171 | **116,4%** | 🟢 overachieve tertinggi |
| OBAT | 15.455 | 16.999 | **110,0%** | 🟢 |
| SUPLEMEN KESEHATAN | 3.108 | 3.430 | 110,4% | 🟢 |
| KOSMETIKA | 22.833 | 25.024 | **109,6%** | 🟢 |
| OBAT TRADISIONAL (OT) | 11.505 | 12.552 | 109,1% | 🟢 |
| OBAT KUASI | 813 | 871 | 107,1% | 🟢 |
| PRODUK PANGAN | 23.488 | 23.762 | **101,2%** | 🟢 tipis |
| KEMASAN PANGAN | — | 358 | ❓ | ⚠️ **tak punya target** |

> **7 dari 7 komoditi yang punya target TERLAMPAUI** (101-116%). KEMASAN PANGAN tidak punya target di `target_balai` — 358 sampel realisasi tak terukur.

---

## Achievement per Balai (top achievers)

| Balai | Target | Realisasi | % |
|---|---|---|---|
| BALAI BESAR POM DI KUPANG | 2.053 | 2.145 | 104,5% |
| BALAI POM DI TANGERANG | 583 | 600 | 102,9% |
| BALAI BESAR POM DI SERANG | 1.377 | 1.412 | 102,5% |
| BALAI POM DI BOGOR | 704 | 721 | 102,4% |
| BALAI BESAR POM DI JAKARTA | 3.463 | 3.507 | 101,3% |
| BALAI BESAR POM DI MATARAM | 2.340 | 2.338 | 99,9% |
| BALAI BESAR POM DI BANDA ACEH | 1.996 | 1.994 | 99,9% |
| BALAI BESAR POM DI DENPASAR | 3.088 | 3.076 | 99,6% |

> Top balai overachieve 100-105%. Mayoritas balai mencapai target.

---

## Metodologi Join yang Benar (KRITIS)

Lihat [13_konsistensi_integritas](13_konsistensi_integritas.md) §13.3 untuk detail lengkap. Ringkasan:

```sql
-- ✅ BENAR: agregasi terpisah lalu join
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

**Tiga kunci:**
1. `lower(komoditi)` di kedua sisi (casing mismatch: target "Kosmetika" vs fact "KOSMETIKA")
2. **Agregasi terpisah** sebelum join (hindari cartesian)
3. `tgl_start BETWEEN '2024-01-01' AND '2024-12-31'` (bukan EXTRACT YEAR)

---

## Limitasi Tabel Target (yang TETAP ada)

Meskipun join berhasil, tabel target punya 3 limitasi struktural:

### Limitasi 1 — Hanya tahun 2024
| Tahun | Tersedia? |
|---|---|
| 2023 | ❌ |
| 2024 | ✅ |
| 2025 | ❌ |
| 2026 | ❌ |

> **3 dari 4 tahun fact tak terukur KPI-nya.** Target 2025/2026 butuh backfill dari Renstra/DIPA BPOM.

### Limitasi 2 — 29 balai tanpa target
| Item | Angka |
|---|---|
| Balai aktif fact | 83 |
| Balai di target | 76 |
| Match | 54 |
| **Balai tanpa target** | **29** |

> 29 balai aktif **tak terukur**. Plus 22 target balai names tak match exact ke fact (perlu name mapping).

### Limitasi 3 — KEMASAN PANGAN absent
`target_balai` hanya 7 komoditi (tanpa KEMASAN PANGAN). 689 sampel kemasan pangan tak punya target → tak terukur sama sekali.

---

## Paradox PANGAN: Target Terlampaui TAPI 0% Completion

| Metrik PANGAN | Angka |
|---|---|
| Target 2024 | 23.488 |
| Realisasi 2024 (sampel masuk) | 23.762 (101,2%) ✅ |
| Completion (sampai status 999) | **0 (0%)** ❌ |

> **PANGAN overachieve target 101% TAPI 0% selesai workflow.** Ini **pencapaian target semu** — sampel masuk & dinilai, tapi workflow mandek di MT (lihat [07_temuan_kritis](07_temuan_kritis.md) §temuan-1). KPI target mengukur "input", bukan "output selesai".

**Implikasi audit:** KPI penandaan PANGAN **menyesatkan** bila hanya lihat realisasi vs target. Perlu KPI tambahan: **completion rate**.

---

## Summary KPI

| Aspek | Status |
|---|---|
| Join target ↔ fact | ✅ Berhasil (lower + agg terpisah) |
| Achievement 2024 | ✅ 101-116% (semua overachieve) |
| Target 2025/2026 | ❌ Tidak ada — butuh backfill |
| 29 balai tanpa target | ⚠️ Butuh backfill |
| KEMASAN PANGAN target | ❌ Tidak ada |
| KPI completion | ❌ Tidak diukur — PANGAN 0% tak terlihat |

> **Koreksi context:** ganti narasi "KPI buta" → "KPI 2024 terlampaui, TAPI butuh backfill tahun lain + metrik completion".

---

Lanjut ke [15_gap_belum_ditemukan.md](15_gap_belum_ditemukan.md).

---

## ⚠️ Status penyaluran ke `context/` — kolom target: BELUM

Diverifikasi 14 Agustus 2026 terhadap warehouse dan terhadap `context/85-target-capaian.md`.

Halaman itu menyebut nama tabel `target_balai` dan kunci join-nya, lalu berhenti — padahal dokumen ini sudah memuat grain balai × komoditi, batas tahun, dan ketujuh kolom target dengan benar. Untuk domain ini kolom yang relevan adalah `target_penandaan`; enam kolom lain milik kegiatan lain dan tidak boleh dipakai.

Pengukuran cakupan lengkapnya di dokumen `cakupan_context_vs_database` di direktori ini.
