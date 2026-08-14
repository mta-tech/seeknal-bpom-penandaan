# 21 — SQL Pairs Tervalidasi (Database penandaan)

> Kumpulan SQL pairs **baru yang ditulis & divalidasi langsung** terhadap database `penandaan` (bukan salinan dari hasil AI KAI).
> Setiap pair: contoh pertanyaan user asli → SQL → hasil eksekusi nyata. Dipakai agent untuk menjawab pola pertanyaan
> yang paling sering muncul (lihat [18](18_pola_pertanyaan_user.md)).

**Cara pakai:**
```bash
export PGURL=$(grep '^WAREHOUSE_URL=' /home/mta/projects/seeknal_audit/seeknal-bpom-penandaan/.env | cut -d= -f2- | tr -d '"')
psql "$PGURL" -c "<SQL>"
```

> Snapshot sync: 2026-08-11 22:53:20. Semua hasil di bawah = eksekusi nyata.

---

## SQL-01 — Total Penandaan per Tahun

**Pertanyaan user:** "berikan total penandaan tahun 2025" / "Berapa jumlah penandaan selama tahun 2024"

```sql
SELECT EXTRACT(YEAR FROM tgl_start)::int AS tahun, COUNT(*) AS jumlah
FROM mv_penandaan
WHERE tgl_start IS NOT NULL
GROUP BY 1 ORDER BY 1;
```

**Hasil (verifikasi):**

| tahun | jumlah |
|---|---:|
| 2023 | 88.878 |
| 2024 | 91.167 |
| 2025 | 68.902 |
| 2026 | 43.811 |

---

## SQL-02 — Gap Kesimpulan Pusat vs UPT

**Pertanyaan user:** "tampilkan data gap/perbedaan hasil pengawasan penandaan obat antara pusat dan upt"

```sql
SELECT nama_balai, kesimpulan_penilaian_balai, kesimpulan_penilaian_pusat, COUNT(*) AS n
FROM mv_penandaan
WHERE kesimpulan_penilaian_balai IS NOT NULL
  AND kesimpulan_penilaian_pusat IS NOT NULL
  AND kesimpulan_penilaian_balai <> kesimpulan_penilaian_pusat
GROUP BY 1,2,3 ORDER BY 4 DESC;
```

**Hasil (verifikasi, total 152.577 baris gap):**

| nama_balai | kesimpulan_balai | kesimpulan_pusat | n |
|---|---|---|---:|
| BALAI BESAR POM DI JAKARTA | MK | VP | 2.836 |
| BALAI BESAR POM DI DENPASAR | MK | VP | 2.543 |
| BALAI BESAR POM DI BANDUNG | MK | VP | 2.428 |
| BALAI BESAR POM DI SURABAYA | (null) | TMK | 2.309 |
| BALAI BESAR POM DI BANDUNG | (null) | TMK | 2.269 |

> ⚠️ Termasuk baris `kesimpulan_balai = NULL` — gap ini = UPT belum menilai tapi pusat sudah putus. Lihat
> [06_penilaian_keputusan](06_penilaian_keputusan.md).

**Varian persamaan:** ganti `<>` menjadi `=`.

---

## SQL-03 — Rekapitulasi per UPT

**Pertanyaan user:** "tampilkan rekapitulasi jumlah laporan pengawasan penandaan masing-masing upt yang telah dikirim ke pusat pada periode waktu antara tanggal 1 januari 2025 hingga 31 desember 2025."

```sql
SELECT p.nama_balai, COUNT(*) AS jumlah
FROM mv_penandaan p
JOIN mv_penandaan_timeline t ON t.id_penandaan = p.id
WHERE t.tanggal_kirim_pusat BETWEEN '2025-01-01' AND '2025-12-31'
GROUP BY 1 ORDER BY 2 DESC;
```

**Hasil (verifikasi, top 5):**

| nama_balai | jumlah |
|---|---:|
| BALAI BESAR POM DI SURABAYA | 11.391 |
| BALAI BESAR POM DI BANDUNG | 11.348 |
| BALAI BESAR POM DI JAKARTA | 11.277 |
| BALAI BESAR POM DI YOGYAKARTA | 10.248 |
| BALAI BESAR POM DI DENPASAR | 9.979 |

