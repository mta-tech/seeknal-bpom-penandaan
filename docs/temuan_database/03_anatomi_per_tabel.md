# 03 — Anatomi Per Tabel

> Detail kolom, tipe, distinct, null, peran untuk semua 10 tabel (6 public + 4 dimension).

---

## 3.1 `mv_penandaan` — FACT CORE (292.598 baris)

1 baris = **1 sampel penandaan** + keputusan final. PK kandidat: `id` (unique).

| Kolom | Tipe | Distinct | Null/Empty | Catatan |
|---|---|---|---|---|
| `id` | bigint | 292.598 | 0 | **Unique — PK kandidat** |
| `komoditi` | text | 8 | 0 | Kategorikal (UPPER). Lihat [04_komoditi](04_komoditi_governing_dimension.md) |
| `nomor_surat` | text | ~22.443 | sentinel | Batch pengantar. Prefix `PW.dd.bb...` |
| `nomorsampel` | text | ~260.112 | 31.985 | ID fisik sampel. Prefix `yy.bbb.` = tahun+balai |
| `tgl_start` | date | 1.281 | 0 | Range 2023-01-01 → 2026-08-11 |
| `tgl_end` | date | 1.286 | 0 | **93,5% same-day** (median durasi 0) |
| `nama_produk` | text | ~84.693 | 200 | Nama pada label. Free text |
| `ed_nie` | date | 3.498 | 29.270 | Expiry NIE. **31 outlier** (1026-2929) |
| `pendaftar` | text | 4.314 | 116.232 | **40% kosong** — deterministik per komoditi |
| `produsen` | text | 20.480 | 80.202 | **27% kosong** — deterministik per komoditi |
| `nama_balai` | text | 83 | 1 | 100% match ke coverage_balai |
| `kesimpulan_penilaian_balai` | text | 6 | 85.113 | Penilaian daerah. **Casing dup**: "TMK Minor" vs "TMK MINOR" |
| `kesimpulan_penilaian_pusat` | text | 5 | 0 | Penilaian FINAL pusat. 0 null |
| `catatan` | text | 27.087 | 179.841 | 62% kosong. Kode regulasi: MKL.02, TME.*, NIK.01 |
| `sync` | timestamp | 1 | 0 | Seragam 2026-08-11 22:53:20 |

### Empat Lapis Makna
- **Lapis A — Identitas:** `id`, `nomor_surat`, `nomorsampel`, `nama_produk`
- **Lapis B — Regulasi:** `ed_nie` (expiry NIE), `komoditi` (→ direktorat)
- **Lapis C — Aktor:** `pendaftar`, `produsen`, `nama_balai`
- **Lapis D — Keputusan:** `kesimpulan_penilaian_balai`, `kesimpulan_penilaian_pusat`, `catatan`

---

## 3.2 `mv_penandaan_log` — AUDIT (3.533.299 baris)

1 baris = **1 event workflow**. 500.717 distinct `id_penandaan` (208.119 orphan).

| Kolom | Tipe | Catatan |
|---|---|---|
| `id_penandaan` | bigint | FK implicit ke `mv_penandaan.id`. 208.119 orphan |
| `trx_steps` | text | **Edge/aksi** — "draft", "spv_1", "pusat", "direktur", "selesai" |
| `status_code` | bigint | **Node/state kanonik** — 19 nilai (0-999). **GUNAKAN INI** untuk analisis state |
| `status_label` | text | Aligned dengan status_code. 39.964 empty |
| `fullname` | text | Petugas. 1.803 unik. Tiap user → 1 balai |
| `nama_balai` | text | 91 entitas (83 balai + 5 direktorat pusat + 3 lain) |
| `catatan` | text | 45% non-empty. Workflow notes + kode regulasi |
| `tanggal_proses` | timestamp | 8,4% null. Range 2020-03 → 2026-08 |
| `sync` | timestamp | ETL timestamp |

> ⚠️ **`trx_steps` vs `status_code`:** Dua kolom berbeda hal — `trx_steps` = edge (aksi/transisi), `status_code` = node (state tujuan). **Jangan campur**. Lihat [05_workflow_state_machine](05_workflow_state_machine.md).

---

