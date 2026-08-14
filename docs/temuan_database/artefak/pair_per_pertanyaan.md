# Pair per Pertanyaan — database `penandaan`

Setiap pasangan pertanyaan→SQL dari `context_stores` KAI, ditembakkan ke database live **2026-08-13**, lalu didiagnosis sebabnya. Total **43 pertanyaan**, **40 menghasilkan data (93%)**.

## Sebaran diagnosa

| Kode | Arti | Jumlah |
|---|---|--:|
| `PULIH_RELASI` | 🔧 Pulih: ganti nama relasi | 21 |
| `OK_LANGSUNG` | ✅ Jalan apa adanya | 18 |
| `NOL_FILTER_SEMPIT` | 🔴 Nol baris — kombinasi filter menyempit sampai kosong | 2 |
| `ERR_KOLOM_DIHAPUS` | ⛔ Gagal — kolom dihapus dari skema (NOT COVERED) | 1 |
| `OK_TAPI_RAKSASA` | ⚠️ Jalan tapi >100rb baris tanpa agregasi | 1 |

> Berkas pendamping: `pair_ringkas.csv` (tabel, satu baris per pertanyaan) dan `pair_detail_sql.csv` (SQL diratakan satu baris).

---

## Generasi v1 — koneksi awal (Jul 2025), SQL menunjuk view `vw_*`

13 pertanyaan.

### [1] "upt mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt memenuhi ketentuan?"

| | |
|---|---|
| Bentuk NER | `"upt mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan <CONCLUSION TYPE>?"` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 1 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT nama_balai FROM vw_penandaan WHERE kesimpulan_penilaian_pusat = 'MK' GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 1;
-- DIPAKAI: SELECT nama_balai FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'MK' GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 1;
```

### [2] "upt mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt memenuhi ketentuan?"

| | |
|---|---|
| Bentuk NER | `"upt mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan <CONCLUSION TYPE>?"` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 5 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT nama_balai, count(*) FROM vw_penandaan WHERE kesimpulan_penilaian_pusat LIKE '%MK%' GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 5
-- DIPAKAI: SELECT nama_balai, count(*) FROM mv_penandaan WHERE kesimpulan_penilaian_pusat LIKE '%MK%' GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 5
```

### [3] "upt mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt tidak memenuhi ketentuan?"

| | |
|---|---|
| Bentuk NER | `"upt mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt <CONCLUSION TYPE>?"` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 1 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT nama_balai FROM vw_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_balai ORDER BY count(*) DESC LIMIT 1;
-- DIPAKAI: SELECT nama_balai FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_balai ORDER BY count(*) DESC LIMIT 1;
```

### [4] "upt mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt tidak memenuhi ketentuan?"

| | |
|---|---|
| Bentuk NER | `"upt mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt <CONCLUSION TYPE>?"` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 5 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT nama_balai, count(*) FROM vw_penandaan WHERE kesimpulan_penilaian_pusat LIKE '%TMK%' GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 5
-- DIPAKAI: SELECT nama_balai, count(*) FROM mv_penandaan WHERE kesimpulan_penilaian_pusat LIKE '%TMK%' GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 5
```

### [5] tampilkan data obat tradisional; suplemen kesehatan; obat kuasi hasil pengawasan penandaan yang merupakan kategori obat tradisional; suplemen kesehatan; obat kuasi dan klaim dalam promosi/iklan.