> ⚠️ Tanpa filter tanggal hasil sama (data didominasi periode ini). Tambah filter `komoditi`/`kesimpulan` sesuai kebutuhan.

---

## SQL-04 — Ranking UPT Tertinggi / Terendah

**Pertanyaan user:** "Balai mana yang memiliki jumlah penandaan tertinggi?" / "terendah?"

```sql
SELECT nama_balai, COUNT(*) AS jumlah
FROM mv_penandaan
GROUP BY 1 ORDER BY 2 DESC LIMIT 10;  -- ganti DESC→ASC utk terendah
```

**Hasil (verifikasi, top):** Surabaya 11.391, Bandung 11.348, Jakarta 11.277, Yogyakarta 10.248, Denpasar 9.979.

---

## SQL-05 — Rekap MK/TMK per Komoditi

**Pertanyaan user:** "Kategori produk apa yang memiliki jumlah 'Tidak Memenuhi Ketentuan TMK' tertinggi?"

```sql
SELECT komoditi, kesimpulan_penilaian_pusat, COUNT(*) AS n
FROM mv_penandaan
WHERE kesimpulan_penilaian_pusat IN ('MK','TMK')
GROUP BY 1,2 ORDER BY 1, 3 DESC;
```

**Hasil (verifikasi, sari):**

| komoditi | MK | TMK |
|---|---:|---:|
| KOSMETIKA | 62.885 | 11.551 |
| OBAT | 5.053 | 48.122 |
| OBAT KUASI | 2.490 | 156 |
| OBAT TRADISIONAL (OT) | 34.524 | 4.013 |
| SUPLEMEN KESEHATAN | 9.988 | 619 |
| PRODUK PANGAN | 16.840 | (TMK ≤ 0 pada level pusat) |
| ROKOK | 19.995 | 12.086 |

---

## SQL-06 — UPT Tanpa Input Catatan TMK

**Pertanyaan user:** "tampilkan nama upt yang tidak melakukan input catatan tmk pada hasil pengawasan penandaan dengan kesimpulan tmk."

```sql
SELECT nama_balai, COUNT(*) AS jumlah_tmk_tanpa_catatan
FROM mv_penandaan
WHERE kesimpulan_penilaian_pusat = 'TMK'
  AND (catatan IS NULL OR TRIM(catatan) = '')
GROUP BY 1 ORDER BY 2 DESC;
```

**Hasil (verifikasi):** Surabaya 2.309, Bandung 2.269, Jakarta 2.163, Semarang 2.041, Yogyakarta 2.030.

---

## SQL-07 — Rekap Berdasarkan Tanggal Proses Direktur

**Pertanyaan user:** "tampilkan rekapitulasi jumlah laporan pengawasan penandaan dengan hasil kesimpulan akhir yaitu memenuhi ketentuan berdasarkan tanggal proses direktur."

```sql
SELECT date_trunc('month', t.tanggal_kirim_direktur)::date AS bulan,
       COUNT(*) AS jumlah
FROM mv_penandaan p
JOIN mv_penandaan_timeline t ON t.id_penandaan = p.id
WHERE t.tanggal_kirim_direktur IS NOT NULL
  AND p.kesimpulan_penilaian_pusat = 'MK'          -- 'TMK' utk tidak memenuhi
GROUP BY 1 ORDER BY 1 DESC;
```

**Hasil (verifikasi, tanpa filter MK):** 2026-08: 2.202, 2026-07: 4.110, 2026-06: 5.703, 2026-05: 4.693, 2026-04: 5.777.

> Kolom `tanggal_kirim_direktur` ada di `mv_penandaan_timeline` (bukan fact). Wajib join.

---

## SQL-08 — Timeline Pemenuhan per UPT

**Pertanyaan user:** "tampilkan data pemenuhan timeline pengawasan oleh masing-masing upt yang dapat diukur sejak tanggal pemeriksaan sampai dengan tanggal laporan dikirim ke pusat oleh kepala upt."

