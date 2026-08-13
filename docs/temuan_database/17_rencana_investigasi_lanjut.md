# 17 — Rencana Investigasi Lanjut (SQL Siap Eksekusi)

> Daftar query verifikasi untuk menutup gap di [15_gap_belum_ditemukan](15_gap_belum_ditemukan.md). K1-K11, prioritas + SQL.

---

## Cara Pakai

Setiap kelompok: **Q** (pertanyaan) → **Mengapa** → **Ekspektasi** → **SQL** (siap eksekusi di `psql "$PGURL"`).

```bash
export PGURL=$(grep '^WAREHOUSE_URL=' /home/mta/projects/seeknal_audit/seeknal-bpom-penandaan/.env | cut -d= -f2- | tr -d '"')
psql "$PGURL" -c "<SQL di bawah>"
```

---

## 🔴 KELOMPOK 1 — Akar Penyebab PANGAN Dead-End (PRIORITAS TERTINGGI)

### K1.1 Adakah sampel PANGAN yg pernah mencapai status 6/7/999?
- **Mengapa:** Membuktikan apakah pipa benar-benar terputus, atau ada jalur sempit tak terlihat
- **Ekspektasi:** 0 baris → konfirmasi total dead-end

```sql
SELECT p.komoditi, l.status_code, l.status_label, count(*) AS n,
       count(DISTINCT l.id_penandaan) AS n_sampel
FROM public.mv_penandaan_log l
JOIN public.mv_penandaan p ON p.id=l.id_penandaan
WHERE p.komoditi IN ('PRODUK PANGAN','KEMASAN PANGAN')
  AND l.status_code IN (6,7,999)
GROUP BY 1,2,3;
```

### K1.2 Kapan PANGAN mulai stuck? Tren completion per tahun
- **Ekspektasi:** 0 selesai SETIAP tahun → bug struktural sejak awal

```sql
SELECT extract(year FROM p.tgl_start)::int AS yr, t.status, count(*) AS n
FROM public.mv_penandaan p
JOIN public.mv_penandaan_timeline t ON t.id_penandaan=p.id
WHERE p.komoditi='PRODUK PANGAN'
GROUP BY 1,2 ORDER BY 1,2;
```

---

## 🔴 KELOMPOK 2 — VP-Limbo: Apakah Masih Bergerak?

### K2.1 Apakah sampel VP PANGAN masih disentuh di log setelah status 4?
- **Mengapa:** Membedakan "sedang diproses" vs "abandoned"
- **Ekspektasi:** Bila event terakhir di 2023-2024 → abandoned permanen

```sql
WITH last_evt AS (
  SELECT id_penandaan, max(tanggal_proses) AS last_proses, count(*) AS n_event
  FROM public.mv_penandaan_log GROUP BY id_penandaan
)
SELECT extract(year FROM l.last_proses)::int AS last_yr,
       count(*) AS n_sampel, round(avg(l.n_event),1) AS avg_event
FROM public.mv_penandaan_timeline t
JOIN last_evt l ON l.id_penandaan=t.id_penandaan
WHERE t.status=4
GROUP BY 1 ORDER BY 1;
```

---

## 🟠 KELOMPOK 3 — Kamus Kode Regulasi

### K3.1 Distribusi kode MKL/TME/NIK per komoditi
- **Ekspektasi:** MKL.* hanya di KOSMETIKA, TME.* di PANGAN, NIK.* lintas

```sql
SELECT p.komoditi, substring(p.catatan from '([A-Z]{2,4}\.[0-9]{2})') AS kode, count(*) AS n
FROM public.mv_penandaan p
WHERE p.catatan ~ '[A-Z]{2,4}\.[0-9]{2}'
GROUP BY 1,2 ORDER BY 1,3 DESC;
```

### K3.2 Konteks kalimat di sekitar MKL.02
- **Ekspektasi:** Pola teks membantu decode makna

```sql
SELECT DISTINCT catatan FROM public.mv_penandaan
WHERE catatan ~ 'MKL.02' ORDER BY catatan LIMIT 20;
```

---

## 🟠 KELOMPOK 4 — Pola Disagreement Produk

### K4.1 Transisi keputusan dominan (MK→TMK atau sebaliknya?)
- **Ekspektasi:** Arah bias — apakah pusat cenderung menaikkan/menurunkan ketegasan

```sql
WITH rk AS (
  SELECT trim(regexp_replace(produsen,'\s{2,}',' ','g')) AS prod, nama_produk,
    string_agg(DISTINCT kesimpulan_penilaian_pusat,',' ORDER BY kesimpulan_penilaian_pusat) AS combo,
    count(*) AS n
  FROM public.mv_penandaan WHERE produsen<>'' AND nama_produk<>''
  GROUP BY 1,2 HAVING count(*)>=3 AND count(DISTINCT kesimpulan_penilaian_pusat)>=2
)
SELECT combo, count(*) AS n_prod FROM rk GROUP BY 1 ORDER BY 2 DESC LIMIT 15;
```

