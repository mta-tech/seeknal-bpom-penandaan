# Temuan Database — seeknal-bpom-penandaan

Dokumentasi komprehensif hasil **deep-dive analisis database `penandaan`** (e-Penandaan BPOM).
Semua angka diturunkan dari **query SQL langsung** ke database (read-only), snapshot `sync = 2026-08-11 22:53:20`.
Total **292.598 baris** di tabel fact `mv_penandaan`.

> Dokumen ini = laporan temuan audit data. Bukan menggantikan `context/*.md` (cheat-sheet operasional),
> melainkan **pemahaman struktural mendalam** yang memvalidasi, mengoreksi, dan memperdalam context.

---

## Status Verifikasi

| Item | Nilai |
|---|---|
| Database | `penandaan` (PostgreSQL, schema `public` + `dimension`) |
| Snapshot sync | 2026-08-11 22:53:20 |
| Total baris fact | 292.598 (`mv_penandaan`) |
| Total baris log | 3.533.299 (`mv_penandaan_log`) |
| Total baris timeline | 500.717 (`mv_penandaan_timeline`) |
| Metode | Query SQL read-only langsung via psql |
| Tanggal analisis | 2026-08-13 |

---

## Navigasi Dokumen

### Ringkasan & Fondasi
| File | Isi |
|---|---|
| [00_ringkasan_eksekutif.md](00_ringkasan_eksekutif.md) | 3 temuan kritis, ranking prioritas audit, peta kausal utuh |
| [01_identitas_domain.md](01_identitas_domain.md) | Apa sistem ini (e-Penandaan BPOM), bukti petunjuk, model mental |
| [02_arsitektur_database.md](02_arsitektur_database.md) | 6 tabel public + 4 dimension, ERD, grain, peran, relasi implicit |

### Anatomi & Struktur
| File | Isi |
|---|---|
| [03_anatomi_per_tabel.md](03_anatomi_per_tabel.md) | Detail tiap tabel: kolom, tipe, distinct, null, peran (semua 10 tabel) |
| [04_komoditi_governing_dimension.md](04_komoditi_governing_dimension.md) | 8 komoditi & kepribadian, null deterministik, form per komoditi |
| [05_workflow_state_machine.md](05_workflow_state_machine.md) | Status codes, trx_steps (edge vs node), piramida user, PANGAN dead-end |
| [06_penilaian_keputusan.md](06_penilaian_keputusan.md) | Taksonomi kesimpulan, VP multi-makna, matriks transisi balai→pusat |

### Temuan Material
| File | Isi |
|---|---|
| [07_temuan_kritis.md](07_temuan_kritis.md) | 3 temuan material: (1) PANGAN 0% completion, (2) 3 orang = 72% approval, (3) 41,6% produk beda keputusan |
| [08_proses_beban_kerja.md](08_proses_beban_kerja.md) | Piramida organisasi, Gama knowledge hub, rework 9,7%, reject loop |
| [09_waktu_durasi.md](09_waktu_durasi.md) | Bottleneck, durasi ≠ selisih tanggal, direktur_pusat, durasi per balai |
| [10_geografi_kapasitas.md](10_geografi_kapasitas.md) | Balai besar vs loka, coverage kabupaten, disparitas TMK |

### Kualitas Data & Kode
| File | Isi |
|---|---|
| [11_data_quality_anomali.md](11_data_quality_anomali.md) | Orphan 208k, tanggal 1970, kode ambigu, dimension stale, ed_nie outlier |
| [12_kode_berstruktur.md](12_kode_berstruktur.md) | Decode nomor_surat (PW.01=OBAT dll), kode balai di nomorsampel |
| [13_konsistensi_integritas.md](13_konsistensi_integritas.md) | Drift timeline-log, agg validation, metodologi join target |

### KPI & Gap
| File | Isi |
|---|---|
| [14_kpi_target_2024.md](14_kpi_target_2024.md) | Achievement 101-116%, metodologi benar, koreksi hipotesis "KPI buta" |
| [15_gap_belum_ditemukan.md](15_gap_belum_ditemukan.md) | Open questions, aspek belum didetailkan, hipotesis tertunda |
| [16_informasi_eksternal_dibutuhkan.md](16_informasi_eksternal_dibutuhkan.md) | Kamus kode regulasi, SOP, SLA, target multi-tahun, master balai |
| [17_rencana_investigasi_lanjut.md](17_rencana_investigasi_lanjut.md) | Daftar query verifikasi (K1-K11) dengan SQL siap eksekusi |

### Perspektif User (validasi pertanyaan KAI)
| File | Isi |
|---|---|
| [18_pola_pertanyaan_user.md](18_pola_pertanyaan_user.md) | Taksonomi 10+ pola pertanyaan user asli (101 unik), frekuensi, contoh verbatim |
| [19_mapping_konsep_ke_schema.md](19_mapping_konsep_ke_schema.md) | Pemetaan istilah user (UPT, kesimpulan akhir, tanggal direktur) → kolom DB |
| [20_gap_schema_user.md](20_gap_schema_user.md) | Gap schema (jenis kemasan, provinsi, media iklan) — anti-halusinasi |
| [21_sql_pairs_penandaan.md](21_sql_pairs_penandaan.md) | 10 SQL pairs baru yang ditulis & divalidasi langsung ke DB |
| [22_prioritas_use_case.md](22_prioritas_use_case.md) | Ranking P0/P1/P2/P3 berdasar frekuensi pertanyaan user |

---

## Cara Membaca

- **Untuk eksekutif/auditor:** mulai dari [00_ringkasan_eksekutif.md](00_ringkasan_eksekutif.md) → ranking prioritas.
- **Untuk data engineer:** [02_arsitektur](02_arsitektur_database.md) → [03_anatomi](03_anatomi_per_tabel.md) → [11_data_quality](11_data_quality_anomali.md).
- **Untuk analis domain:** [04_komoditi](04_komoditi_governing_dimension.md) → [05_workflow](05_workflow_state_machine.md) → [06_penilaian](06_penilaian_keputusan.md).
- **Untuk investigator lanjut:** [15_gap](15_gap_belum_ditemukan.md) → [17_rencana_investigasi](17_rencana_investigasi_lanjut.md).
- **Untuk menjawab pertanyaan user (agent):** [18_pola_pertanyaan](18_pola_pertanyaan_user.md) → [19_mapping](19_mapping_konsep_ke_schema.md) → [21_sql_pairs](21_sql_pairs_penandaan.md) → [20_gap_schema](20_gap_schema_user.md).

## Konvensi

- Setiap angka disertai sumber (tabel.kolom + query).
- `✅` = terverifikasi, `⚠️` = peringatan, `❌` = kontradiksi/bug, `🟡` = hipotesis.
- "Koreksi" menandai temuan yang mengganti asumsi context awal.
- Bahasa: Indonesia (istilah teknis EN).