| | |
|---|---|
| Bentuk NER | `tampilkan data <CLASSIFICATION>; <CLASSIFICATION>; <CLASSIFICATION> hasil pengawasan penandaan yang merupakan kategori <CLASSIFICATION>; <CLASSIFICATI` |
| Tabel | `mv_penandaan` · agregasi: tidak |
| Status | ERR_SCHEMA → **OK** · 52,685 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT * FROM vw_penandaan WHERE lower(komoditi) IN ('obat tradisional (ot)', 'suplemen kesehatan', 'obat kuasi')
-- DIPAKAI: SELECT * FROM mv_penandaan WHERE lower(komoditi) IN ('obat tradisional (ot)', 'suplemen kesehatan', 'obat kuasi')
```

### [6] tampilkan nama upt yang tidak melakukan input catatan tmk pada hasil pengawasan penandaan dengan kesimpulan tmk.

| | |
|---|---|
| Bentuk NER | `tampilkan nama upt yang tidak melakukan input catatan <CONCLUSION TYPE> pada hasil pengawasan penandaan dengan kesimpulan <CONCLUSION TYPE>.` |
| Tabel | `mv_penandaan` · agregasi: tidak |
| Status | ERR_SCHEMA → **NOL_BARIS** · 0 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔴 Nol baris — kombinasi filter menyempit sampai kosong** |
| Sebab | Kombinasi filter menyempit sampai kosong — periksa tiap klausa terhadap katalog nilai |

```sql
-- ASLI   : SELECT DISTINCT nama_balai FROM vw_penandaan WHERE catatan IS NULL AND kesimpulan_penilaian_pusat = 'TMK';
-- DIPAKAI: SELECT DISTINCT nama_balai FROM mv_penandaan WHERE catatan IS NULL AND kesimpulan_penilaian_pusat = 'TMK';
```

### [7] tampilkan nama upt yang tidak melakukan input catatan tmk pada hasil pengawasan penandaan dengan kesimpulan tmk.

| | |
|---|---|
| Bentuk NER | `tampilkan nama upt yang tidak melakukan input catatan <CONCLUSION TYPE> pada hasil pengawasan penandaan dengan kesimpulan <CONCLUSION TYPE>.` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **NOL_BARIS** · 0 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔴 Nol baris — kombinasi filter menyempit sampai kosong** |
| Sebab | Kombinasi filter menyempit sampai kosong — periksa tiap klausa terhadap katalog nilai |

```sql
-- ASLI   : SELECT DISTINCT nama_balai, count(*) FROM vw_penandaan WHERE catatan IS NULL AND kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_balai;
-- DIPAKAI: SELECT DISTINCT nama_balai, count(*) FROM mv_penandaan WHERE catatan IS NULL AND kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_balai;
```

### [8] tampilkan rekapitulasi jumlah laporan pengawasan penandaan masing-masing upt yang telah dikirim ke pusat pada periode waktu antara tanggal 1 januari 2025 hingga 31 desember 2025.

| | |
|---|---|
| Bentuk NER | `tampilkan rekapitulasi jumlah laporan pengawasan penandaan masing-masing upt yang telah dikirim ke pusat pada periode waktu antara tanggal 1 januari <` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 77 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT nama_balai, COUNT(id) AS jumlah_laporan FROM vw_penandaan WHERE tgl_start BETWEEN '2025-01-01' AND '2025-12-31' GROUP BY nama_balai;
-- DIPAKAI: SELECT nama_balai, COUNT(id) AS jumlah_laporan FROM mv_penandaan WHERE tgl_start BETWEEN '2025-01-01' AND '2025-12-31' GROUP BY nama_balai;
```

### [9] tampilkan rekapitulasi jumlah laporan pengawasan penandaan masing-masing upt yang telah dikirim ke pusat pada periode waktu antara tanggal 1 januari 2025 sampai dengan 31 desember 2025.

| | |
|---|---|
| Bentuk NER | `tampilkan rekapitulasi jumlah laporan pengawasan penandaan masing-masing upt yang telah dikirim ke pusat pada periode waktu antara tanggal 1 januari <` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 77 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT nama_balai, COUNT(id) AS jumlah_laporan FROM vw_penandaan WHERE tgl_start >= '2025-01-01' AND tgl_end <= '2025-12-31' GROUP BY nama_balai;
-- DIPAKAI: SELECT nama_balai, COUNT(id) AS jumlah_laporan FROM mv_penandaan WHERE tgl_start >= '2025-01-01' AND tgl_end <= '2025-12-31' GROUP BY nama_balai;
```

### [10] tampilkan urutan data nama industri farmasi dengan hasil pengawasan akhir dari pusat yang memiliki obat dengan penandaan yang tidak memenuhi ketentuan paling banyak sampai dengan paling rendah.

| | |
|---|---|
| Bentuk NER | `tampilkan urutan data <CLASSIFICATION> dengan <CONCLUSION TYPE> dari pusat yang memiliki <PRODUCT NAME> dengan <REGULATION TYPE> paling banyak sampai ` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 6,788 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT produsen, COUNT(*) AS jumlah_tidak_memenuhi_ketentuan FROM vw_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY produsen ORDER BY jumlah_tidak_memenuhi_ketentuan DESC;
-- DIPAKAI: SELECT produsen, COUNT(*) AS jumlah_tidak_memenuhi_ketentuan FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY produsen ORDER BY jumlah_tidak_memenuhi_ketentuan DESC;
```