### K4.2 Korelasi disagreement dengan selisih waktu antar pemeriksaan
- **Ekspektasi:** Disagreement karena regulasi berubah (span besar) vs penilai beda (span kecil)

```sql
WITH rk AS (
  SELECT trim(regexp_replace(produsen,'\s{2,}',' ','g')) AS prod, nama_produk,
    max(tgl_start)-min(tgl_start) AS span_hari,
    count(DISTINCT kesimpulan_penilaian_pusat) AS n_kpp
  FROM public.mv_penandaan WHERE produsen<>'' AND nama_produk<>''
  GROUP BY 1,2 HAVING count(*)>=3)
SELECT n_kpp, round(avg(span_hari),0) AS avg_span, count(*) AS n FROM rk GROUP BY 1;
```

---

## 🟠 KELOMPOK 5 — Metodologi Durasi

### K5.1 Apakah mulai_kabalai = business days?
- **Mengapa:** Menjelaskan mengapa hanya 4,5% cocok selisih kalender
- **Ekspektasi:** Cocok tinggi jika hari kerja

```sql
SELECT count(*) AS n,
  count(*) FILTER (WHERE mulai_kabalai = (tanggal_kirim_kabalai - tgl_start)) AS cocok_kalender,
  count(*) FILTER (WHERE mulai_kabalai = (
    SELECT count(*) FROM generate_series(t.tgl_start, t.tanggal_kirim_kabalai, '1 day') d
    WHERE extract(dow from d) NOT IN (0,6)
  ) AND t.tanggal_kirim_kabalai > t.tgl_start) AS cocok_hari_kerja
FROM public.mv_penandaan_timeline t
WHERE tanggal_kirim_kabalai IS NOT NULL AND tgl_start IS NOT NULL;
```

---

## 🟠 KELOMPOK 6 — Biaya Rework Sesungguhnya

### K6.1 Total durasi (tgl_end−tgl_start) pernah-ditolak vs lolos
- **Mengapa:** Reject spv_1 tak terlihat di mulai_kabalai. Ukur total.
- **Ekspektasi:** Sampel ditolak → total durasi 2-3× lebih panjang

```sql
WITH rej AS (SELECT DISTINCT id_penandaan FROM public.mv_penandaan_log WHERE status_code=991)
SELECT CASE WHEN r.id_penandaan IS NULL THEN 'lolos' ELSE 'pernah_ditolak' END AS grp,
       count(*) AS n, round(avg(p.tgl_end-p.tgl_start),1) AS avg_total,
       percentile_cont(0.5) WITHIN GROUP (ORDER BY p.tgl_end-p.tgl_start) AS median_total
FROM public.mv_penandaan p LEFT JOIN rej r ON r.id_penandaan=p.id
WHERE p.tgl_end IS NOT NULL GROUP BY 1;
```

### K6.2 Profil sampel di-draft ≥10× (patologi)
- **Ekspektasi:** Identifikasi komoditi/balai sampel patologis

```sql
SELECT p.komoditi, p.nama_balai, count(*) AS n_patologi
FROM (SELECT id_penandaan FROM public.mv_penandaan_log
      GROUP BY 1 HAVING count(*) FILTER(WHERE status_code=0)>=10) x
JOIN public.mv_penandaan p ON p.id=x.id_penandaan
GROUP BY 1,2 ORDER BY 3 DESC LIMIT 15;
```

---

## 🟡 KELOMPOK 7 — Profil Orphan 2021

### K7.1 Komoditi orphan status-4 tahun 2021 (rekonstruksi dari log)
- **Ekspektasi:** Apakah backlog 2021 juga dominan PANGAN?

```sql
SELECT substring(l.catatan from 1 for 40) AS sample_catatan, count(DISTINCT l.id_penandaan) AS n
FROM public.mv_penandaan_log l
WHERE l.id_penandaan IN (
  SELECT t.id_penandaan FROM public.mv_penandaan_timeline t
  LEFT JOIN public.mv_penandaan p ON p.id=t.id_penandaan
  WHERE p.id IS NULL AND t.status=4)
GROUP BY 1 ORDER BY 2 DESC LIMIT 10;
```

---

## 🟡 KELOMPOK 8 — Geografi & Kapasitas

### K8.1 Korelasi n_kabupaten vs durasi per balai
- **Ekspektasi:** Balai beban kabupaten tinggi → durasi lebih panjang?

```sql
WITH cv AS (SELECT nama_balai, count(*) AS n_kab FROM public.coverage_balai GROUP BY 1),
dur AS (
  SELECT p.nama_balai, percentile_cont(0.5) WITHIN GROUP (ORDER BY t.kabalai_direktur) AS med
  FROM public.mv_penandaan_timeline t JOIN public.mv_penandaan p ON p.id=t.id_penandaan
  WHERE t.status=999 AND p.komoditi<>'PRODUK PANGAN' GROUP BY 1
)
SELECT cv.nama_balai, cv.n_kab, dur.med
FROM cv JOIN dur ON dur.nama_balai=cv.nama_balai
ORDER BY cv.n_kab DESC LIMIT 15;
```

