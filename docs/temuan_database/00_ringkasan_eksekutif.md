# 00 — Ringkasan Eksekutif

> Snapshot: `sync = 2026-08-11 22:53:20` · 292.598 baris fact · 3.533.299 baris log · 500.717 baris timeline

---

## 3 TEMUAN PALING MATERIAL (Kritis untuk Audit)

### 🔴 TEMUAN 1 — PRODUK PANGAN & KEMASAN PANGAN TIDAK PERNAH SELESAI (0%)

Database e-Penandaan punya **workflow jalan buntu** untuk seluruh kategori pangan.

| Komoditi | Total sampel | Selesai (status 999) | % completion |
|---|---|---|---|
| OBAT | 53.163 | 52.427 | **98,6%** |
| SUPLEMEN KESEHATAN | 10.752 | 10.479 | 97,5% |
| OBAT KUASI | 2.687 | 2.606 | 97,0% |
| ROKOK | 32.082 | 31.012 | 96,7% |
| OBAT TRADISIONAL (OT) | 39.170 | 37.816 | 96,5% |
| KOSMETIKA | 78.495 | 74.949 | 95,5% |
| **PRODUK PANGAN** | **75.560** | **0** | **0,0%** |
| **KEMASAN PANGAN** | **689** | **0** | **0,0%** |

**Bukti event-level (pipa terputus setelah MT):** 75.560 sampel PANGAN mencapai status 4 (MT), tapi hanya 168 sampai status 5 (Deputi), dan **0 sampai status 6, 7, atau 999**. Workflow PANGAN tidak punya jalur ke penyelesaian.

**Implikasi:** 76.249 sampel pangan terkubur permanen (limbo). Direktorat Pengawasan Peredaran Pangan Olahan: **0 completion** (2.519 event diproses, tak satupun selesai). Ini bukan "lambat" — ini **dead-end by design**.

**Umur limbo membuktikan kepermanenan:**
- PANGAN VP 2023: median **1.128 hari (3,1 tahun)**, max 1.319 hari
- 2024: median 740 hari (2 tahun)
- 2025: 17.706 sampel, median 407 hari

---

### 🔴 TEMUAN 2 — 3 Orang = 72% Penyelesaian Nasional

Distribusi 22 approver final (status 999):

| Rank | Approver | Balai | n selesai | % |
|---|---|---|---|---|
| 1 | Irwan, S.Si., Apt., M.KM | BBPOM SEMARANG | 106.611 | **30,4%** |
| 2 | Nova Emelda, S.Si, MS, Apt | BBPOM MEDAN | 83.750 | 23,9% |
| 3 | Tri Asti Isnariani, Apt., M.Pharm | BBPOM BANJARBARU | 62.854 | 17,9% |
| 4 | Rustyawati | — | 36.481 | 10,4% |
| 5 | Arustiyono | — | 31.955 | 9,1% |

**Top 3 = 72,2% seluruh penyelesaian nasional. Top 5 = 91,7%.** Sisanya 17 orang bagi 8,3%.

**Implikasi:** Sistem e-Penandaan Indonesia bertumpu pada **3 individu di 3 Balai Besar**. Jika salah satu cuti/pindah/sakit, 30% throughput nasional terhenti. **Single point of failure total**. Ditambah Mohammad Gama Ramadhan (Direktorat Kosmetik) sebagai knowledge hub (135.818 event, "Yth. Pak Gama" muncul 10.443× di catatan) → key-person dependency ganda.

---

### 🔴 TEMUAN 3 — 41,6% Produk Identik Dapat Keputusan Berbeda Lintas Balai

- 12.698 produk (produsen + nama) diperiksa ≥3 kali
- **5.277 (41,6%)** mendapat kesimpulan pusat berbeda antar pemeriksaan
- **5.240 (99,3%)** disagreement terjadi **LINTAS balai** (inter-rater antar-daerah)
- Hanya 37 (0,7%) dalam satu balai → konsistensi internal baik

