# 07 — Temuan Kritis (3 Temuan Material)

> Evidence lengkap untuk 3 temuan paling material yang mempengaruhi audit tata kelola.

---

## 🔴 TEMUAN 1 — PANGAN & KEMASAN PANGAN: Workflow Jalan Buntu (0% Completion)

### Pernyataan temuan
**Tidak ada SATU PUN sampel PRODUK PANGAN atau KEMASAN PANGAN yang pernah mencapai status 999 (selesai)** di seluruh database. Workflow untuk kategori pangan **terputus permanen** setelah status 4 (MT).

### Evidence — Completion rate per komoditi

| Komoditi | Total | Selesai (999) | % | Stuck status 4 |
|---|---|---|---|---|
| OBAT | 53.163 | 52.427 | **98,6%** | 0,2% |
| SUPLEMEN KESEHATAN | 10.752 | 10.479 | 97,5% | 0,7% |
| OBAT KUASI | 2.687 | 2.606 | 97,0% | 1,0% |
| ROKOK | 32.082 | 31.012 | 96,7% | 1,7% |
| OBAT TRADISIONAL (OT) | 39.170 | 37.816 | 96,5% | 0,9% |
| KOSMETIKA | 78.495 | 74.949 | 95,5% | 2,3% |
| **PRODUK PANGAN** | **75.560** | **0** | **0,0%** | **99,8%** |
| **KEMASAN PANGAN** | **689** | **0** | **0,0%** | **100%** |

**Query verifikasi:**
```sql
SELECT komoditi, count(*) AS total,
  count(*) FILTER (WHERE t.status=999) AS selesai,
  round(100.0*count(*) FILTER (WHERE t.status=999)/count(*),1) AS pct
FROM mv_penandaan p LEFT JOIN mv_penandaan_timeline t ON t.id_penandaan=p.id
GROUP BY 1 ORDER BY pct DESC;
```

### Evidence — Pipeline event-level (pipa terputus)

| Step status | OBAT+ROKOK+KOSMETIKA | PRODUK PANGAN |
|---|---|---|
| 0 Draft | 163.740 | 75.560 |
| 1 Supervisor | 163.425 | 75.524 |
| 3 TPS/Kabala | 163.740 | 75.560 |
| 4 MT SPK | 163.740 | 75.560 |
| 5 Deputi | 161.314 | **168** |
| 6 Penyelia | 79.806 | **0** |
| 7 Penguji | 160.124 | **0** |
| **999 Selesai** | **158.428** | **0** |

### Anatomi PANGAN (75.560 sampel)
- 75.392 (99,8%) mandek di status 4
- 168 (0,2%) di status 5
- **0 di status 6, 7, atau 999**
- TAPI `kesimpulan_penilaian_pusat` ADA: **VP=54.117, MK=16.840**

### Paradox yang menjelaskan segalanya
PANGAN **sudah dinilai** (kesimpulan_pusat terisi), tapi workflow **tidak pernah ditutup** (status macet 4). Keputusan dibuat, proses tidak diakhiri. **Diskonek antara assessment dan closure.**

### Direktorat pemroses — bukti organisasional
| Direktorat | Event | Selesai |
|---|---|---|
| KMEI ONPPZA (OBAT+ROKOK) | 590.086 | 146.822 |
| Kosmetik | 372.752 | 33.890 |
| OTSK | 243.923 | 8.686 |
| **Peredaran Pangan Olahan** | **2.519** | **0** ❌ |
| Distribusi dan Pelayanan ONPP | 1.928 | 0 |

**Direktorat pangan memproses 2.519 event tapi menyelesaikan 0.** Dead-end terjadi di tingkat direktorat.

### Umur limbo = bukti kepermanenan
| Tahun mulai | VP stuck | Median umur (hari) | Max umur |
|---|---|---|---|
| 2023 | 11.983 | **1.128 (3,1 tahun)** | 1.319 |
| 2024 | 13.671 | **740 (2 tahun)** | 954 |
| 2025 | 17.706 | 407 (1,1 tahun) | 588 |
| 2026 | 13.532 | 95 | 223 |

VP PANGAN dari 2023 sudah menggantung **3+ tahun**. Ini bukan antrian lambat — ini **pengabaian terstruktur**.