### [11] tampilkan urutan data nama sarana produksi dengan hasil pengawasan akhir dari pusat yang memiliki obat tradisional; suplemen kesehatan; obat kuasi dengan penandaan yang tidak memenuhi ketentuan diurut

| | |
|---|---|
| Bentuk NER | `tampilkan urutan data nama <FACILITY TYPE> dengan hasil pengawasan akhir dari pusat yang memiliki <COMMODITY NAME>; <COMMODITY NAME>; <COMMODITY NAME>` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 2,589 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT nama_produk FROM vw_penandaan WHERE lower(komoditi) IN ('obat tradisional (ot)', 'suplemen kesehatan', 'obat kuasi') AND kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_produk ORDER BY COUNT(*) DESC;
-- DIPAKAI: SELECT nama_produk FROM mv_penandaan WHERE lower(komoditi) IN ('obat tradisional (ot)', 'suplemen kesehatan', 'obat kuasi') AND kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_produk ORDER BY COUNT(*) DESC;
```

### [12] urutkan nama kemasan yang memiliki kesimpulan akhir pengawasan penandaan tmk paling banyak sampai dengan paling rendah.

| | |
|---|---|
| Bentuk NER | `urutkan nama kemasan yang memiliki kesimpulan akhir <CONCLUSION TYPE> paling banyak sampai dengan paling rendah.` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 15,529 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT nama_produk FROM vw_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_produk ORDER BY count(*) DESC;
-- DIPAKAI: SELECT nama_produk FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_produk ORDER BY count(*) DESC;
```

### [13] urutkan nama kemasan yang memiliki kesimpulan akhir pengawasan penandaan tmk paling banyak sampai dengan paling rendah.

| | |
|---|---|
| Bentuk NER | `urutkan nama kemasan yang memiliki kesimpulan akhir <CONCLUSION TYPE> paling banyak sampai dengan paling rendah.` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 15,529 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT nama_produk, count(*) FROM vw_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_produk ORDER BY COUNT(*) DESC;
-- DIPAKAI: SELECT nama_produk, count(*) FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_produk ORDER BY COUNT(*) DESC;
```

---

## Generasi v2 — koneksi `_all` (Ags 2025)

10 pertanyaan.

### [14] berikan top 5 upt yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt memenuhi ketentuan?

| | |
|---|---|
| Bentuk NER | `berikan top 5 upt yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt <CONCLUSION TYPE>?` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 5 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT nama_balai, count(*) FROM vw_penandaan WHERE kesimpulan_penilaian_pusat = 'MK' GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 5
-- DIPAKAI: SELECT nama_balai, count(*) FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'MK' GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 5
```

### [15] berikan top 5 upt yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt tidak memenuhi ketentuan?

| | |
|---|---|
| Bentuk NER | `berikan top 5 upt yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt <CONCLUSION TYPE>?` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 5 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT nama_balai, count(*) FROM vw_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 5
-- DIPAKAI: SELECT nama_balai, count(*) FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 5
```

### [16] tampilkan data obat tradisional; suplemen kesehatan; obat kuasi hasil pengawasan penandaan yang merupakan kategori obat tradisional; suplemen kesehatan; obat kuasi dan klaim dalam promosi/iklan.

| | |
|---|---|
| Bentuk NER | `tampilkan data <CLASSIFICATION>; <CLASSIFICATION>; <CLASSIFICATION> hasil pengawasan penandaan yang merupakan kategori <CLASSIFICATION>; <CLASSIFICATI` |
| Tabel | `mv_penandaan` · agregasi: tidak |
| Status | ERR_SCHEMA → **OK** · 52,685 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT * FROM public.vw_penandaan WHERE komoditi IN ('OBAT TRADISIONAL (OT)', 'SUPLEMEN KESEHATAN', 'OBAT KUASI');
-- DIPAKAI: SELECT * FROM mv_penandaan WHERE komoditi IN ('OBAT TRADISIONAL (OT)', 'SUPLEMEN KESEHATAN', 'OBAT KUASI');
```