## 3.3 `mv_penandaan_timeline` — LIFECYCLE (500.717 baris)

1 baris = **1 sampel lifecycle + durasi per tahap**. PK kandidat: `id_penandaan`.

| Kolom | Tipe | Null % | Catatan |
|---|---|---|---|
| `id_penandaan` | bigint | 0 | Unique — PK kandidat |
| `tgl_start` / `tgl_end` | date | 0,003% | Cocok dengan fact |
| `tanggal_kirim_kabalai` | date | 4,4% | Milestone |
| `tanggal_kirim_direktur` | date | 29,5% | Banyak null = belum mencapai tahap ini |
| `tanggal_kirim_pusat` | date | 4,7% | |
| `status` | bigint | 0 | Sama dengan log `status_code` |
| `mulai_kabalai` | integer | 4,4% | Hari: start → kirim kabalai. Median 6, max 1475 |
| `kabalai_direktur` | integer | 29,5% | **BOTTLENECK.** Median 21, max 996 |
| `direktur_pusat` | integer | 29,5% | Median 0 (instan). Max 209 |

> ⚠️ **Kolom durasi ≠ selisih tanggal!** Hanya 4,5% cocok dengan `tanggal_kirim_kabalai - tgl_start`. Kolom durasi dihitung logika lain (kemungkinan business days). **Jangan verifikasi dengan selisih tanggal.** Lihat [09_waktu_durasi](09_waktu_durasi.md).

---

## 3.4 `mv_penandaan_agg` — ROLLUP (94.683 baris)

Pre-aggregated per `(periode_type, tanggal_periode, komoditi, nama_balai, kesimpulan_balai, kesimpulan_pusat)`.

| Kolom | Tipe | Catatan |
|---|---|---|
| `periode_type` | text | `day` (63.947) atau `month` (30.736) |
| `tanggal_periode` | date | |
| `komoditi` | text | 8 nilai |
| `nama_balai` | text | 83 balai |
| `kesimpulan_penilaian_balai` | text | 6 nilai, 26% null |
| `kesimpulan_penilaian_pusat` | text | 5 nilai |
| `jumlah_penandaan` | bigint | NOT NULL. Σ = 292.598 = fact ✅ |
| `jumlah_surat_unik` | bigint | Distinct nomor_surat |
| `jumlah_sampel_unik` | bigint | Distinct nomorsampel |
| `jumlah_produk_unik` | bigint | Distinct nama_produk |
| `avg_durasi_hari` | double | Avg tgl_end − tgl_start |
| `min_durasi_hari` / `max_durasi_hari` | integer | |
| `last_updated` | timestamp | ETL timestamp |

> agg = satu-satunya tabel dengan integritas terbukti sempurna vs fact. Tapi hanya era 2023+.

---

## 3.5 `coverage_balai` — DIMENSI GEOGRAFI (668 baris)

| Kolom | Tipe | Catatan |
|---|---|---|
| `id_balai` | bigint | 88 distinct |
| `nama_balai` | text | ↔ id_balai **1:1** |
| `id_kabupaten` | integer | 514 distinct kabupaten |
| `kabupaten_kota` | text | Nama kabupaten |
| `sync` | timestamp | |

**Grain:** 1 baris = 1 balai × 1 kabupaten (many-to-many). 88 balai, avg 8 kabupaten/balai. Terluas: Surabaya 38 kabupaten. Lihat [10_geografi_kapasitas](10_geografi_kapasitas.md).

---

## 3.6 `target_balai` — DIMENSI KPI (532 baris)

| Kolom | Tipe | Catatan |
|---|---|---|
| `id` | bigint | |
| `nama_balai` | text | 76 distinct |
| `komoditi` | text | 7 komoditi (Title case, bukan UPPER). **KEMASAN PANGAN absent** |
| `tahun` | bigint | **2024 SAJA** |
| `target_penandaan` | bigint | Σ per komoditi |
| `target_pengawasan` | bigint | |
| `target_pengujian` | bigint | |
| `target_pengujian_pangan` | bigint | |
| `target_pengujian_pangan_fortifikasi` | bigint | |
| `target_sarana_distribusi` | bigint | |
| `target_sarana_produksi` | bigint | |
| `sync` | timestamp | |