---

## 🟡 KELOMPOK 9 — Konsistensi Detail agg

### K9.1 Cek cell agg vs fact yang berbeda (per balai × bulan)
- **Mengapa:** Grand total cocok, tapi apakah per cell?
- **Ekspektasi:** Identifikasi cell offset (saling meniadakan di grand total)

```sql
WITH fa AS (
  SELECT nama_balai, to_char(tgl_start,'YYYY-MM') AS bln, count(*) AS n
  FROM public.mv_penandaan GROUP BY 1,2
),
ag AS (
  SELECT nama_balai, to_char(tanggal_periode,'YYYY-MM') AS bln, sum(jumlah_penandaan) AS n
  FROM public.mv_penandaan_agg WHERE periode_type='month' GROUP BY 1,2
)
SELECT fa.nama_balai, fa.bln, fa.n AS fact_n, ag.n AS agg_n, fa.n-ag.n AS diff
FROM fa JOIN ag ON ag.nama_balai=fa.nama_balai AND ag.bln=fa.bln
WHERE fa.n<>ag.n ORDER BY abs(fa.n-ag.n) DESC LIMIT 15;
```

---

## 🟡 KELOMPOK 10 — Struktur Kode

### K10.1 Decode segmen ke-3 (bidang) nomor_surat
- **Ekspektasi:** Kode bidang/sub-direktorat

```sql
SELECT split_part(nomor_surat,'.',3) AS bidang,
       split_part(nomor_surat,'.',2) AS direktorat,
       count(DISTINCT nama_balai) AS n_balai, count(*) AS n
FROM public.mv_penandaan
WHERE nomor_surat ~ '^PW\.[0-9]+\.[0-9A-Z]+\.'
GROUP BY 1,2 ORDER BY 1,4 DESC LIMIT 20;
```

### K10.2 Validasi prefix tahun nomorsampel vs tgl_start
- **Ekspektasi:** Prefix 2-digit tahun = `extract(year from tgl_start) % 100`

```sql
SELECT substring(nomorsampel from 1 for 2) AS yy_sample,
       extract(year FROM tgl_start)::int % 100 AS yy_tgl, count(*) AS n
FROM public.mv_penandaan
WHERE nomorsampel ~ '^\d\d\.' AND nomorsampel<>''
GROUP BY 1,2
HAVING substring(nomorsampel from 1 for 2) <> (extract(year FROM tgl_start)::int % 100)::text
ORDER BY 3 DESC LIMIT 10;
```

---

## ⚪ KELOMPOK 11 — Eksternal (TANPA SQL)

Lihat [16_informasi_eksternal_dibutuhkan](16_informasi_eksternal_dibutuhkan.md) — butuh dokumen/wawancara BPOM.

| # | Dokumen | Untuk |
|---|---|---|
| 1 | SOP workflow PANGAN + wawancara direktorat pangan | Root cause temuan #1 (PANGAN 0%) |
| 2 | Struktur organisasi + wewenang approver | Root cause temuan #2 (key-person) |
| 3 | Kamus kode MKL/TME/NIK | Decode pelanggaran (K3) |
| 4 | SOP VP/MK/TMK + Mayor/Minor | Konfirmasi tafsir |
| 5 | SLA per tahap | Konteks durasi (K5) |
| 6 | Target 2023/2025/2026 | KPI lengkap |
| 7 | Master kode balai | Validasi silang (K10) |

---

## Ringkasan Prioritas Eksekusi

| Prioritas | Kelompok | Mengapa |
|---|---|---|
| 🔴 KRITIS | K1 (akar PANGAN), K2 (VP-limbo movement) | Konfirmasi temuan terbesar |
| 🟠 TINGGI | K3 (kamus kode), K4 (disagreement), K5 (durasi), K6 (rework) | Perdalam interpretasi |
| 🟡 SEDANG | K7 (orphan), K8 (geografi), K9 (konsistensi) | Validasi pelengkap |
| 🟡 RENDAH | K10 (struktur kode) | Housekeeping |
| ⚪ EKSTERNAL | K11 | Kumpul paralel dari BPOM |

---

## Setelah Eksekusi

Update dokumen temuan dengan hasil verifikasi:
- Hipotesis `🟡` di [15_gap](15_gap_belum_ditemukan.md) → naik ke `✅` atau `❌`
- Tambah evidence ke [07_temuan_kritis](07_temuan_kritis.md) bila menemukan yang baru
- Update [00_ringkasan_eksekutif](00_ringkasan_eksekutif.md) ranking prioritas

---

**Selesai.** Kembali ke [README.md](README.md).