### [17] tampilkan nama upt yang tidak melakukan input catatan tmk pada hasil pengawasan penandaan dengan kesimpulan tmk.

| | |
|---|---|
| Bentuk NER | `tampilkan nama upt yang tidak melakukan input catatan <CONCLUSION TYPE> pada hasil pengawasan penandaan dengan kesimpulan <CONCLUSION TYPE>.` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 84 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT nama_balai, count(*) as jumlah_penandaan FROM public.vw_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' AND (catatan IS NULL OR catatan = '') group by nama_balai order by jumlah_penandaan desc;
-- DIPAKAI: SELECT nama_balai, count(*) as jumlah_penandaan FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' AND (catatan IS NULL OR catatan = '') group by nama_balai order by jumlah_penandaan desc;
```

### [18] tampilkan nama upt yang tidak melakukan input catatan tmk pada hasil pengawasan penandaan dengan kesimpulan tmk.

| | |
|---|---|
| Bentuk NER | `tampilkan nama upt yang tidak melakukan input catatan <CONCLUSION TYPE> pada hasil pengawasan penandaan dengan kesimpulan <CONCLUSION TYPE>.` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 84 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT nama_balai, count(*) as jumlah_penandaan FROM public.vw_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' AND (catatan IS NULL OR catatan = '') group by nama_balai order by jumlah_penandaan desc;
-- DIPAKAI: SELECT nama_balai, count(*) as jumlah_penandaan FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' AND (catatan IS NULL OR catatan = '') group by nama_balai order by jumlah_penandaan desc;
```

### [19] tampilkan urutan data nama sarana produksi dengan hasil pengawasan akhir dari pusat yang memiliki obat tradisional; suplemen kesehatan; obat kuasi dengan penandaan yang tidak memenuhi ketentuan diurut

| | |
|---|---|
| Bentuk NER | `tampilkan urutan data nama <FACILITY TYPE> dengan hasil pengawasan akhir dari pusat yang memiliki <COMMODITY NAME>; <COMMODITY NAME>; <COMMODITY NAME>` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 1,228 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT produsen, COUNT(*) AS jumlah_temuan FROM public.vw_penandaan WHERE komoditi IN ('OBAT TRADISIONAL (OT)', 'SUPLEMEN KESEHATAN', 'OBAT KUASI') AND kesimpulan_penilaian_pusat = 'TMK' GROUP BY produsen ORDER BY jumlah_temuan DESC;
-- DIPAKAI: SELECT produsen, COUNT(*) AS jumlah_temuan FROM mv_penandaan WHERE komoditi IN ('OBAT TRADISIONAL (OT)', 'SUPLEMEN KESEHATAN', 'OBAT KUASI') AND kesimpulan_penilaian_pusat = 'TMK' GROUP BY produsen ORDER BY jumlah_temuan DESC;
```

### [20] upt mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt memenuhi ketentuan?

| | |
|---|---|
| Bentuk NER | `upt mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt <CONCLUSION TYPE>?` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 10 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT nama_balai FROM vw_penandaan WHERE kesimpulan_penilaian_pusat = 'MK' GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 10;
-- DIPAKAI: SELECT nama_balai FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'MK' GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 10;
```

### [21] upt mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt tidak memenuhi ketentuan?

