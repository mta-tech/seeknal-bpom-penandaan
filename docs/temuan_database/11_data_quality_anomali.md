# 11 — Data Quality & Anomali

> Orphan 208k, tanggal 1970, kode ambigu, dimension stale, ed_nie outlier.

---

## Anomali #1 — Orphan 208.119 (riwayat tanpa induk fact)

`mv_penandaan_log` & `mv_penandaan_timeline` punya **208.119 `id_penandaan` yang tak ada di fact**. Fact mulai 2023, log/timeline simpan 2020+.

### Profil orphan (terverifikasi)

| Status orphan | n | % | Interpretasi |
|---|---|---|---|
| **999 (selesai)** | **140.747** | **67,6%** | Arsip historis aman |
| 4 (stuck) 2021 | 18.666 | 9,0% | ⚠️ **VP-limbo LEBIH TUA dari fact!** |
| 0 (draft) 1970 | 6.312 | 3,0% | Tanggal korup (Unix epoch) |
| Lainnya | ~42.394 | 20,4% | Termasuk reject, intermediate |

### Komposisi orphan per tahun (sampel)

| Tahun | Status dominan | Catatan |
|---|---|---|
| 1907 | (1 baris) | Tanggal korup |
| 1970 | status 0 (6.312) | **Unix epoch** — null date di-default 1970-01-01 |
| 2019-2020 | status 999 (11.213) | Arsip |
| 2021 | status 4 (18.666) ⚠ | **Backlog tersembunyi** |
| 2022 | status 999 mayoritas | Arsip |

> **Orphan bukan sampah.** Ada:
> - **Arsip aman** (140.747 selesai)
> - **Backlog tersembunyi 2021** (18.666 stuck — VP-limbo pra-fact)
> - **6.312 baris tanggal korup** (Unix epoch 1970)
>
> Populasi proses sesungguhnya = **500.717**, bukan 292.598 (fact).

### Dapat direkonstruksi?
208.119 orphan punya `nama_balai` di log → **identitas parsial bisa dipulihkan**. Keputusan "buang atau backfill" ada di tangan tim data, bukan dipaksa keterbatasan.

---

## Anomali #2 — 6.312 baris Tanggal 1970 (Unix Epoch)

| Status | n |
|---|---|
| 0 (draft) | 6.312 |
| 1 | 41 |

`tgl_start = '1970-01-01'` = **null date di-default ke Unix epoch** (timestamp 0). Ini **data quality issue** — 6.312 baris dengan tanggal tidak valid tersembunyi di timeline.

> Saat GROUP BY tahun, baris 1970 muncul sebagai "tahun 1970" yang menyesatkan. **Filter `WHERE tgl_start >= '2020-01-01'`** sebelum analisis temporal.

---

## Anomali #3 — Casing Duplikat kesimpulan_balai

| Nilai | n |
|---|---|
| TMK MINOR | 7.407 |
| **TMK Minor** | **1.864** ← casing dup |

**Normalisasi WAJIB** sebelum filter/group:
```sql
CASE WHEN kesimpulan_penilaian_balai = 'TMK Minor' THEN 'TMK MINOR'
     ELSE kesimpulan_penilaian_balai END
```

---

## Anomali #4 — 31 ed_nie Outlier

| Tahun ed_nie | n |
|---|---|
| 1026, 1027, 1028 | 1 masing-masing |
| 1747 | 1 |
| 1924-1929 | ~19 |
| 2102, 2124-2127, 2225, 2929 | ~9 |

**Filter outlier:** `WHERE ed_nie BETWEEN '2000-01-01' AND '2100-01-01'` untuk analisis expiry valid.

### Distribusi ed_nie normal
- Expired (≤2025): ~126k
- Valid (2026-2031): ~137k
- Bulk expire 2024-2028

> ROKOK 69,8% ed_nie null — **deterministik** (rokok tidak pakai skema NIE sama). Null-nya bukan acak, lihat [04_komoditi](04_komoditi_governing_dimension.md).

---

## Anomali #5 — Dimension Schema STALE