> ⚠️ Casing mismatch: target "Kosmetika" vs fact "KOSMETIKA". Join wajib `lower()`. Lihat [14_kpi_target_2024](14_kpi_target_2024.md).

---

## 3.7–3.10 Schema `dimension` (STALE — ringkasan)

| Tabel | Baris | Kolom di-drop vs public |
|---|---|---|
| `dimension.mv_penandaan` | 218.472 | id, tgl_*, ed_nie, kesimpulan_balai (10 kolom tersisa) |
| `dimension.mv_penandaan_log` | 33.137 | id_penandaan, status_code, tanggal_proses |
| `dimension.coverage_balai` | 513 | id_balai, id_kabupaten (2 kolom) |
| `dimension.target_balai` | 76 | semua target_* numerik |

> **`dimension` = proyeksi BI yang di-flatten.** Tanpa id → tak bisa join balik. Selisih baris vs public = snapshot tertinggal. **Bukan sumber kebenaran.** Lihat [11_data_quality_anomali](11_data_quality_anomali.md) §dimension-stale.

---

## Tipe Kolom — Tidak Perlu Cast

Berbeda dari sistem ERBA (semua TEXT), di database `penandaan` semua kolom `mv_penandaan` sudah **native PostgreSQL types** (bigint, date, text, timestamp). Tidak perlu `::timestamp` atau `::numeric` cast.

---

Lanjut ke [04_komoditi_governing_dimension.md](04_komoditi_governing_dimension.md).

---

## Katalog Nilai Penuh (live 2026-08-13, 292.758 baris)

`GROUP BY` penuh atas seluruh baris. Beberapa angka berbeda dari profil awal karena ETL terus
menambah baris; yang penting adalah **struktur nilainya**.

### `kesimpulan_penilaian_balai` — 6 nilai, dan dua di antaranya nilai yang sama 🔴

| Nilai | Baris | % |
|---|--:|--:|
| `MK` | 177.440 | 60,6 |
| **`''` (kosong)** | **85.257** | **29,1** |
| `TMK` | 14.140 | 4,8 |
| `TMK MINOR` | 7.424 | 2,5 |
| `TMK MAYOR` | 6.607 | 2,3 |
| **`TMK Minor`** | **1.890** | **0,6** |

⚠️ **`TMK Minor` (huruf kecil) adalah entri terpisah dari `TMK MINOR`.** Filter
`kesimpulan_penilaian_balai = 'TMK MINOR'` **melewatkan 1.890 baris** (20,3% dari seluruh TMK
Minor). Pakai `upper(kesimpulan_penilaian_balai)` atau `ILIKE 'TMK MINOR'`.

Kolom `kesimpulan_penilaian_pusat` **tidak** punya masalah ini — 5 nilai bersih:
`MK` 151.777 · `TMK` 76.547 · `VP` 59.831 · `TMK MAYOR` 2.367 · `TMK MINOR` 2.236.

**Asimetri taksonomi:** balai punya 5 nilai vonis + kosong; pusat punya 5 nilai termasuk `VP`
yang tidak pernah dipakai balai. Jadi "TMK family" berbeda di kedua sisi:
`balai ∈ {TMK, TMK MAYOR, TMK MINOR, TMK Minor}` · `pusat ∈ {TMK, TMK MAYOR, TMK MINOR}`.

### Kosongnya `_balai` = OBAT + ROKOK, tepat

85.257 baris kosong ≈ OBAT (53.175) + ROKOK (32.081) = 85.256, ditambah 1 baris PRODUK PANGAN.
Jadi kekosongan itu **deterministik per komoditi**, bukan tersebar. Konsekuensi analitiknya:
`04_komoditi_governing_dimension.md` §Populasi yang Sah Dibandingkan.

### `komoditi` — 8 nilai (satu-satunya domain dengan `KEMASAN PANGAN` terpakai)

`KOSMETIKA` 78.550 · `PRODUK PANGAN` 75.629 · `OBAT` 53.175 · `OBAT TRADISIONAL (OT)` 39.189 ·
`ROKOK` 32.081 · `SUPLEMEN KESEHATAN` 10.757 · `OBAT KUASI` 2.688 · `KEMASAN PANGAN` 689.

