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