### Implikasi audit
1. **76.249 sampel pangan terkubur permanen** (75.560 PANGAN + 689 KEMASAN) → utang proses terbesar
2. **KPI penandaan pangan menyesatkan** — "realisasi" ada (sampel masuk) tapi "penyelesaian" 0%
3. **Sistem e-Penandaan untuk pangan fundamentally broken** — butuh fix workflow ATAU kebijakan eksplisit
4. **Target 2024 PANGAN terlampaui 101%** ([14_kpi_target_2024](14_kpi_target_2024.md)) TAPI 0 selesai → pencapaian target = pikiran semu

---

## 🔴 TEMUAN 2 — 3 Orang = 72% Approval Nasional (Key-Person Dependency Ekstrem)

### Pernyataan temuan
Distribusi 22 approver final (status 999) **sangat timpang** — 3 individu memproses 72,2% seluruh penyelesaian nasional. Sistem e-Penandaan Indonesia bergantung pada 3 orang di 3 Balai Besar.

### Evidence — Distribusi approver final

| Rank | Approver | Balai | n selesai | % | Kumulatif |
|---|---|---|---|---|---|
| 1 | Irwan, S.Si., Apt., M.KM | BBPOM SEMARANG | 106.611 | **30,4%** | 30,4% |
| 2 | Nova Emelda, S.Si, MS, Apt | BBPOM MEDAN | 83.750 | 23,9% | **54,3%** |
| 3 | Tri Asti Isnariani, Apt., M.Pharm | BBPOM BANJARBARU | 62.854 | 17,9% | **72,2%** |
| 4 | Dra. Rustyawati, Apt., M.Kes (Epid) | — | 36.481 | 10,4% | 82,6% |
| 5 | Drs. Arustiyono, Apt., MPH | — | 31.955 | 9,1% | **91,7%** |
| 6-22 | (17 orang lain) | — | 28.385 | 8,3% | 100% |

**Query verifikasi:**
```sql
SELECT fullname, count(*) AS n_selesai,
       round(100.0*count(*)/sum(count(*)) OVER (),1) AS pct
FROM mv_penandaan_log WHERE status_code=999
GROUP BY fullname ORDER BY n_selesai DESC;
```

### Gama = Knowledge Hub (key-person dependency ganda)
**Mohammad Gama Ramadhan** (Direktorat Pengawasan Kosmetik):
- 135.818 event (1 direktorat, 3 status)
- Catatan "Yth. Pak Gama, mohon arahan..." muncul **10.443×** di log
- Dia = simpul pengetahuan pusat untuk kosmetik — semua orang bertanya padanya

### Implikasi audit
1. **Single point of failure total** — jika Irwan/Nova/Tri Asti cuti/pindah/sakit, 30-70% throughput nasional terhenti
2. **Tidak ada redundansi** — 3 orang pegang kunci seluruh sistem
3. **Risiko operasional tertinggi** — kelangsungan pengawasan nasional bergantung individu
4. **Penjelasan bottleneck durasi** (kabalai→direktur median 21 hari): bukan karena kerja sulit, tapi karena terlalu sedikit tangan di puncak

### Rekomendasi
- Bangun **bench-strength**: identifikasi & training penerus top-3
- Dekonsentrasi wewenang approval ke lebih banyak Balai Besar
- Dokumentasikan SOP Gama (knowledge transfer) sebelum retirement

---

## 🔴 TEMUAN 3 — 41,6% Produk Identik Dapat Keputusan Berbeda Lintas Balai

### Pernyataan temuan
Dari 12.698 produk (produsen + nama) yang diperiksa ≥3 kali, **41,6% mendapat kesimpulan pusat berbeda** antar pemeriksaan. 99,3% disagreement terjadi **lintas balai** — standar penilaian tidak seragam nasional.

### Evidence — Konsistensi keputusan produk berulang

| Metrik | Angka |
|---|---|
| Produk diperiksa ≥3× | 12.698 |
| Produk dengan keputusan berbeda | **5.277 (41,6%)** |
| Disagreement lintas balai | 5.240 (**99,3%** dari disagreement) |
| Disagreement dalam 1 balai | 37 (0,7%) |
| Produk konsisten (keputusan sama) | 7.421 (58,4%) |

