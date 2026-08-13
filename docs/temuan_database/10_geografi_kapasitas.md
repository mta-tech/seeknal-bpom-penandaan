# 10 — Geografi & Kapasitas

> Balai besar vs loka, coverage kabupaten, disparitas TMK.

---

## Peta Yurisdiksi (coverage_balai)

| Metrik | Angka |
|---|---|
| Total balai (coverage) | 88 |
| Balai aktif (di fact) | 83 |
| Total kabupaten | 514 |
| Total baris coverage | 668 (many-to-many) |
| Avg kabupaten/balai | ~8 |
| Relasi id_balai ↔ nama_balai | **1:1 bersih** |

### Balai dengan cakupan kabupaten terluas

| Balai | n_kabupaten |
|---|---|
| BALAI BESAR POM DI SURABAYA | **38** |
| BALAI BESAR POM DI SEMARANG | 35 |
| BALAI BESAR POM DI BANDUNG | 27 |
| BALAI BESAR POM DI MEDAN | 26 |
| BALAI BESAR POM DI JAYAPURA | 22 |
| BALAI BESAR POM DI PADANG | 19 |
| BALAI BESAR POM DI MAKASSAR | 17 |
| BALAI BESAR POM DI KUPANG | 16 |

> Balai Besar di pulau Jawa + Jayapura cakupan terluas. Beban yurisdiksi tidak merata.

---

## Beban Volume per Balai (top 8, 2026)

| Balai | n sampel 2026 |
|---|---|
| BALAI BESAR POM DI JAKARTA | 1.720 |
| BALAI BESAR POM DI YOGYAKARTA | 1.674 |
| BALAI BESAR POM DI SURABAYA | 1.583 |
| BALAI BESAR POM DI DENPASAR | 1.487 |
| BALAI BESAR POM DI BANDUNG | 1.374 |
| BALAI BESAR POM DI SEMARANG | 1.351 |
| BALAI BESAR POM DI PEKANBARU | 1.350 |
| BALAI BESAR POM DI PADANG | 1.147 |

> Konsentrasi di **Jawa + Denpasar**. 5 dari 8 balai top di Jawa.

---

## TMK Rate — Seragam di Semua Tier (menenangkan)

### Balai Besar (top 8 by volume all-time)

| Balai | n | TMK% | MK% |
|---|---|---|---|
| BALAI BESAR POM DI SURABAYA | 11.391 | 27,2 | 51,3 |
| BALAI BESAR POM DI BANDUNG | 11.348 | 24,6 | 50,6 |
| BALAI BESAR POM DI JAKARTA | 11.277 | 28,6 | 40,5 |
| BALAI BESAR POM DI YOGYAKARTA | 10.248 | 28,5 | 52,4 |
| BALAI BESAR POM DI DENPASAR | 9.963 | 23,3 | 45,3 |
| BALAI BESAR POM DI SEMARANG | 9.937 | 27,7 | 54,3 |
| BALAI BESAR POM DI PEKANBARU | 8.221 | **30,9** | 45,3 |
| BALAI BESAR POM DI PADANG | 7.974 | 26,3 | **54,9** |

### Loka POM (balai kecil)

| Balai | n | TMK% |
|---|---|---|
| LOKA POM DI KABUPATEN ACEH SELATAN | 2.116 | 23,3 |
| LOKA POM DI KABUPATEN KOTAWARINGIN BARAT | 2.075 | 24,5 |
| LOKA POM DI KABUPATEN REJANG LEBONG | 2.059 | 23,6 |
| LOKA POM DI KABUPATEN ACEH TENGAH | 1.973 | 25,5 |
| LOKA POM DI KOTA TANJUNG PINANG | 1.764 | **35,0** |
| LOKA POM DI KABUPATEN KEPULAUAN SANGIHE | 1.636 | 24,6 |