**Implikasi:** Standar penilaian **tidak seragam nasional**. Produk identik bisa MK di Balai A tapi TMK di Balai B. Ini bukti *inter-rater disagreement* terstruktur — masalah kalibrasi nasional, bukan satu balai.

---

## RANKING PRIORITAS AUDIT (23 temuan)

| # | Prioritas | Temuan | Materialitas |
|---|---|---|---|
| 1 | 🔴 KRITIS | PANGAN 0% completion — pipa terputus di MT (76k sampel) | Workflow broken by design |
| 2 | 🔴 KRITIS | 3 orang = 72% approval nasional | Single point of failure |
| 3 | 🔴 TINGGI | 41,6% produk identik beda keputusan lintas balai | Standar tidak seragam |
| 4 | 🟠 SEDANG | OBAT/ROKOK dinilai pusat (standar beda) → bias | Governance bias |
| 5 | 🟠 SEDANG | 6.312 baris tanggal 1970 (Unix epoch) | Data quality |
| 6 | 🟠 SEDANG | 18.666 backlog 2021 tersembunyi di orphan | Backlog tersembunyi |
| 7 | 🟠 SEDANG | Anomali Q1 2025 sistemik tak terjelaskan | Gangguan operasional |
| 8 | 🟡 KECIL | Rework 9,7% + sampel di-draft 33× | Kualitas input |
| 9 | 🟡 KECIL | 12 prefix kode balai ambigu (99k sampel) | Data quality |
| 10 | ✅ POSITIF | Target 2024 semua terlampaui (101-116%) | KPI sehat |

---

## PETA KAUSAL UTUH (Semua Titik Terhubung)

```
                    KOMODITI (governing dimension)
                          │
        ┌─────────────────┼──────────────────────┐
        ▼                 ▼                      ▼
   [FORM berbeda]    [SIAPA MENILAI]       [TAKSONOMI keputusan]
   kosmetik: no      OBAT/ROKOK:            biner vs VP vs Mayor/Minor
   pendaftar         pusat langsung            │
   pangan: no        → 90% TMK               VP = "keranjang keraguan"
   produsen          (standar ketat)            │
        │                 │                      │
        │          [BALAI nilai beda]           ▼
        │          inter-rater 41,6%       [PUSAT enggan tolak]
        │          lintas balai                 → semua tersedot VP
        │                                      │
        ▼                                      ▼
   [PANGAN: pipa terputus di MT] ◄──── [VP pangan = limbo]
   0 selesai · 76.249 terkubur · median 3 thn
              │
      [3 orang = 72% approval nasional]
      [balai terpencil lambat 31-37 hari]
              │
              ▼
      UTANG PROSES STRUKTURAL BESAR
```

**Tiga temuan inti saling menguatkan:**
1. **Komoditi mengendalikan segalanya** — form, alur penilaian, taksonomi, pola null. Semua "kekotoran" data = aturan bisnis per-komoditi.
2. **VP + Status-4 + PANGAN = satu utang proses** (~76k sampel limbo), akibat VP dipakai menunda × kapasitas pusat terbatas.
3. **Tiga isolasi data:** orphan 208k (waktu), dimension stale (SSoT ganda), 12 kode balai ambigu (identitas).

---

## KOREKSI ATAS ASUMSI CONTEXT AWAL

| Asumsi context v1.0 | Temuan sebenarnya |
|---|---|
| "KPI buta" (target_balai tak bisa di-join) | ❌ Target 2024 **terlampaui 101-116%**; join berhasil dengan `lower()` + agregasi terpisah |
| `direktur_pusat` = flag (bukan hari) | ❌ Memang **dalam hari**; 98,4% = 0 (instan), ekor 1-209 |
| Kolom durasi = selisih tanggal | ❌ Hanya **4,5% cocok** dengan selisih tanggal; dihitung logika lain |
| Total baris 292.169 | ⚠️ Kini **292.598** (ETL refresh, +429) |
| "23,9% sampel stuck" umum | ⚠️ Lebih tajam: **95,4% status-4 adalah PANGAN** (bukan acak) |