```sql
SELECT p.nama_balai,
       COUNT(*) AS total,
       ROUND(AVG(t.tanggal_kirim_pusat - p.tgl_start), 1) AS avg_hari_pengiriman
FROM mv_penandaan p
JOIN mv_penandaan_timeline t ON t.id_penandaan = p.id
WHERE t.tanggal_kirim_pusat IS NOT NULL
GROUP BY 1 ORDER BY 3 DESC;
```

**Hasil (verifikasi):** Rata-rata nasional **14,9 hari** (n=292.758). Lambat: Kediri 31,3; Lubuklinggau 28,4; Balikpapan 26,1; Manokwari 25,5; Bandung 25,1.

> SLA user: "dikirim ke pusat maksimal tanggal 15 bulan berikutnya". Lihat [09_waktu_durasi](09_waktu_durasi.md).

---

## SQL-09 — Komoditi dengan TMK Tertinggi

**Pertanyaan user:** "Kategori produk manakah yang memiliki jumlah penandaan 'TMK' Tidak Memenuhi Ketentuan tertinggi?"

```sql
SELECT komoditi, COUNT(*) AS jumlah_tmk
FROM mv_penandaan
WHERE kesimpulan_penilaian_pusat = 'TMK'
GROUP BY 1 ORDER BY 2 DESC;
```

**Hasil (verifikasi):** OBAT 48.122, ROKOK 12.086, KOSMETIKA 11.551, OT 4.013, SUPLEMEN KESEHATAN 619, OBAT KUASI 156.

---

## SQL-10 — Triplet OT / Supkes / Kuasi

**Pertanyaan user:** "tampilkan data obat tradisional; suplemen kesehatan; obat kuasi ..." (triplet khas)

```sql
SELECT komoditi, kesimpulan_penilaian_pusat, COUNT(*) AS n
FROM mv_penandaan
WHERE komoditi IN ('OBAT TRADISIONAL (OT)','SUPLEMEN KESEHATAN','OBAT KUASI')
GROUP BY 1,2 ORDER BY 1, 3 DESC;
```

**Hasil (verifikasi):**

| komoditi | MK | TMK | VP |
|---|---:|---:|---:|
| OBAT TRADISIONAL (OT) | 34.524 | 4.013 | 652 |
| SUPLEMEN KESEHATAN | 9.988 | 619 | 150 |
| OBAT KUASI | 2.490 | 156 | 42 |

---

## Ringkasan

| SQL | Pola user | Tabel | Status |
|---|---|---|---|
| 01 | Total per tahun | fact | ✅ |
| 02 | Gap pusat vs UPT | fact | ✅ |
| 03 | Rekap per UPT | fact + timeline | ✅ |
| 04 | Ranking UPT | fact | ✅ |
| 05 | MK/TMK per komoditi | fact | ✅ |
| 06 | UPT tanpa catatan TMK | fact | ✅ |
| 07 | Rekap per tanggal direktur | fact + timeline | ✅ |
| 08 | Timeline pemenuhan | fact + timeline | ✅ |
| 09 | Komoditi TMK tertinggi | fact | ✅ |
| 10 | Triplet OT/Supkes/Kuasi | fact | ✅ |

> Semua query di atas **terverifikasi jalan** terhadap database penandaan. SQL dari export KAI (`kai_question_sql_pairs.csv`)
> TIDAK dipakai karena hasil AI belum tervalidasi.

---

Lanjut ke [22_prioritas_use_case.md](22_prioritas_use_case.md) untuk ranking prioritas.

---

## §21.B — Hasil Eksekusi 43 SQL Pair KAI ke DB Live (2026-08-13)

Dokumen ini menyatakan SQL export KAI "TIDAK dipakai karena hasil AI belum tervalidasi". Bagian
ini memberi **buktinya**: seluruh 43 pasangan `context_stores` domain penandaan dijalankan apa
adanya terhadap DB live (`SELECT count(*) FROM (<sql>) q`).

### Hasil per generasi koneksi

| Generasi | Pair | OK | ERROR | Sebab kegagalan |
|---|--:|--:|--:|---|
| `penandaan` (v1, Jul 2025) | 13 | **0** | 13 | `vw_penandaan` — relasi tidak ada |
| `penandaan_all` (v2, Ags 2025) | 10 | **0** | 10 | idem |
| `penandaan_all_v2` (v3, Nov 2025) | 20 | **19** | 1 | `column mp.kabupaten does not exist` |

