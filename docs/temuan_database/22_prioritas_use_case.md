# 22 — Prioritas Use Case (Berdasarkan Frekuensi Pertanyaan User)

> Ranking use case pengembangan jawaban/SQL pairs — ditentukan oleh **frekuensi & kemudahan jawab** dari pertanyaan
> user asli (lihat [18](18_pola_pertanyaan_user.md)). Memandu mana yang harus dikerjakan dulu oleh agent/data engineer.

---

## Matriks Prioritas

| Prioritas | Use case | n pertanyaan | Bisa dijawab DB? | SQL |
|---|---|---|---|---|
| **P0** | Gap pusat vs UPT | 15 | ✅ | SQL-02 |
| **P0** | Rekap per UPT | 14 | ✅ | SQL-03 |
| **P0** | Agregasi total / filter waktu | 17 | ✅ | SQL-01 |
| **P1** | MK/TMK per kategori | 27 | ✅ | SQL-05, SQL-09 |
| **P1** | Rekap tanggal proses direktur | 11 | ✅ | SQL-07 |
| **P1** | Ranking UPT | 6–10 | ✅ | SQL-04 |
| **P2** | Timeline pemenuhan | 5 | ✅ | SQL-08 |
| **P2** | Investigasi catatan TMK | 5 | ✅ | SQL-06 |
| **P2** | Triplet OT/Supkes/Kuasi | 6 | ✅ | SQL-10 |
| **P3** | Ranking produsen | 4 | ✅ (proxy produsen) | SQL varian |
| **P3** | Pangan / label pangan | 3 | ✅ (komoditi) | SQL varian |
| **P3** | Raw export | 3 | ✅ | SELECT * LIMIT |
| **⛔** | Iklan/klaim/media | 7 | ❌ domain pengawasan | — |
| **⛔** | Jenis kemasan | 3–5 | ❌ gap schema | — |
| **⛔** | Kabupaten/provinsi produsen | 2–4 | ❌ gap schema | — |

---

## P0 — Wajib Dilengkapi Dulu (Dampak Terbesar)

### P0.1 Gap Pusat vs UPT (15 pertanyaan)
- **Inti bisnis:** konsistensi keputusan balai vs pusat.
- **Temuan pendukung:** [07_temuan_kritis](07_temuan_kritis.md) §temuan-3 (41,6% produk identik beda keputusan;
  99,3% lintas balai) = bukti kuantitatif untuk jawaban gap user.
- **SQL:** SQL-02 (+ varian persamaan `=`).

### P0.2 Rekap per UPT (14 pertanyaan)
- **Inti bisnis:** beban & output tiap balai.
- **Temuan pendukung:** [10_geografi_kapasitas](10_geografi_kapasitas.md) — 5 dari 8 top balai di Jawa.
- **SQL:** SQL-03.

### P0.3 Agregasi Total / Filter Waktu (17 pertanyaan)
- **Inti bisnis:** jumlah penandaan per tahun/bulan.
- **SQL:** SQL-01 (+ filter komoditi, + filter bulan).

---

## P1 — Tinggi

### P1.1 MK/TMK per Kategori (27 — terbanyak tapi sederhana)
- **SQL:** SQL-05 (rekap 2 arah), SQL-09 (TMK saja).
- Perhatikan "kesimpulan UPT" vs "kesimpulan akhir" — disambiguasi di [19](19_mapping_konsep_ke_schema.md).

### P1.2 Rekap Tanggal Proses Direktur (11)
- **SQL:** SQL-07. Kolom di timeline — wajib join.

### P1.3 Ranking UPT (6–10)
- **SQL:** SQL-04 (DESC/ASC, LIMIT).

---

## P2 — Sedang

- **Timeline pemenuhan** (SQL-08): SLA "maksimal tanggal 15 bulan berikutnya".
- **Investigasi catatan TMK** (SQL-06): terkait "catatan ceremonial" [06](06_penilaian_keputusan.md).
- **Triplet OT/Supkes/Kuasi** (SQL-10): group filter khas.

---

## P3 — Lengkapi Nanti

- **Ranking produsen:** proxy `produsen`. Perlu konfirmasi makna "sarana produksi".
- **Pangan/label pangan:** filter `komoditi='PRODUK PANGAN'`. ⚠️ PANGAN tidak pernah tuntas di pusat — lihat
  [07_temuan_kritis](07_temuan_kritis.md).
- **Raw export:** `SELECT * ... LIMIT n`.

---

## ⛔ Tidak Dikerjakan di DB Ini

| Use case | Alasan | Arahkan ke |
|---|---|---|
| Iklan/media/klaim/golongan obat | Domain `pengawasan`, bukan penandaan | DB `pengawasan` |
| Jenis kemasan | Kolom tidak ada | Laporkan gap (G1) |
| Kabupaten/provinsi produsen | Kolom tidak ada | Data eksternal master produsen (G2) |
| e-labeling | Tidak ada flag | Data eksternal |

---

## Rekomendasi Eksekusi (urutan kerja agent)

1. **Daftarkan SQL-01–10** sebagai SQL pairs resmi (sudah tervalidasi di [21](21_sql_pairs_penandaan.md)).
2. **Tambahkan mapping UPT/balai & 3-level kesimpulan** ke context — pencegahan salah terjemah (lihat
   [19](19_mapping_konsep_ke_schema.md)).
3. **Siapkan jawaban "gap" untuk P0.1** yang memanfaatkan temuan disagreement 41,6% — nilai tambah dari dokumentasi temuan.
4. **Siapkan respon anti-halusinasi** untuk gap schema (G1–G7) — lihat [20](20_gap_schema_user.md).

---

## Metrik Keberhasilan

- ≥80% pertanyaan penandaan user bisa dijawab SQL tervalidasi.
- 0 halusinasi kolom (jenis kemasan, provinsi, media).
- Kesalahan terjemah "kesimpulan UPT vs akhir" = 0.

---

## Indeks Silang

| Topik | File |
|---|---|
| Taksonomi pertanyaan | [18_pola_pertanyaan_user](18_pola_pertanyaan_user.md) |
| Mapping istilah → kolom | [19_mapping_konsep_ke_schema](19_mapping_konsep_ke_schema.md) |
| Gap schema | [20_gap_schema_user](20_gap_schema_user.md) |
| SQL tervalidasi | [21_sql_pairs_penandaan](21_sql_pairs_penandaan.md) |
| Temuan kritis pendukung | [07_temuan_kritis](07_temuan_kritis.md) |

---

# Epilog — 5 File Baru (18–22)

5 file ini melengkapi dokumentasi temuan (00–17) dengan **perspektif user**: taksonomi pertanyaan, pemetaan istilah,
gap schema, SQL pairs tervalidasi, dan prioritas. Baca urutan: `00 → 07 → 18 → 19 → 21 → 20 → 22`.