> **TMK% seragam di semua tier** (23-35%). Tidak ada disparitas ekstrem antar balai besar/kecil atau geografis di RATA-RATA. Pekanbaru tertinggi (30,9%), Tanjung Pinang 35% (tapi sampel kecil).
>
> ⚠️ TAPI ini berbeda dari temuan [07_temuan_kritis](07_temuan_kritis.md) §temuan-3: **rata-rata TMK seragam, tapi produk-INDIVIDU tidak konsisten lintas balai**. Standar rata-rata nasional sehat, kalibrasi produk-spesifik rusak.

---

## Ketimpangan Kapasitas — Durasi Lambat di Balai Kecil/Terpencil

| Balai | avg kabalai_direktur |
|---|---|
| LOKA POM KOTAWARINGIN BARAT | 36,5 hari |
| LOKA POM KEPULAUAN SANGIHE | 32,0 |
| BALAI BESAR PALANGKARAYA | 31,5 |
| BALAI POM KEDIRI | 31,4 |
| BALAI BESAR KUPANG | 31,2 |
| LOKA POM TANJUNG PINANG | 30,9 |
| BALAI POM SOFIFI | 30,0 |
| BALAI BESAR GORONTALO | 29,8 |

> **Balai Indonesia Timur/luar Jawa paling lambat.** Ketimpangan kapasitas geografis — pengawasan tidak merata secara nasional. Lihat [09_waktu_durasi](09_waktu_durasi.md).

---

## Coverage Balai × Target — 29 Balai "Buta" KPI

| Item | Angka |
|---|---|
| Balai aktif di fact | 83 |
| Balai di target_balai | 76 |
| Match (intersect) | 54 |
| **Balai tanpa target 2024** | **29** |

> 29 balai aktif **tidak punya target 2024** → tak terukur KPI-nya. Lihat [14_kpi_target_2024](14_kpi_target_2024.md).

---

## Koneksi ke Pertanyaan User — Perspektif UPT/Balai

User memakai istilah **"UPT"** (= `nama_balai`) dalam ~40 pertanyaan. Pola yang paling sering:

| Pola user | SQL | n pertanyaan |
|---|---|---|
| "Balai/UPT mana yang tertinggi/terendah" | `GROUP BY nama_balai ORDER BY count DESC/ASC LIMIT n` | 6–10 |
| "rekapitulasi jumlah laporan masing-masing UPT" | `GROUP BY nama_balai` (+ filter `tanggal_kirim_pusat`) | 14 |
| "UPT paling banyak melaporkan kesimpulan MK/TMK" | `WHERE kesimpulan_* = 'MK'/'TMK' GROUP BY nama_balai` | ~5 |
| "gap antara pusat dan UPT" | `kesimpulan_balai <> kesimpulan_pusat` + `GROUP BY nama_balai` | 15 |

> **UPT = balai** — 83 nilai aktif di `nama_balai`. Jangan salah terjemah (lihat [19_mapping](19_mapping_konsep_ke_schema.md)).
> SQL siap pakai: [21_sql_pairs_penandaan](21_sql_pairs_penandaan.md) SQL-03/SQL-04.

> ⚠️ **Gap geografi user:** "kabupaten/kota/provinsi produsen" (2–4 pertanyaan) TIDAK tersedia — DB tidak punya kolom
> lokasi/alamat produsen. Jawab per `produsen` saja. Lihat [20_gap_schema_user](20_gap_schema_user.md) G2.

---

## Implikasi Geografis

1. **Volume terkonsentrasi Jawa+Denpasar** — 5 dari 8 top balai di Jawa
2. **TMK rata-rata seragam** — standar nasional bekerja di tingkat agregat
3. **TAPI kalibrasi produk-spesifik tidak seragam** — 41,6% produk identik beda keputusan lintas balai
4. **Kapasitas timpang** — balai Indonesia Timur 30-37 hari vs balai Jawa lebih cepat
5. **29 balai buta KPI** — perlu backfill target atau pengakuan eksplisit

---

Lanjut ke [11_data_quality_anomali.md](11_data_quality_anomali.md).
