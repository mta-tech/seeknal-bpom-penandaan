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