---

## KONEKSI KE PERTANYAAN USER (Validasi dari Export KAI)

Dokumentasi di bawah (18–22) memvalidasi temuan kita terhadap **101 pertanyaan user asli** dari export KAI
(`kai_question_sql_pairs.csv`, filter `db_alias LIKE 'penandaan%'`). Ringkasan koneksinya:

### Temuan yang DIKONFIRMASI user tanyakan ✅

| Temuan kita | Pola pertanyaan user | n pertanyaan |
|---|---|---|
| Temuan #3 (41,6% produk beda keputusan) | "gap/perbedaan antara pusat dan UPT" | 15 |
| 3 level kesimpulan (balai/pusat/akhir) | "kesimpulan akhir MK/TMK", "kesimpulan UPT" | 27 |
| Geografi (10) — ranking UPT | "Balai/UPT mana tertinggi/terendah", "top 5 UPT" | 6–10 |
| Geografi (10) — rekap per balai | "rekapitulasi masing-masing UPT" | 14 |
| Waktu (09) — tanggal direktur | "berdasarkan tanggal proses direktur" | 11 |
| Waktu (09) — timeline pemenuhan | "pemenuhan timeline ... maksimal tgl 15" | 5 |
| Penilaian (06) — catatan kosong 62% | "UPT yang tidak input catatan TMK" | 5 |
| Komoditi (04) — triplet OT/Supkes/Kuasi | "obat tradisional; suplemen kesehatan; obat kuasi" | 6 |

### Temuan yang TIDAK ditanya user (tetap penting untuk audit internal)

PANGAN 0% completion (#1), 3 approver = 72% (#2), orphan 208k, dimension stale, agg validation, kode PW.XX,
VP multi-makna, rework 9,7% — mayoritas TIDAK muncul di pertanyaan user harian, tapi tetap material untuk audit
struktural dan menjadi bukti kuantitatif di balik jawaban user (mis. disagreement → gap pusat-UPT).

### Gap yang user tanya tapi DB tidak punya ⚠️

| User tanya | Kolom DB | Solusi |
|---|---|---|
| "jenis/nama kemasan" (3–5) | ❌ tidak ada | Laporkan gap (G1) |
| "kabupaten/provinsi produsen" (2–4) | ❌ tidak ada | Butuh master eksternal (G2) |
| "media publikasi/klaim iklan" (7) | ❌ domain `pengawasan` | Arahkan ke DB pengawasan (G3) |

> Detail lengkap: [18_pola_pertanyaan_user](18_pola_pertanyaan_user.md) → [20_gap_schema_user](20_gap_schema_user.md).
> SQL pairs tervalidasi: [21_sql_pairs_penandaan](21_sql_pairs_penandaan.md).

---

## Angka Kunci untuk Dijingat

| Metrik | Angka |
|---|---|
| Total penandaan (baris fact) | 292.598 |
| Event workflow (log) | 3.533.299 |
| Sampel dengan timeline | 500.717 |
| Komoditi | 8 |
| Balai aktif | 83 (fact), 88 (coverage) |
| Status workflow | 19 (11 operasional + 8 reject) |
| Kode kesimpulan pusat | 5 (MK, TMK, VP, TMK MAYOR, TMK MINOR) |
| Orphan (id tanpa induk fact) | 208.119 |
| Approval approver top-1 | 30,4% (Irwan, SEMARANG) |
| Completion PANGAN | 0% |
| Completion rata-rata non-PANGAN | 96,9% |

---

Lihat detail per temuan di file-file berikutnya. Mulai dari [01_identitas_domain.md](01_identitas_domain.md) untuk pemahaman dasar, atau langsung ke [07_temuan_kritis.md](07_temuan_kritis.md) untuk evidence lengkap 3 temuan material.