| | |
|---|---|
| Bentuk NER | `upt mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt <CONCLUSION TYPE>?` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 10 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT nama_balai FROM vw_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_balai ORDER BY count(*) DESC LIMIT 10;
-- DIPAKAI: SELECT nama_balai FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_balai ORDER BY count(*) DESC LIMIT 10;
```

### [22] urutkan nama kemasan yang memiliki kesimpulan akhir pengawasan penandaan tmk paling banyak sampai dengan paling rendah.

| | |
|---|---|
| Bentuk NER | `urutkan nama kemasan yang memiliki kesimpulan akhir <CONCLUSION TYPE> paling banyak sampai dengan paling rendah.` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 15,529 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT nama_produk, count(*) FROM vw_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_produk ORDER BY COUNT(*) DESC;
-- DIPAKAI: SELECT nama_produk, count(*) FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_produk ORDER BY COUNT(*) DESC;
```

### [23] urutkan nama kemasan yang memiliki kesimpulan akhir pengawasan penandaan tmk paling banyak sampai dengan paling rendah.

| | |
|---|---|
| Bentuk NER | `urutkan nama kemasan yang memiliki kesimpulan akhir <CONCLUSION TYPE> paling banyak sampai dengan paling rendah.` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | ERR_SCHEMA → **OK** · 15,529 baris |
| Lapis terjemahan | relasi vw_penandaan→mv_penandaan |
| Diagnosa | **🔧 Pulih: ganti nama relasi** |
| Sebab | Pulih setelah penyesuaian relasi ke skema live |

```sql
-- ASLI   : SELECT nama_produk, count(*) FROM vw_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_produk ORDER BY COUNT(*) DESC;
-- DIPAKAI: SELECT nama_produk, count(*) FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_produk ORDER BY COUNT(*) DESC;
```

---

## Generasi v3 — koneksi `_all_v2` (Nov 2025), skema berlaku

20 pertanyaan.

### [24] Berdasarkan kabupaten/kota, tampilkan jumlah perbedaan kesimpulan balai dengan pusat saat kesimpulan penilaian pusat = TMK untuk tahun 2025

| | |
|---|---|
| Bentuk NER | `Berdasarkan kabupaten/kota, tampilkan jumlah perbedaan kesimpulan balai dengan pusat saat kesimpulan penilaian pusat = <CONCLUSION TYPE> untuk tahun <` |
| Tabel | `mv_penandaan + mp` · agregasi: ya |
| Status | ERR_SCHEMA → **ERR_SCHEMA** |
| Lapis terjemahan | - |
| Diagnosa | **⛔ Gagal — kolom dihapus dari skema (NOT COVERED)** |
| Sebab | Kolom mp.kabupaten sudah dihapus dari skema live (provinsi/kabupaten) — tidak bisa ditulis ulang, NOT COVERED |

```sql
SELECT mp.kabupaten, COUNT(*) AS jumlah_perbedaan FROM mv_penandaan mp WHERE mp.kesimpulan_penilaian_balai IS NOT NULL AND mp.kesimpulan_penilaian_pusat IS NOT NULL AND LOWER(mp.kesimpulan_penilaian_balai) <> LOWER(mp.kesimpulan_penilaian_pusat) and mp.kesimpulan_penilaian_pusat = 'TMK' and extract(year from mp.tgl_end) = 2025 GROUP BY mp.kabupaten ORDER BY jumlah_perbedaan DESC;
```

> ERROR: `ERROR: column mp.kabupaten does not exist`

### [25] Berdasarkan produsen dua kelinci, tampilkan berapa jumlah label yang dilaporkan MK atau TMK beserta jumlah perbedaan antara kesimpulan balai dengan pusat

| | |
|---|---|
| Bentuk NER | `Berdasarkan produsen <COMPANY NAME>, tampilkan berapa jumlah label yang dilaporkan MK atau TMK beserta jumlah perbedaan antara kesimpulan balai dengan` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | OK → **OK** · 3 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
select mp.pendaftar, count(*) as total_laporan, sum(case when lower(mp.kesimpulan_penilaian_pusat) = 'mk' then 1 else 0 end) as jumlah_mk_pusat, sum(case when lower(mp.kesimpulan_penilaian_pusat) = 'tmk' then 1 else 0 end) as jumlah_tmk_pusat, sum(case when lower(mp.kesimpulan_penilaian_balai) = 'mk' then 1 else 0 end) as jumlah_mk_balai, sum(case when lower(mp.kesimpulan_penilaian_balai) = 'tmk' then 1 else 0 end) a
```

### [26] berikan top 5 upt yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt memenuhi ketentuan?

| | |
|---|---|
| Bentuk NER | `berikan top 5 upt yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt <CONCLUSION TYPE>?` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | OK → **OK** · 5 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
SELECT nama_balai, count(*) FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'MK' GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 5
```