Domain `pengawasan` punya 7 (tanpa `KEMASAN PANGAN`); `pemeriksaan` punya 13 dengan ejaan berbeda
(`KOSMETIK`, `OBAT TRADISIONAL`). Jangan menyalin daftar komoditi antar domain.

### `mv_penandaan_log` — label status **dipinjam dari domain pengujian** 🟡

| `trx_steps` | Baris | `status_label` | Baris |
|---|--:|---|--:|
| `draft` | 576.586 | `Operator - Draft Sampling` | 576.135 |
| `pusat` | 536.586 | `MT - Pembuatan SPK` | 536.084 |
| `spv_1` | 516.563 | `Supervisor - Verifikasi` | 518.409 |
| `kepala_balai` | 489.702 | `TPS - Penerimaan SPU` | 490.508 |
| `spv_1_pusat` | 359.921 | `Deputi MT - Pembuatan SPK` | 359.974 |
| `direktur` | 354.181 | `Penguji - Entri Hasil Pengujian` | 354.065 |
| `selesai` | 350.804 | `Sampel Rujukan Selesai` | 350.735 |
| `spv_2_pusat` | 251.602 | `Penyelia - Pembuatan SPP` | 251.411 |
| `spv_2` | 56.843 | `Supervisor 2 - Verifikasi` | 56.841 |
| 6 langkah `ditolak_*` | 42.174 | `<SQL NULL>` | 40.020 |
| `receive_tps` · `spv1` · NULL | 10 | `Operator - Perbaikan Sampel` · `Kepala Balai` | 793 |

**`status_label` di domain penandaan menyalin kamus label domain pengujian.** Langkah `direktur`
berlabel *"Penguji - Entri Hasil Pengujian"*, `kepala_balai` berlabel *"TPS - Penerimaan SPU"*,
`selesai` berlabel *"Sampel Rujukan Selesai"* — semuanya istilah alur **sampling/pengujian**, bukan
penandaan. Labelnya salah domain.

**Aturan:** untuk menceritakan alur penandaan, pakai **`trx_steps`**, jangan `status_label`.
Yang terakhir akan menghasilkan kalimat seperti "laporan penandaan masuk tahap Penguji - Entri
Hasil Pengujian", yang tidak masuk akal bagi pengguna.

Perhatikan juga **`spv1`** (1 baris, tanpa garis bawah) terpisah dari `spv_1` (516.563) — salah
ketik yang menjadi nilai tersendiri.

### Filter yang tersedia per tabel

| Tabel | Dimensi filter | Ukuran |
|---|---|---|
| `mv_penandaan` | `komoditi` (8) · `nama_balai` · `kesimpulan_penilaian_balai` (6) · `kesimpulan_penilaian_pusat` (5) · `tgl_start`/`tgl_end` · `ed_nie` · `pendaftar` · `produsen` · `catatan` | 292.758 baris, `id` unik |
| `mv_penandaan_log` | `trx_steps` (19) · `status_code` (20) · `status_label` (12) · `fullname` | 3.533.765 baris · 500.820 id |
| `mv_penandaan_timeline` | `status` (18) · 3 tanggal milestone · 3 kolom durasi | 500.820 baris |
| `mv_penandaan_agg` | `komoditi` · `nama_balai` · 2 kolom verdict · `periode_type` (2) | 94.737 baris |
| `coverage_balai` | `nama_balai` (88) · `kabupaten_kota` (514) | 668 baris |
| `target_balai` | `nama_balai` (76) · `komoditi` (7, **Title Case**) · `tahun` (**2024 saja**) | 532 baris |

⚠️ **`provinsi` dan `kabupaten` TIDAK ADA** di `mv_penandaan` (15 kolom). Kolom keduanya masih
tercatat di `table_descriptions` KAI generasi `_all_v2` — sudah dihapus di live. Geografi hanya
lewat `nama_balai` → `coverage_balai` (wilayah kerja balai, bukan alamat produsen).

⚠️ **Tidak ada kolom jenis/nama kemasan.** Lihat `20_gap_schema_user.md` G1.
