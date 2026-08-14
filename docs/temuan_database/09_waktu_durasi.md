# 09 — Waktu & Durasi

> Bottleneck, durasi ≠ selisih tanggal, direktur_pusat, durasi per balai.

---

## 3 Kolom Durasi (semua dalam HARI)

| Tahap | n | Median | P95 | Max | Diagnosa |
|---|---|---|---|---|---|
| `mulai_kabalai` (start → kirim kabalai) | 478.592 | 6 | 123 | 1.475 | Ekor sangat panjang |
| **`kabalai_direktur`** (kabalai → direktur) | 352.830 | **21** | 102 | 996 | **BOTTLENECK UTAMA** |
| `direktur_pusat` (direktur → pusat) | 352.804 | 0 | 0 | 209 | Instan (formalitas) |

> **Proses inti macet di kabalai→direktur** (median 21 hari). Tahap direktur→pusat median 0 = bukan tahap kerja, tapi tanda-tangan otomatis.

---

## ⚠️ KOREKSI: `direktur_pusat` = HARI, bukan flag

**Asumsi awal context:** "`direktur_pusat` hanya 7 distinct = flag/kode, bukan durasi."

**Temuan sebenarnya:** Memang **dalam hari**, tapi 98,4% = 0 (instan).

| nilai | n | % |
|---|---|---|
| 0 | 347.052 | **98,4%** |
| 1 | 5.764 | 1,6% |
| 5 | 2 | |
| 19 | 4 | |
| 142 | 1 | |
| 177 | 1 | |
| 209 | 1 | |

**Interpretasi:** Direktur→pusat hampir selalu instan (median 0), TAPI ada ekor panjang hingga 209 hari. Jangan buang kolom ini sebagai "flag" — valid untuk SLA, hanya saja mayoritas nol.

---

## ⚠️ KOREKSI: Kolom Durasi ≠ Selisih Tanggal

**Asumsi awal:** `mulai_kabalai = tanggal_kirim_kabalai - tgl_start`

**Temuan:** Hanya **4,5% baris cocok** dengan rumus selisih kalender. Untuk `kabalai_direktur` hanya **0,5% cocok**.

**Query verifikasi:**
```sql
SELECT count(*) AS n,
  count(*) FILTER (WHERE mulai_kabalai = (tanggal_kirim_kabalai - tgl_start)) cocok_mulai_kb,
  count(*) FILTER (WHERE kabalai_direktur = (tanggal_kirim_direktur - tanggal_kirim_kabalai)) cocok_kb_dir
FROM mv_penandaan_timeline
WHERE tanggal_kirim_kabalai IS NOT NULL AND tgl_start IS NOT NULL;
-- Hasil: cocok_mulai_kb = 21.306 (4,5%) · cocok_kb_dir = 2.400 (0,5%)
```

**Interpretasi:** Kolom durasi dihitung dengan **logika berbeda** — kemungkinan:
1. Hari kerja (business days, exclude weekend)
2. Anchor event berbeda (bukan tanggal_kirim_*)
3. Pre-computed dengan aturan bisnis khusus

> ⚠️ **Implikasi:** Angka "median 21 hari" TETAP VALID (dari kolom pre-computed). TAPI **jangan verifikasi durasi dengan selisih tanggal** — akan menghasilkan ketidakcocokan yang menyesatkan. Hipotesis hari-kerja perlu verifikasi lanjut ([17_rencana_investigasi](17_rencana_investigasi_lanjut.md) §K5).

---

## Bottleneck per Komoditi (yang SELESAI, status 999)

| Komoditi | median kabalai_direktur | avg | P90 | n_selesai |
|---|---|---|---|---|
| **KOSMETIKA** | **37** | 41,0 | 72 | 74.949 |
| ROKOK | 17 | 27,5 | 61 | 31.012 |
| OBAT | 16 | 17,8 | 33 | 52.427 |
| OBAT TRADISIONAL (OT) | 9 | 11,4 | 22 | 37.816 |
| SUPLEMEN KESEHATAN | 9 | 11,4 | 22 | 10.479 |
| OBAT KUASI | 8 | 11,1 | 21 | 2.606 |

**KOSMETIKA paling lambat** (median 37 hari, P90 72) — volume terbesar (78k) + direktorat kosmetik kelebihan beban (372.709 event).

> **Hipotesis:** Bottleneck = fungsi dari **beban**, bukan kompleksitas produk. KOSMETIKA lambat bukan karena produknya rumit, tapi karena direktorat kosmetik kewalahan volume.

---

## Durasi per Balai (yang lambat — Indonesia Timur/Kecil)

| Balai | avg kabalai_direktur | n_selesai |
|---|---|---|
| LOKA POM DI KABUPATEN KOTAWARINGIN BARAT | **36,5** | 1.391 |
| LOKA POM DI KABUPATEN KEPULAUAN SANGIHE | 32,0 | 1.179 |
| BALAI BESAR POM DI PALANGKARAYA | 31,5 | 4.981 |
| BALAI POM DI KEDIRI | 31,4 | 1.019 |
| BALAI BESAR POM DI KUPANG | 31,2 | 5.195 |
| LOKA POM DI KOTA TANJUNG PINANG | 30,9 | 1.412 |
| BALAI POM DI SOFIFI | 30,0 | 1.645 |
| BALAI BESAR POM DI GORONTALO | 29,8 | 3.647 |