### [27] berikan top 5 upt yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt memenuhi ketentuan?

| | |
|---|---|
| Bentuk NER | `berikan top 5 upt yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt <CONCLUSION TYPE>?` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | OK → **OK** · 5 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
SELECT nama_balai, count(*) FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'MK' GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 5
```

### [28] berikan top 5 upt yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt tidak memenuhi ketentuan?

| | |
|---|---|
| Bentuk NER | `berikan top 5 upt yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt <CONCLUSION TYPE>?` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | OK → **OK** · 5 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
SELECT nama_balai, count(*) FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 5
```

### [29] berikan top 5 upt yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt tidak memenuhi ketentuan?

| | |
|---|---|
| Bentuk NER | `berikan top 5 upt yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt <CONCLUSION TYPE>?` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | OK → **OK** · 5 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
SELECT nama_balai, count(*) FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 5
```

### [30] Tampilkan data gap/perbedaan hasil pengawasan penandaan obat antara pusat dan UPT

| | |
|---|---|
| Bentuk NER | `Tampilkan data gap/perbedaan hasil pengawasan penandaan <COMMODITY NAME> antara pusat dan UPT` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | OK → **OK** · 6,988 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
SELECT mp.tgl_start, mp.kesimpulan_penilaian_balai, mp.kesimpulan_penilaian_pusat, COUNT(*) AS jumlah_gap FROM mv_penandaan mp WHERE mp.kesimpulan_penilaian_balai IS NOT NULL AND mp.kesimpulan_penilaian_pusat IS NOT NULL AND mp.kesimpulan_penilaian_balai <> mp.kesimpulan_penilaian_pusat GROUP BY mp.tgl_start, mp.kesimpulan_penilaian_balai, mp.kesimpulan_penilaian_pusat ORDER BY mp.tgl_start;
```

### [31] tampilkan data obat tradisional; suplemen kesehatan; obat kuasi hasil pengawasan penandaan yang merupakan kategori obat tradisional; suplemen kesehatan; obat kuasi dan klaim dalam promosi/iklan.

| | |
|---|---|
| Bentuk NER | `tampilkan data <CLASSIFICATION>; <CLASSIFICATION>; <CLASSIFICATION> hasil pengawasan penandaan yang merupakan kategori <CLASSIFICATION>; <CLASSIFICATI` |
| Tabel | `mv_penandaan` · agregasi: tidak |
| Status | OK → **OK** · 52,685 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
SELECT * FROM public.mv_penandaan WHERE komoditi IN ('OBAT TRADISIONAL (OT)', 'SUPLEMEN KESEHATAN', 'OBAT KUASI');
```

### [32] Tampilkan data pemenuhan timeline pengawasan oleh masing-masing UPT yang dapat diukur sejak tanggal pemeriksaan sampai dengan tanggal laporan dikirim ke Pusat oleh kepala UPT

| | |
|---|---|
| Bentuk NER | `Tampilkan data pemenuhan timeline pengawasan oleh masing-masing UPT yang dapat diukur sejak tanggal pemeriksaan sampai dengan tanggal laporan dikirim ` |
| Tabel | `mv_penandaan + mv_penandaan_timeline` · agregasi: tidak |
| Status | OK → **OK** · 212,688 baris |
| Lapis terjemahan | - |
| Diagnosa | **⚠️ Jalan tapi >100rb baris tanpa agregasi** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah. TAPI hasilnya 212,688 baris tanpa agregasi — jalan, bukan jawaban |

```sql
select mp.nama_balai, mp.id, mpt.tgl_start, mpt.tanggal_kirim_pusat, (mpt.mulai_kabalai + mpt.kabalai_direktur + mpt.direktur_pusat) as durasi_hari from mv_penandaan mp join mv_penandaan_timeline mpt on mp.id = mpt.id_penandaan where tanggal_kirim_direktur is not null order by 1, 2
```

### [33] Tampilkan jumlah label tepat waktu yang telah dikirimkan ke pusat pada UPT tertentu pada tahun 2025 Hasil pengawasan label dikirimkan ke pusat maksimal tanggal 15 bulan berikutnya

| | |
|---|---|
| Bentuk NER | `Tampilkan jumlah label tepat waktu yang telah dikirimkan ke pusat pada UPT tertentu pada tahun <YEAR> Hasil pengawasan label dikirimkan ke pusat maksi` |
| Tabel | `mv_penandaan + mv_penandaan_timeline + mpt` · agregasi: ya |
| Status | OK → **OK** · 77 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
SELECT mp.nama_balai, COUNT(*) AS jumlah_laporan, SUM( CASE WHEN mpt.tanggal_kirim_kabalai < (DATE_TRUNC('month', mp.tgl_start) + INTERVAL '1 month' + INTERVAL '14 day') THEN 1 ELSE 0 END ) AS laporan_tepat_waktu, ROUND( SUM( CASE WHEN mpt.tanggal_kirim_kabalai < (DATE_TRUNC('month', mp.tgl_start) + INTERVAL '1 month' + INTERVAL '14 day') THEN 1 ELSE 0 END )::DECIMAL / COUNT(*) * 100, 2 ) AS persentase_tepat_waktu FR
```