**24 dari 43 pair (56%) mati.** Dua generasi pertama seluruhnya menunjuk view `vw_penandaan`
yang sudah diganti `mv_penandaan`.

### §21.C — Drift skema: `provinsi` & `kabupaten` sudah dihapus

```sql
SELECT string_agg(column_name, ', ' ORDER BY ordinal_position)
FROM information_schema.columns
WHERE table_schema='public' AND table_name='mv_penandaan';
-- id, komoditi, nomor_surat, nomorsampel, tgl_start, tgl_end, nama_produk, ed_nie,
-- pendaftar, produsen, nama_balai, kesimpulan_penilaian_balai, kesimpulan_penilaian_pusat,
-- catatan, sync                                            → 15 kolom
```

`table_descriptions` KAI generasi `_all_v2` (Nov 2025) mencatat **17 kolom** termasuk `provinsi`
dan `kabupaten`, lengkap dengan contoh baris (`"kabupaten": "KOTA PANGKAL PINANG"`). Kedua kolom
itu **sudah tidak ada** di live.

**Konsekuensi langsung:**

- `BPOM User Relevant Query` **#98** (*"Berdasarkan kabupaten kota dan provinsi alamat produsen,
  tampilkan berapa jumlah label yang dilaporkan MK atau TMK"*) → **NOT COVERED**.
- Setiap klaim geografi di `10_geografi_kapasitas.md` yang bersandar pada `mv_penandaan.kabupaten`
  perlu diperiksa ulang; satu-satunya sumber geografi yang tersisa adalah `coverage_balai`
  (wilayah kerja balai), yang **bukan** alamat produsen.

### §21.D — Jebakan `IS NOT NULL` pada sentinel `''` — pair #17 melaporkan gap palsu

Pair paling sering ditanya di domain ini (17× di log KAI, dan `SQL Training` #17 pada
`BPOM User Relevant Query`) berbunyi:

```sql
SELECT tgl_start, kesimpulan_penilaian_balai, kesimpulan_penilaian_pusat, COUNT(*)
FROM mv_penandaan
WHERE kesimpulan_penilaian_balai IS NOT NULL
  AND kesimpulan_penilaian_pusat IS NOT NULL
  AND kesimpulan_penilaian_balai <> kesimpulan_penilaian_pusat
GROUP BY 1,2,3;
```

SQL ini **jalan** (masuk kategori OK di atas) tetapi hasilnya salah untuk komoditi OBAT dan ROKOK.
Sebabnya: sentinel di kolom ini adalah **string kosong `''`**, dan `'' IS NOT NULL` bernilai TRUE.

```sql
SELECT kesimpulan_penilaian_balai, kesimpulan_penilaian_pusat, count(*)
FROM mv_penandaan WHERE komoditi='OBAT' GROUP BY 1,2 ORDER BY 3 DESC;
--  ''  | TMK | 48122
--  ''  | MK  |  5053
```

Seluruh 53.175 baris OBAT lolos filter, dan `'' <> 'TMK'` juga TRUE — sehingga query melaporkan
**48.122 "gap"** yang sebenarnya berarti *"balai tidak pernah menilai"*, bukan *"balai dan pusat
berbeda pendapat"*. Ini sejalan dengan `04_komoditi_governing_dimension.md` yang sudah mencatat
"OBAT & ROKOK: balai 100% skip" — yang belum tercatat adalah **bahwa SQL resmi domain ini
menghitungnya sebagai gap**.

**Populasi yang benar-benar bisa dibandingkan** (balai terisi DAN pusat sudah berkeputusan):

```sql
SELECT komoditi, count(*) AS n,
       count(*) FILTER (WHERE kesimpulan_penilaian_balai <> ''
                          AND kesimpulan_penilaian_pusat NOT IN ('','VP')) AS bisa_dibandingkan,
       count(*) FILTER (WHERE kesimpulan_penilaian_balai <> ''
                          AND kesimpulan_penilaian_pusat NOT IN ('','VP')
                          AND kesimpulan_penilaian_balai <> kesimpulan_penilaian_pusat) AS beda
FROM mv_penandaan GROUP BY 1 ORDER BY 3 DESC;
```