| Tabel | public | dimension | Selisih |
|---|---|---|---|
| mv_penandaan | 292.598 | 218.472 | **74.126 tertinggal** |
| mv_penandaan_log | 3.533.299 | 33.137 | (subset kecil) |
| coverage_balai | 668 | 513 | 81 balai (vs 88) |
| target_balai | 532 | 76 | (pivot) |

**`dimension` = proyeksi BI yang di-flatten:**
- Tanpa id, tanpa tanggal, tanpa kode
- Tak bisa di-join balik ke public
- Snapshot beku — tertinggal 74k baris vs public

> ⚠️ **Public = sumber kebenaran tunggal.** Dimension perlu di-label "arsip" atau dibangun ulang sebagai view hidup. Risiko: dua angka berbeda untuk pertanyaan sama ("berapa penandaan?" → 292k vs 218k).

---

## Anomali #6 — 12 Prefix Kode Balai Ambigu (nomorsampel)

`nomorsampel` prefix 3-digit (`yy.bbb.`) mengandung kode balai. Dari 73 prefix:

| Kategori | n prefix | % | Arti |
|---|---|---|---|
| **Konsisten** (1 prefix → 1 balai) | 61 | **84%** | Validasi silang andal |
| **Ambigu** (1 prefix → >1 balai) | 12 | 16% | Identitas balai ragu |

### Prefix ambigu terbesar
| Kode | n_balai | n_sampel |
|---|---|---|
| 094 | 3 | 9.297 |
| 119 | 3 | 5.844 |
| 093, 098, 100, 102, 104, 108 | 2 masing-masing | 5.000-10.000 |

> 9.297 sampel kode 094 punya identitas balai ambigu — kemungkinan **kode dipakai ulang saat reorganisasi balai** ATAU kesalahan input. Lihat [12_kode_berstruktur](12_kode_berstruktur.md).

---

## Anomali #7 — Produsen Double-Space & Corrupt String

Beberapa `produsen` punya spasi ganda atau string corrupt (artefak ETL):
- `BERNOFARM   INDONESIA` (3 spasi) → hasil penggabungan field nama+negara
- String diduplikasi tanpa delimiter (mirip pengawasan)

**Normalisasi:**
```sql
TRIM(REGEXP_REPLACE(produsen, '\s{2,}', ' ', 'g'))
```

---

## Anomali #8 — Drift Antar-Tabel (timeline vs log)

| Sumber | n status 999 |
|---|---|
| `mv_penandaan_timeline` status=999 | 350.036 |
| `mv_penandaan_log` event status_code=999 | 350.495 |
| **Selisih** | **459 (0,13%)** |

> **459 baris drift** — timeline bilang "belum selesai" TAPI log sudah punya event selesai. ETL hampir sinkron tapi tidak sempurna. Lihat [13_konsistensi_integritas](13_konsistensi_integritas.md).

---

## Peta Anomali (skeptis ringkas)

| # | Anomali | Materialitas | Status |
|---|---|---|---|
| 1 | 208.119 orphan | Tinggi — definisi populasi | ✅ terverifikasi |
| 2 | 6.312 baris tanggal 1970 | Sedang — data quality | ✅ terverifikasi |
| 3 | Casing dup TMK Minor | Rendah — normalisasi mudah | ✅ terverifikasi |
| 4 | 31 ed_nie outlier | Rendah — filter sederhana | ✅ terverifikasi |
| 5 | Dimension stale 74k | Sedang — SSoT ganda | ✅ terverifikasi |
| 6 | 12 prefix kode ambigu | Sedang — 99k sampel identitas ragu | ✅ terverifikasi |
| 7 | Produsen double-space | Rendah — normalisasi | ✅ terverifikasi |
| 8 | Drift 459 baris | Rendah — 0,13% | ✅ terverifikasi |

> **Tidak ada anomali yang menggagalkan analisis agregat.** Tapi semua perlu diakui di jawaban (answer contract).

---

Lanjut ke [12_kode_berstruktur.md](12_kode_berstruktur.md).