> **Balai kecil/terpencil paling lambat** (Kotawaringin, Sangihe, Kupang, Sofifi — Indonesia Timur/luar Jawa). **Ketimpangan kapasitas geografis.** Lihat [10_geografi_kapasitas](10_geografi_kapasitas.md).

---

## Same-Day Completion (93,5%)

| Metrik | Angka |
|---|---|
| Total baris dengan tgl_end | 292.598 |
| Same-day (tgl_end = tgl_start) | 273.545 (**93,5%**) |
| Avg durasi (tgl_end − tgl_start) | 1,2 hari |

> **93,5% penandaan selesai hari yang sama** — TAPI ini menyesatkan. `tgl_end` = tanggal assessment selesai, BUKAN tanggal workflow status 999. PANGAN punya tgl_end TAPI status stuck di 4. Lihat [07_temuan_kritis](07_temuan_kritis.md).

---

## Anomali Temporal — Ritme & Penurunan

### Pola tahunan (puncak tengah tahun, lembah Desember)

| Bulan | Total 2023-2026 |
|---|---|
| Mei-Jul | ~8.000-10.000/bulan (puncak) |
| Desember | 954-1.297 (lembah, ~85% drop) |

> **Siklus anggaran** — pengawasan mengikuti pencairan dana yang menumpuk tengah tahun.

### Anomali Q1 2025 (sistemik)

| Bulan | Sampel | Balai aktif |
|---|---|---|
| 2024-11 | 4.332 | 76 |
| 2024-12 | 1.288 | 64 |
| 2025-01 | 3.858 | 72 |
| 2025-02 | 2.749 | 71 |
| **2025-03** | **1.438** | **76** |
| 2025-04 | 6.108 | 75 |
| 2025-05 | 9.756 | 76 |

> Maret 2025 runtuh ke 1.438 sampel padahal **76 balai tetap aktif** (~19/balai). **Bukan kekosongan balai** — semua hidup tapi volume anjlok. Recovery Apr-Mei → gangguan sistemik sementara (kemungkinan **migrasi sistem / perubahan kebijakan nasional**). Perlu konfirmasi BPOM.

---

## Koneksi ke Pertanyaan User — "Tanggal Proses Direktur" & SLA

User sering memakai **"tanggal proses direktur"** sebagai dimensi grouping (11 pertanyaan). Ini = **`tanggal_kirim_direktur`**
di `mv_penandaan_timeline` (kolom TANGGAL, bukan durasi). Jangan tertukar dengan `kabalai_direktur`/`direktur_pusat`
(kolom durasi dalam hari).

```sql
-- Rekap per bulan proses direktur (jawab "berdasarkan tanggal proses direktur")
SELECT date_trunc('month', t.tanggal_kirim_direktur)::date AS bulan, COUNT(*) AS jumlah
FROM mv_penandaan p
JOIN mv_penandaan_timeline t ON t.id_penandaan = p.id
WHERE t.tanggal_kirim_direktur IS NOT NULL
  AND p.kesimpulan_penilaian_pusat = 'MK'          -- filter kesimpulan akhir
GROUP BY 1 ORDER BY 1 DESC;
```

**SLA user:** "hasil pengawasan label dikirimkan ke pusat maksimal tanggal 15 bulan berikutnya" (1 pertanyaan).
Ukur aktual: `tanggal_kirim_pusat - tgl_start` (rata-rata nasional **14,9 hari**, n=292.758). Lihat
[21_sql_pairs_penandaan](21_sql_pairs_penandaan.md) SQL-07/SQL-08.

> ⚠️ Kolom durasi (`kabalai_direktur` median 21 hari) ≠ selisih tanggal → jangan validasi silang dengan rumus kalender.
> TAPI untuk SLA user "tanggal pemeriksaan → laporan ke pusat", selisih `tanggal_kirim_pusat - tgl_start` VALID
> karena memakai kolom tanggal aktual, bukan kolom durasi pre-computed.

---

## Implikasi SLA

1. **Bottleneck utama: kabalai→direktur** (median 21 hari, P90 44-72) — fokus perbaikan di sini
2. **KOSMETIKA paling tertekan** — direktorat butuh SDM tambahan
3. **Balai Indonesia Timur tertinggal** — ketimpangan kapasitas geografis
4. **Tanggal kolom durasi tidak boleh di-validasi via selisih kalender** — pakai nilai kolom langsung
5. **Q1 2025 gangguan sistemik** — butuh postmortem operasional

---

Lanjut ke [10_geografi_kapasitas.md](10_geografi_kapasitas.md).

---

## ⚠️ Status penyaluran ke `context/` — kolom tahap timeline: BELUM

Diverifikasi 14 Agustus 2026 terhadap warehouse dan terhadap `context/50-waktu-dan-durasi.md`.

Halaman itu menyebut nama tabel tanpa menyebut satu pun kolom tahapnya — `tanggal_kirim_kabalai`, `tanggal_kirim_pusat`, `tanggal_kirim_direktur`, `mulai_kabalai`, `kabalai_direktur`, `direktur_pusat`. Keenamnya dipakai SQL sistem lama, dan keenamnya hilang dari context. Ini kelompok regresi terbesar di domain ini. Kekosongannya deterministik, sehingga rata-rata durasi yang menyertakan baris kosong akan salah — dan karena porsi kosongnya besar, salahnya besar.

Pengukuran cakupan lengkapnya di dokumen `cakupan_context_vs_database` di direktori ini.