| Komoditi | Baris | Bisa dibandingkan | Benar-benar beda |
|---|--:|--:|--:|
| KOSMETIKA | 78.550 | 74.436 | **6.336** |
| OBAT TRADISIONAL (OT) | 39.189 | 38.537 | 643 |
| PRODUK PANGAN | 75.629 | 21.443 | 374 |
| SUPLEMEN KESEHATAN | 10.757 | 10.607 | 109 |
| OBAT KUASI | 2.688 | 2.646 | 28 |
| KEMASAN PANGAN | 689 | 2 | 0 |
| **OBAT** | 53.175 | **0** | **0** |
| **ROKOK** | 32.081 | **0** | **0** |

**Aturan pair:** pertanyaan "gap balai vs pusat" **wajib** menyebut komoditi. Untuk OBAT/ROKOK
jawabannya adalah *"penilaian UPT tidak direkam untuk komoditi ini — gap tidak terdefinisi"*,
bukan angka.

### §21.E — Bukti mekanis `VP` = tahap proses, bukan putusan

`06_penilaian_keputusan.md` menyebut VP sebagai "keranjang penampung keraguan" berdasar pola
distribusi. Berikut buktinya dari sisi timeline:

```sql
SELECT p.kesimpulan_penilaian_pusat, count(*) AS n,
       count(t.tanggal_kirim_direktur) AS ada_direktur,
       count(t.tanggal_kirim_pusat)    AS ada_pusat
FROM mv_penandaan p
LEFT JOIN mv_penandaan_timeline t ON t.id_penandaan = p.id
GROUP BY 1 ORDER BY 2 DESC;
```

| `pusat` | Baris | Ada tgl direktur | Ada tgl pusat |
|---|--:|--:|--:|
| MK | 151.777 | 134.114 (88,4%) | 151.777 (100%) |
| TMK | 76.547 | 75.837 (99,1%) | 76.547 (100%) |
| **VP** | **59.831** | **2.663 (4,5%)** | 59.831 (100%) |
| TMK MAYOR | 2.367 | **0** | 2.367 |
| TMK MINOR | 2.236 | **0** | 2.236 |

VP sudah sampai pusat (100%) tetapi **hampir tidak pernah diproses direktur (4,5%)**, sementara
MK/TMK mencapai 88–99%. Jadi VP adalah **keadaan menunggu**, dan setiap hitungan MK/TMK wajib
mengeluarkannya sambil menyebut porsinya. Catatan tambahan: `TMK MAYOR` dan `TMK MINOR`
**tidak pernah** mencapai direktur (0 dari 4.603 baris).

### §21.F — `mv_penandaan_agg` boleh dipakai, satu syarat

```sql
SELECT periode_type, count(*) AS baris, sum(jumlah_penandaan) FROM mv_penandaan_agg GROUP BY 1;
--  month | 30750 | 292758
--  day   | 63987 | 292758
```

Tiap `periode_type` menjumlah **tepat** ke cacah fakta (292.758). Yang menggandakan bukan tabelnya,
melainkan **lupa memfilter `periode_type`**. Pola identik berlaku di `mv_pemeriksaan_agg`,
`mv_pengawasan_agg`, dan `mv_sampel_agg`.

---

## §21.G — Eksekusi 8 `SQL Training` dari `BPOM User Relevant Query` (2026-08-13)

Kolom **SQL Training** di CSV berisi SQL tulisan tim (bukan hasil AI). Untuk modul `penandaan` ada
8; **7 jalan, 1 gagal**.

| # | Pertanyaan | Baris | Catatan |
|---|---|--:|---|
| **17** | **Gap hasil penandaan obat: pusat vs UPT** | **6.981** | ⚠️ angka ini **bukan gap** — lihat §21.D |
| 18 | Pemenuhan timeline sampai kirim pusat | **212.614** | ⚠️ melebihi cacah fakta 292.758? tidak — tapi menarik dari timeline (500.820 id) |
| 21 | Rekap laporan penandaan per UPT dikirim ke pusat 2025 | 229 | |
| 22 | Rekap kesimpulan akhir **MK** per tanggal direktur | 214 | |
| 23 | Rekap kesimpulan akhir **TMK** per tanggal direktur | 214 | angka identik dengan #22 — keduanya menghasilkan 214 kelompok tanggal |
| 93 | Label tepat waktu (< tgl 15 bulan berikutnya) 2025 | 77 | 66.921 dari 69.173 laporan tepat waktu |
| 96 | Per produsen: MK/TMK + beda balai-pusat | 3 | contoh "dua kelinci" |
| **98** | **Per kabupaten/provinsi alamat produsen** | **GAGAL** | `column mp.kabupaten does not exist` — lihat §21.C |

