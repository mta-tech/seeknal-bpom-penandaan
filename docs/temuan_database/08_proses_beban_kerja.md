# 08 — Proses & Beban Kerja

> Piramida organisasi, Gama knowledge hub, rework 9,7%, reject loop.

---

## Piramida Organisasi (dari #user per status workflow)

```
[0] Draft        1.428 user   ← BANYAK operator daerah
[1] Supervisor   1.423 user   ← banyak supervisor daerah
[3] Kepala Balai   739 user   ← lebih sedikit
[4] MT             281 user   ← menyempit (tim pusat)
[5] Deputi          93 user   ← makin sempit
[6] Penyelia        30 user   ← spesialis
[7] Penguji         40 user   ← spesialis lab
[999] Selesai       22 user   ← SANGAT sedikit (approver final)
```

### Beban per orang (event/user) — meledak di puncak

| Level | event/user | Diagnosa |
|---|---|---|
| Operator (0) | 403 | Banyak tangan, beban ringan |
| Supervisor (1) | 364 | |
| Kepala Balai (3) | 663 | |
| MT (4) | 1.905 | Mulai berat |
| Deputi (5) | 3.867 | |
| Penyelia (6) | **8.380** ⚠ | Spesialis kewalahan |
| Penguji (7) | **8.839** ⚠ | |
| **Approver 999** | **15.931** ⚠⚠ | **22 orang × ~16.000 penyelesaian** |

> **Bottleneck bukan di daerah** (banyak tangan) **tapi di pusat** (sedikit tangan, beban ekstrem). Ini menjelaskan durasi `kabalai_direktur` median 21 hari — 30-40 spesialis pusat kewalahan menampung arus dari 1.400+ operator daerah.

---

## Gama = Knowledge Hub (Sentralitas Personel)

**Mohammad Gama Ramadhan** — Direktorat Pengawasan Kosmetik:
- 135.818 event (paling tinggi di seluruh sistem)
- 1 direktorat (Kosmetik), 3 status dikerjakan
- Catatan `"Yth. Pak Gama, mohon arahan..."` muncul **10.443×** di log

**Arti:** Gama bukan sekadar petugas — dia **simpul pengetahuan** tempat semua orang bertanya. Data mengungkap **struktur informal organisasi** yang tak ada di bagan resmi.

> ⚠️ **Risiko key-person dependency ganda:** 3 approver final (lihat [07_temuan_kritis](07_temuan_kritis.md) §temuan-2) + 1 knowledge hub (Gama). Jika Gama tidak ada, throughput kosmetik terganggu.

---

## Rework — 9,7% Sampel dengan Ekor Patologis

### Distribusi berapa kali sampel di-draft ulang

| n_draft | n_sampel | % | Kumulatif |
|---|---|---|---|
| 1× (clean) | 452.188 | 90,3% | 90,3% |
| 2× | 32.599 | 6,5% | 96,8% |
| 3× | 9.954 | 2,0% | 98,8% |
| 4× | 3.438 | 0,7% | 99,5% |
| 5× | 1.410 | 0,3% | 99,8% |
| 6-10× | 1.457 | 0,3% | 99,9%+ |
| 11-20× | ~80 | | |
| 21-33× | ~7 sampel | | **patologi** |

**Sampel di-draft 33×** = terjebak loop koreksi tanpa akhir. Tidak ada mekanisme eskalasi "keluar dari loop".

**Query verifikasi:**
```sql
SELECT n_draft, count(*) AS n_sampel FROM (
  SELECT id_penandaan, count(*) FILTER(WHERE status_code=0) AS n_draft
  FROM mv_penandaan_log GROUP BY id_penandaan
) x WHERE n_draft>=1 GROUP BY 1 ORDER BY 1;
```

---

## Reject Loop — Titik Gagal di Gerbang Awal

| Reject code | n event | n sampel unik | User |
|---|---|---|---|
| 991 (ditolak_spv_1) | 36.261 | 29.837 | 650 |
| 994 (ditolak_pusat) | 1.460 | 1.386 | 38 |
| 995 (ditolak_spv_1_pusat) | 1.415 | 1.393 | 15 |
| 993 (ditolak_kepala_balai) | 636 | 634 | 75 |
| 992 (ditolak_spv_2) | 78 | 72 | 12 |
| 996 (ditolak_spv_2_pusat) | 63 | 63 | 4 |
| 997 (ditolak_direktur) | 43 | 43 | 6 |

**Tingkat reject:** 39.957 rejection dari 3.533.299 event = **1,13%**.

> **91% reject terjadi di spv_1** (supervisor 1 daerah). **Titik lemah = kualitas input operator daerah.** Mayoritas koreksi terjadi SEBELUM sampel naik ke pusat — bukan di pusat.

### Biaya waktu reject (perlu ukur ulang)
Catatan: reject di spv_1 tidak terlihat di kolom `mulai_kabalai` (terjadi sebelum kabalai). Untuk ukur biaya total, pakai `tgl_end - tgl_start` (lihat [17_rencana_investigasi](17_rencana_investigasi_lanjut.md) §K6).

---

## Top Petugas per Direktorat (pusat)

| Direktorat | Event total | #user | Selesai (999) |
|---|---|---|---|
| KMEI ONPPZA | 590.086 | 36 | 146.822 |
| Kosmetik | 372.752 | 20 | 33.890 |
| OTSK | 243.923 | 15 | 8.686 |
| Peredaran Pangan Olahan | 2.519 | 10 | **0** ❌ |
| Distribusi dan Pelayanan ONPP | 1.928 | 3 | **0** ❌ |

> Direktorat Pangan Olahan & Distribusi ONPP = **0 completion**. Ini konsisten dengan dead-end PANGAN (lihat [07_temuan_kritis](07_temuan_kritis.md) §temuan-1).

---

## Koneksi ke Pertanyaan User

| Pola pertanyaan user | Hubungan dengan beban kerja | n pertanyaan |
|---|---|---|
| "top 5 UPT / UPT paling banyak melaporkan MK/TMK" | mencerminkan **beban output per balai** (jumlah laporan) | 6–10 |
| "rekapitulasi jumlah laporan masing-masing UPT" | **beban volume per balai** | 14 |
| "UPT yang tidak input catatan TMK" | kualitas input operator (terkait rework) | 5 |
| "tanggal proses direktur" | bottleneck puncak (approver pusat) sebagai dimensi waktu | 11 |

> **Insight:** Pertanyaan user tentang "UPT paling banyak melaporkan" secara data = `GROUP BY nama_balai COUNT(*)`
> (beban output), BUKAN beban event workflow. Jangan tertukar dengan piramida organisasi di atas (event per user).
> Lihat [21_sql_pairs_penandaan](21_sql_pairs_penandaan.md) SQL-03/SQL-04.

---

## Implikasi Operasional

1. **Bottleneck di puncak, bukan di dasar** — solusi "tambah operator daerah" tidak akan membantu; perlu tambah approver pusat
2. **Kualitas input operator lemah** — 5,96% sampel ditolak spv_1; training/validasi input lebih efektif daripada tambah petugas
3. **Key-person risk di 2 lapis** — 3 approver final + 1 knowledge hub
4. **Loop koreksi patologis** — sampel di-draft 33× butuh eskalasi otomatis (stop setelah N retry → human review)

---

Lanjut ke [09_waktu_durasi.md](09_waktu_durasi.md).