**Query verifikasi:**
```sql
WITH rk AS (
  SELECT trim(regexp_replace(produsen,'\s{2,}',' ','g')) AS prod, nama_produk,
    count(DISTINCT kesimpulan_penilaian_pusat) AS n_kpp,
    count(DISTINCT nama_balai) AS n_balai, count(*) AS n
  FROM mv_penandaan WHERE produsen<>'' AND nama_produk<>''
  GROUP BY 1,2 HAVING count(*)>=3 AND count(DISTINCT kesimpulan_penilaian_pusat)>=2
)
SELECT count(*) AS beda_keputusan,
  count(*) FILTER(WHERE n_balai>=2) AS lintas_balai,
  count(*) FILTER(WHERE n_balai=1) AS satu_balai_saja FROM rk;
```

### Interpretasi
- **Konsistensi internal balai BAIK** (99,3%) — dalam 1 balai, produk identik dinilai konsisten
- **Kalibrasi antar-balai RUSAK** (41,6%) — produk identik dapat MK di Balai A, TMK di Balai B
- Ini **inter-rater disagreement terstruktur** — bukan kebetulan, pola sistemik

### Penjelasan yang mungkin
1. Standar subjektif antar penilai balai
2. Regulasi berubah antar waktu pemeriksaan
3. Bias regional (balai tertentu lebih ketat/santun)
4. Komoditi confound — produk sama dinilai balai berbeda karena konteks lokal

### Implikasi audit
1. **Keputusan penandaan TIDAK bisa dipakai sebagai kebenaran objektif** tanpa konteks siapa-menilai
2. Produsen yang sama bisa **lulus di satu daerah, ditolak di daerah lain** — ketidakpastian regulasi
3. **Kalibrasi nasional perlu di-overhaul** — training & SOP seragam lintas balai
4. Auditor perlu sampling kasus disagreement untuk audit kualitas penilaian

### Rekomendasi investigasi lanjut
Lihat [17_rencana_investigasi_lanjut](17_rencana_investigasi_lanjut.md) §K4 — pola transisi keputusan & korelasi waktu.

---

## Sintesis: 3 Temuan Saling Menguatkan

```
[PANGAN: pipa terputus di MT]      [3 orang = 72% approval]
         ↓                                  ↓
         ↓                          [kapasitas pusat sangat terbatas]
         ↓                                  ↓
[VP pangan = limbo] ◄──────────── [VP = "keranjang keraguan" — enggan tolak]
         ↓
[76.249 sampel terkubur permanen]


[41,6% produk identik beda keputusan lintas balai]
         ↓
[Standar penilaian tidak seragam → kalibrasi nasional rusak]
         ↓
[Diperparah: OBAT/ROKOK dinilai pusat (ketat) vs lainnya dinilai balai (longgar)]
```

**Ketiga temuan mengarah ke satu kesimpulan:** sistem e-Penandaan punya **masalah tata kelola struktural** — workflow broken untuk pangan, key-person dependency ekstrem, dan standar penilaian tidak seragam. Ini bukan masalah teknis DB, tapi masalah proses bisnis yang terekam jelas di data.

---

## Koneksi ke Pertanyaan User

Validasi terhadap **101 pertanyaan user asli** (export KAI) — mana temuan yang user tanyakan langsung:

| Temuan | Pertanyaan user terkait | n |
|---|---|---|
| **Temuan 3** (41,6% beda keputusan lintas balai) | "tampilkan data gap/perbedaan hasil pengawasan antara pusat dan UPT" | 15 |
| **Temuan 3** + 3-level kesimpulan | "kesimpulan akhir MK/TMK", "kesimpulan UPT", "apa perbedaan Kesimpulan Balai dan Pusat" | ~27 |
| **Temuan 2** (key-person) | TIDAK langsung ditanya — tapi relevan utk jawab "top UPT" | 0 |
| **Temuan 1** (PANGAN 0%) | TIDAK langsung ditanya — PANGAN muncul via "pangan olahan", "label pangan" | 3 |

> **Insight penting:** Temuan #3 adalah bukti kuantitatif untuk menjawab pertanyaan "gap pusat vs UPT" yang paling
> sering (15×). Agent harus memakai angka 41,6% / 152.577 baris gap sebagai konteks ketika user tanya gap.
> Temuan #1 dan #2 TIDAK muncul di pertanyaan harian — tetap material untuk audit struktural, tapi bukan kebutuhan
> langsung user. Lihat [00_ringkasan_eksekutif](00_ringkasan_eksekutif.md) §Koneksi + [22_prioritas_use_case](22_prioritas_use_case.md).

---

Lanjut ke [08_proses_beban_kerja.md](08_proses_beban_kerja.md).