### #17 menghasilkan 6.981 baris, dan itu bukan gap

SQL #17 mengelompokkan per `(tgl_start, balai, pusat)` tanpa memfilter sentinel `''`, sehingga
6.981 kelompok yang dilaporkan **didominasi pasangan `('', 'TMK')` dan `('', 'MK')`** — yaitu baris
komoditi OBAT dan ROKOK yang balainya memang tidak pernah menilai. Rincian dan angka yang benar
ada di §21.D.

### #18 menarik dari sisi timeline

`mv_penandaan_timeline` punya 500.820 id sementara fakta 292.758. Query #18 melakukan
`JOIN mv_penandaan … mv_penandaan_timeline` sehingga terbatas pada irisan, tetapi 212.614 baris
hasilnya tetap **bukan** cacah laporan — melainkan cacah baris berpasangan yang punya
`tanggal_kirim_direktur`. Untuk "berapa laporan", pakai `COUNT(DISTINCT mp.id)`.

### Status pertanyaan CSV modul `penandaan`

8 pertanyaan bermodul `penandaan`, kolom *Pengecekan Data* menyatakan **seluruhnya "sudah
tersedia"**. Hasil eksekusi: 7 jalan, 1 gagal struktural (geografi), dan **1 di antara yang
"jalan" (#17) memberi angka yang menyesatkan**. Status "tersedia" di CSV menandai ketersediaan
kolom, bukan kebenaran jawabannya.

Catatan tambahan: empat pertanyaan CSV lain (#24, #25, #70, #71) meminta rekap **per jenis
kemasan**. Keempatnya diberi status "Done add mv_*_timeline" di CSV, padahal yang dibutuhkan bukan
timeline melainkan kolom jenis kemasan — yang **tidak ada** di `mv_penandaan` (15 kolom).
Lihat `20_gap_schema_user.md` G1.

---

## §21.H — Terjemahan `vw_penandaan` → `mv_penandaan`: 21 dari 24 pair "mati" bisa dipulihkan

§21.B menyimpulkan 24 pair generasi v1/v2 mati. Setelah diterjemahkan ke skema live, **21 langsung
menghasilkan data**.

| | Sebelum terjemahan | Sesudah terjemahan |
|---|--:|--:|
| Pair OK | 19 / 43 (44%) | **40 / 43 (93%)** |
| Gagal skema | 24 | **1** (kolom `kabupaten` yang memang dihapus) |
| Jalan tapi nol baris | 0 | 2 |

### Peta terjemahan — nama kolom identik, hanya relasi yang berubah

| `vw_penandaan` (14 kolom) | `mv_penandaan` (15 kolom) |
|---|---|
| `id`, `komoditi`, `nomor_surat`, `nomorsampel`, `tgl_start`, `tgl_end`, `nama_produk`, `ed_nie`, `pendaftar`, `produsen`, `nama_balai`, `kesimpulan_penilaian_pusat`, `catatan`, `sync` | **nama identik** |
| — | **`kesimpulan_penilaian_balai` ditambahkan** di skema baru |

Penambahan `kesimpulan_penilaian_balai` inilah yang membuat pertanyaan "gap balai vs pusat"
menjadi mungkin — dan sekaligus membuka jebakan `''` yang dibahas di §21.D.

⚠️ Satu perubahan makna yang mudah terlewat: pada contoh baris generasi v1, kolom **`catatan`
berisi `"MK"`** — yaitu nilai verdict, bukan catatan bebas. Di live, `catatan` adalah teks bebas
dan 74,2% kosong pada baris TMK (§21.D). Pair lama yang memfilter `catatan = 'MK'` karena itu
bukan sekadar perlu ganti nama tabel — ia salah kolom.

### 2 pair yang tetap nol baris

Keduanya pertanyaan yang sama: *"tampilkan nama UPT yang tidak melakukan input catatan TMK pada
hasil pengawasan penandaan dengan kesimpulan TMK"*. SQL-nya mencari UPT yang **sama sekali tidak
pernah** mengisi catatan pada baris TMK. Himpunan itu kosong: setiap UPT punya minimal satu baris
TMK bercatatan. Jawaban yang benar bukan daftar kosong, melainkan **peringkat persentase** —
74,2% baris TMK secara nasional tidak bercatatan (§21.D).

### Ringkas: tiga lapis terjemahan yang dibutuhkan domain ini

1. **Relasi** — `vw_penandaan` → `mv_penandaan` (menyelamatkan 21 pair).
2. **Kolom** — tidak ada perubahan nama; hanya penambahan `kesimpulan_penilaian_balai`.
3. **Semantik** — `catatan` berubah peran (dulu memuat verdict, kini teks bebas), dan sentinel
   `''` pada kolom verdict mengubah cara menulis filter. Lapis ini tidak bisa diotomatiskan.

---

## §21.I — Perlakuan terhadap pair yang tetap gagal: ditulis ulang, bukan dibiarkan

Setelah terjemahan (§21.H), tiga pair masih bermasalah. Semuanya ditelusuri lalu ditulis ulang dan
diuji.

### (1) Anti-join `catatan IS NULL` — dua lapis kesalahan sekaligus

```sql
-- ASLI → 0 baris
SELECT DISTINCT nama_balai FROM mv_penandaan
WHERE catatan IS NULL AND kesimpulan_penilaian_pusat = 'TMK';
```

Dua hal salah:

1. **`catatan` tidak pernah SQL NULL** — sentinelnya string kosong `''`. `IS NULL` tidak pernah
   cocok.
2. **`= 'TMK'` melewatkan `TMK MAYOR` dan `TMK MINOR`** (4.603 baris).

Dan bahkan setelah dibetulkan, bentuk "UPT yang **tidak pernah** mengisi catatan" tetap kosong —
setiap UPT punya minimal satu baris TMK bercatatan. Bentuk yang menjawab maksud penanya adalah
peringkat porsi:

```sql
SELECT nama_balai, count(*) AS tmk,
       count(*) FILTER (WHERE coalesce(trim(catatan),'') = '') AS tanpa_catatan,
       round(100.0*count(*) FILTER (WHERE coalesce(trim(catatan),'') = '')/count(*),1) AS pct
FROM mv_penandaan
WHERE kesimpulan_penilaian_pusat LIKE 'TMK%'
GROUP BY 1 ORDER BY 4 DESC;
```

Konteks nasionalnya: **74,2%** baris TMK tidak bercatatan (60.208 dari 81.150). Jadi jawaban yang
jujur bukan daftar UPT, melainkan "ini masalah menyeluruh; berikut UPT dengan porsi tertinggi".

### (2) Kolom yang benar-benar dihapus

*"berdasarkan kabupaten/kota, tampilkan jumlah perbedaan kesimpulan balai dengan pusat"* gagal
karena `mv_penandaan` tidak lagi punya `kabupaten`/`provinsi` (§21.C). Pengganti terdekat adalah
per **balai**, dan itu unit pemeriksa — bukan alamat produsen:

```sql
SELECT mp.nama_balai, count(*) AS beda
FROM mv_penandaan mp
WHERE mp.kesimpulan_penilaian_balai <> ''
  AND mp.kesimpulan_penilaian_pusat NOT IN ('','VP')
  AND mp.kesimpulan_penilaian_balai <> mp.kesimpulan_penilaian_pusat
GROUP BY 1 ORDER BY 2 DESC;
```

Perhatikan bahwa query ini **juga** memperbaiki dua kesalahan lain dari SQL aslinya: menyaring
sentinel `''` (yang membuat OBAT/ROKOK terhitung sebagai gap) dan mengeluarkan `VP` (tahap
menunggu, bukan putusan). Sajikan hasilnya sebagai per-balai **dengan menyatakan pergantiannya**,
atau jawab NOT COVERED bila penanya benar-benar meminta wilayah produsen.

### Ringkas perlakuan

| Kategori | Jumlah | Perlakuan |
|---|--:|---|
| Anti-join dengan sentinel & family filter salah | 2 | diperbaiki + diubah bentuk menjadi peringkat porsi — **berhasil** |
| Kolom dihapus dari skema | 1 | **tidak bisa** — NOT COVERED, atau ganti ke per-balai dengan disebutkan |

---

## §21.J — Duplikasi & konsistensi: satu pertanyaan, banyak jawaban

Pertanyaan yang wajib dijawab sebelum memakai `context_stores` sebagai rujukan: **apakah pertanyaan
yang sama selalu menghasilkan SQL dan jawaban yang sama?** Jawabannya **tidak**.

### Angka untuk domain penandaan

| Ukuran | Nilai |
|---|--:|
| Pair tersimpan | 43 |
| **Pertanyaan unik** | **19** |
| Pertanyaan dengan >1 versi | 8 |
| — di antaranya **SQL-nya berbeda** | **8 (100%)** |
| — di antaranya **hasilnya berbeda** | **4** |
| Pertanyaan dengan selisih hasil >3× | 2 |
| Pair redundan persis (pertanyaan + SQL identik) | 7 |

**43 pair hanya mewakili 19 pertanyaan.** Setiap pertanyaan yang berulang punya SQL berbeda —
tidak satu pun konsisten.

### Kasus terburuk: `LIKE '%MK%'` mencampur vonis yang berlawanan

Pertanyaan *"UPT mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan
kesimpulan UPT memenuhi ketentuan?"* tersimpan dalam **empat versi**, semuanya di alias yang sama:

```sql
-- versi A → 10 baris
SELECT nama_balai FROM mv_penandaan WHERE kesimpulan_penilaian_pusat = 'MK'
GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 10;

-- versi B → 1 baris   (hanya beda LIMIT)
... LIMIT 1;

-- versi C → 5 baris   (LIMIT 5, DAN filternya berubah)
SELECT nama_balai, count(*) FROM mv_penandaan
WHERE kesimpulan_penilaian_pusat LIKE '%MK%'          -- ⚠️
GROUP BY nama_balai ORDER BY COUNT(*) DESC LIMIT 5;
```

Versi C bukan sekadar beda panjang daftar — **`LIKE '%MK%'` menangkap TMK juga**:

```sql
SELECT kesimpulan_penilaian_pusat, count(*) FROM mv_penandaan
WHERE kesimpulan_penilaian_pusat LIKE '%MK%' GROUP BY 1 ORDER BY 2 DESC;
--  MK 151.855 · TMK 76.614 · TMK MAYOR 2.367 · TMK MINOR 2.236
```

Jadi pertanyaan tentang **Memenuhi Ketentuan** dijawab dengan populasi yang **81.217 barisnya justru
Tidak Memenuhi Ketentuan** — vonis yang berlawanan. Peringkat UPT yang dihasilkan tidak sah.

Kesalahan yang sama ada di pasangan pertanyaannya (*"…tidak memenuhi ketentuan"*) yang memakai
`LIKE '%TMK%'` — kebetulan yang ini benar, karena `%TMK%` tidak menangkap `MK`. Satu pola tulis,
satu benar satu salah.

### Kenapa ini menghancurkan arsitektur berbasis pair

Pemilihan pair dilakukan dengan kemiripan embedding. Keempat versi di atas punya `prompt_text`
yang **identik**, sehingga kemiripannya sama besar — tidak ada mekanisme yang memilih versi benar.
Jawaban yang keluar bergantung pada versi mana yang kebetulan terambil: 10 UPT, 1 UPT, atau 5 UPT
dengan populasi tercemar.

**Implikasi untuk migrasi ke context skill:** yang harus dipindahkan adalah **pertanyaannya**
(19 unik), bukan SQL-nya (43 versi yang saling bertentangan). Aturan yang menggantikannya:
`kesimpulan_penilaian_pusat = 'MK'` untuk MK, dan `LIKE 'TMK%'` untuk keluarga TMK — tertulis
sekali di `context/`, bukan 43 kali di pair.