### [34] tampilkan nama upt yang tidak melakukan input catatan tmk pada hasil pengawasan penandaan dengan kesimpulan tmk.

| | |
|---|---|
| Bentuk NER | `tampilkan nama upt yang tidak melakukan input catatan <CONCLUSION TYPE> pada hasil pengawasan penandaan dengan kesimpulan <CONCLUSION TYPE>.` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | OK → **OK** · 84 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
SELECT nama_balai, count(*) as jumlah_penandaan FROM public.mv_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' AND (catatan IS NULL OR catatan = '') group by nama_balai order by jumlah_penandaan desc;
```

### [35] tampilkan nama upt yang tidak melakukan input catatan tmk pada hasil pengawasan penandaan dengan kesimpulan tmk.

| | |
|---|---|
| Bentuk NER | `tampilkan nama upt yang tidak melakukan input catatan <CONCLUSION TYPE> pada hasil pengawasan penandaan dengan kesimpulan <CONCLUSION TYPE>.` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | OK → **OK** · 84 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
SELECT nama_balai, count(*) as jumlah_penandaan FROM public.mv_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' AND (catatan IS NULL OR catatan = '') group by nama_balai order by jumlah_penandaan desc;
```

### [36] Tampilkan rekapitulasi jumlah laporan pengawasan penandaan dengan hasil kesimpulan akhir yaitu memenuhi ketentuan berdasarkan tanggal proses direktur untuk tahun 2025

| | |
|---|---|
| Bentuk NER | `Tampilkan rekapitulasi jumlah laporan pengawasan penandaan dengan hasil kesimpulan akhir yaitu memenuhi ketentuan berdasarkan tanggal proses direktur ` |
| Tabel | `mv_penandaan + mv_penandaan_timeline + mp` · agregasi: ya |
| Status | OK → **OK** · 214 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
select mpt.tanggal_kirim_direktur, mp.kesimpulan_penilaian_pusat, count(*) from mv_penandaan mp join mv_penandaan_timeline mpt on mp.id = mpt.id_penandaan where mp.kesimpulan_penilaian_pusat is not null and extract(year from mp.tgl_start) = 2025 and mpt.tanggal_kirim_direktur is not null and mp.kesimpulan_penilaian_pusat = 'MK' group by 1, 2 order by 1, 2
```

### [37] tampilkan rekapitulasi jumlah laporan pengawasan penandaan dengan hasil kesimpulan akhir yaitu tidak memenuhi ketentuan berdasarkan tanggal proses direktur

| | |
|---|---|
| Bentuk NER | `tampilkan rekapitulasi jumlah laporan pengawasan penandaan dengan hasil kesimpulan akhir yaitu <CONCLUSION TYPE> berdasarkan tanggal proses direktur` |
| Tabel | `mv_penandaan + mv_penandaan_timeline + mp` · agregasi: ya |
| Status | OK → **OK** · 214 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
select mpt.tanggal_kirim_direktur, mp.kesimpulan_penilaian_pusat, count(*) from mv_penandaan mp join mv_penandaan_timeline mpt on mp.id = mpt.id_penandaan where mp.kesimpulan_penilaian_pusat is not null and extract(year from mp.tgl_start) = 2025 and mpt.tanggal_kirim_direktur is not null and mp.kesimpulan_penilaian_pusat = 'TMK' group by 1, 2 order by 1, 2
```

