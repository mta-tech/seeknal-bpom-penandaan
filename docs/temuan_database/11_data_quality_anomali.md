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

---

# Konsistensi penulisan nilai dan anomali tanggal

> Diverifikasi langsung ke warehouse, 14 Agustus 2026. Bagian ini menjawab satu pertanyaan: **apakah ada
> nilai yang maksudnya sama tetapi ditulis berbeda**, dan **apakah ada lubang atau tanggal mustahil
> pada rentang waktunya**. Seluruh isinya khusus domain ini.

Metodenya: tiap kolom berkode dinormalkan berlapis — rapatkan spasi, samakan besar-kecil huruf,
buang tanda baca, lalu kanonikkan angka (`5`, `5.0`, dan `05` dianggap satu). Nilai mentah yang
jatuh ke bentuk normal yang sama berarti **kembaran palsu**: dua baris berbeda di `GROUP BY`
padahal satu makna.

## K1. `kesimpulan_penilaian_balai` — satu gradasi, dua kapitalisasi

| Nilai tersimpan | Baris |
|---|---|
| `TMK MINOR` | 7.425 |
| `TMK Minor` | 1.900 |

Varian minoritas adalah **20% dari keluarga TMK MINOR** — bukan sisa yang bisa diabaikan.
Memfilter dengan `= 'TMK MINOR'` membuang seperlimanya; memfilter dengan `= 'TMK Minor'` membuang
empat perlimanya. Keduanya menghasilkan angka yang terlihat masuk akal.

Defek identik ada di `mv_penandaan_agg.kesimpulan_penilaian_balai`.

**Aturan:** keluarga TMK **wajib** dicocokkan dengan pola awalan **dan** tanpa peduli besar-kecil
huruf — `upper(kesimpulan_penilaian_balai) LIKE 'TMK%'` — bukan dengan kesamaan persis.

## K2. `trx_steps` — satu baris salah ketik di antara setengah juta

| Nilai | Baris |
|---|---|
| `spv_1` | 516.654 |
| `spv1` | **1** |

Satu baris tanpa garis bawah. Dampaknya kecil pada agregat, tetapi ia **menambah satu nilai palsu**
ke daftar tahap alur, dan setiap pemetaan tahap yang dibuat dari `SELECT DISTINCT trx_steps` akan
memuat langkah yang sebenarnya tidak ada.

**Aturan:** saat menyusun daftar tahap, abaikan nilai bervolume sangat kecil yang merupakan varian
penulisan dari nilai bervolume besar.

## K3. `pendaftar` — spasi ganda setelah bentuk badan usaha

Kolom nama pendaftar memuat puluhan kembaran yang lahir dari spasi ganda sesudah `PT` atau `CV`:

| Contoh | Baris |
|---|---|
| `PT SOHO INDUSTRI PHARMASI` versus `PT  SOHO INDUSTRI PHARMASI` | 1.060 vs 3 |
| `PT ULTRA SAKTI` versus `PT  ULTRA SAKTI` | 823 vs 84 |
| `PT DUA KELINCI` versus `PT  DUA KELINCI` | 191 vs 1 |
| `PT TIRTA ALAM SEGAR` versus `PT  TIRTA ALAM SEGAR` | 126 vs 1 |
| `CV  MANNA INDO LAKTA` versus `CV   MANNA INDO LAKTA` | 6 vs 2 |

Ditemukan **46 grup kembaran** semacam ini. Sebagian nilai juga berspasi di ujung
(`RAFINS `, `PT IKAPHARMINDO PUTRAMAS TBK `, `  SAMICHSAN MULIA`).

**Aturan:** cacah perusahaan unik dan peringkat pendaftar dari kolom ini **terlalu tinggi** tanpa
normalisasi `btrim(regexp_replace(pendaftar, '\s+', ' ', 'g'))`. Sebutkan bahwa varian penulisan
digabungkan bila pertanyaannya menyangkut peringkat atau jumlah perusahaan.

## K4. Spasi ekor pada nama balai — filter kesamaan persis gagal

`BALAI POM DI DUMAI ` tersimpan dengan spasi di belakang di `mv_penandaan`, `mv_penandaan_agg`, dan
`mv_penandaan_log`. Karena konsisten, join tetap jalan; yang gagal adalah filter literal
`nama_balai = 'BALAI POM DI DUMAI'` — nol baris, tanpa pesan kesalahan.

**Aturan:** filter kesamaan persis pada nama balai harus lewat `trim()`.

## K5. Nama pelaku di log — kembaran karena gelar

`mv_penandaan_log.fullname` memuat orang yang sama dengan penulisan gelar berbeda
(`Eka Akhriana, S.Farm, Apt` 750 versus `Eka Akhriana, S.Farm., Apt.` 241), dan ada nilai berspasi
ekor (`Redo Rizaldi ` 92 baris). Peringkat berbasis nama orang dari kolom ini terpecah.

## K6. Anomali tanggal

| Kolom | Temuan |
|---|---|
| `mv_penandaan_timeline.tgl_start` | `1970-01-01` sebanyak **6.353 baris** (epoch, pengganti kosong); satu baris 1907; satu baris 2032 |
| `mv_penandaan_timeline.tgl_end` | `1970-01-01` sebanyak **6.353 baris**; satu baris 2032 |
| `mv_penandaan.ed_nie` | tahun terpotong `1026`-`1028`; tahun kacau `1747`, `2102`-`2127`, `2225`, `2227`, `2929`; serta 1924-1929 |

Epoch `1970-01-01` di kolom timeline adalah temuan terpenting di sini: **6.353 baris** bukan jumlah
yang bisa diabaikan, dan karena ia tanggal yang sah secara tipe data, ia **ikut terhitung** dalam
`MIN()`, dalam selisih durasi, dan dalam `GROUP BY` tahun. Durasi yang dihitung dari baris itu akan
bernilai puluhan ribu hari.

Catatan soal `ed_nie`: tahun 2027-2056 **wajar** karena ini tanggal berakhirnya nomor izin edar,
yang memang jatuh di masa depan. Yang anomali adalah tahun terpotong dan tahun mustahil di atas.

**Aturan:** setiap perhitungan durasi dari kolom timeline wajib membuang `1970-01-01` lebih dulu,
dan pertanyaan "paling awal / paling lama" wajib membatasi tahun ke rentang wajar. Rentang
operasional sesungguhnya dimulai **2020**.