### [38] Tampilkan rekapitulasi jumlah laporan pengawasan penandaan masing-masing UPT yang telah dikirim ke Pusat pada tahun 2025

| | |
|---|---|
| Bentuk NER | `Tampilkan rekapitulasi jumlah laporan pengawasan penandaan masing-masing UPT yang telah dikirim ke Pusat pada tahun <YEAR>` |
| Tabel | `mv_penandaan + mp` · agregasi: ya |
| Status | OK → **OK** · 229 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
select mp.nama_balai, mp.kesimpulan_penilaian_pusat, count(*) from mv_penandaan mp where mp.kesimpulan_penilaian_pusat is not null and extract(year from mp.tgl_start) = 2025 group by 1, 2 order by 1, 2
```

### [39] tampilkan urutan data nama sarana produksi dengan hasil pengawasan akhir dari pusat yang memiliki obat tradisional; suplemen kesehatan; obat kuasi dengan penandaan yang tidak memenuhi ketentuan diurut

| | |
|---|---|
| Bentuk NER | `tampilkan urutan data nama <FACILITY TYPE> dengan hasil pengawasan akhir dari pusat yang memiliki <COMMODITY NAME>; <COMMODITY NAME>; <COMMODITY NAME>` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | OK → **OK** · 1,228 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
SELECT produsen, COUNT(*) AS jumlah_temuan FROM public.mv_penandaan WHERE komoditi IN ('OBAT TRADISIONAL (OT)', 'SUPLEMEN KESEHATAN', 'OBAT KUASI') AND kesimpulan_penilaian_pusat = 'TMK' GROUP BY produsen ORDER BY jumlah_temuan DESC;
```

### [40] upt mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt memenuhi ketentuan?

| | |
|---|---|
| Bentuk NER | `upt mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt <CONCLUSION TYPE>?` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | OK → **OK** · 10 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
SELECT nama_balai FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'MK' GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 10;
```

### [41] upt mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt tidak memenuhi ketentuan?

| | |
|---|---|
| Bentuk NER | `upt mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt <CONCLUSION TYPE>?` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | OK → **OK** · 10 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
SELECT nama_balai FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_balai ORDER BY count(*) DESC LIMIT 10;
```

### [42] urutkan nama kemasan yang memiliki kesimpulan akhir pengawasan penandaan tmk paling banyak sampai dengan paling rendah.

| | |
|---|---|
| Bentuk NER | `urutkan nama kemasan yang memiliki kesimpulan akhir <CONCLUSION TYPE> paling banyak sampai dengan paling rendah.` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | OK → **OK** · 15,529 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
SELECT nama_produk, count(*) FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_produk ORDER BY COUNT(*) DESC;
```

### [43] urutkan nama kemasan yang memiliki kesimpulan akhir pengawasan penandaan tmk paling banyak sampai dengan paling rendah.

| | |
|---|---|
| Bentuk NER | `urutkan nama kemasan yang memiliki kesimpulan akhir <CONCLUSION TYPE> paling banyak sampai dengan paling rendah.` |
| Tabel | `mv_penandaan` · agregasi: ya |
| Status | OK → **OK** · 15,529 baris |
| Lapis terjemahan | - |
| Diagnosa | **✅ Jalan apa adanya** |
| Sebab | SQL generasi berjalan sudah cocok dengan skema live; tidak perlu diubah |

```sql
SELECT nama_produk, count(*) FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'TMK' GROUP BY nama_produk ORDER BY COUNT(*) DESC;
```
