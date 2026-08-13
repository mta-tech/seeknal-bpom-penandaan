# 06 — Penilaian & Keputusan

> Taksonomi kesimpulan, VP multi-makna, matriks transisi balai→pusat.

---

## 3 Kolom Verdict (pilih sesuai pertanyaan)

| Kolom | Distinct values | Kapan dipakai |
|---|---|---|
| `kesimpulan_penilaian_balai` | MK (177.080), TMK (14.113), TMK MINOR (7.407+1.864 dup), TMK MAYOR (6.592), NULL (85.113) | Penilaian tingkat **daerah**. 29% null = OBAT/ROKOK skip |
| `kesimpulan_penilaian_pusat` | MK, TMK, VP, TMK MAYOR, TMK MINOR (5 nilai, 0 null) | Penilaian **FINAL pusat**. Authoritative |
| `catatan` | kode regulasi | Alasan/kode pelanggaran |

⚠️ **Casing dup di kesimpulan_balai:** "TMK Minor" vs "TMK MINOR". Normalisasi sebelum filter:
```sql
CASE WHEN kesimpulan_penilaian_balai = 'TMK Minor' THEN 'TMK MINOR'
     ELSE kesimpulan_penilaian_balai END
```

---

## Taksonomi Kesimpulan Pusat (FINAL)

| Kode | Makna | Komoditi | Interpretasi bisnis |
|---|---|---|---|
| **MK** | Memenuhi Ketentuan | semua 8 | Label **COMPLIES**. Produk boleh beredar |
| **TMK** | Tidak Memenuhi Ketentuan | 6 (excl. PANGAN, KEMASAN) | Label **TIDAK complIES**. Pelanggaran ditemukan |
| **VP** | Verifikasi Produk | 6 (excl. OBAT, ROKOK) | Butuh verifikasi tambahan. **BUKAN penolakan** |
| **TMK MAYOR** | TMK Mayor | **PANGAN only** | Pelanggaran mayor (kemasan salah, klaim tak sah) |
| **TMK MINOR** | TMK Minor | **PANGAN only** | Pelanggaran minor (batch code hilang, logo halal salah) |

### Aturan Taksonomi (terverifikasi)
1. MK/TMK = **universal** (semua komoditi)
2. VP = hanya komoditi yang bisa "diverifikasi ulang". **OBAT & ROKOK tidak pernah VP** → keputusan obat biner (lolos/tolak)
3. TMK MAYOR/MINOR = **eksklusif PANGAN** → hanya pangan punya gradasi keparahan

---

## VP — Satu Kode, TIGA Nasib Berbeda (Bahaya Agregasi)

| Komoditi | VP% | Arti VP |
|---|---|---|
| KOSMETIKA | 5,1% | Klaim butuh data dukung (MKL.02). **VP substantif** — "klaim belum terbukti" |
| PRODUK PANGAN | 71,5% | **Limbo permanen** → status tempat sampah (lihat [07_temuan_kritis](07_temuan_kritis.md)) |
| KEMASAN PANGAN | 99,7% | Hampir semua → limbo permanen |
| OT / SUPLEMEN / OBAT KUASI | 1,3-1,5% | Verifikasi minor, jarang |
| OBAT / ROKOK | **0%** | Tidak ada VP (biner) |

> ⚠️ **Bahaya agregasi:** "Total VP" mencampur 3 makna berbeda — "klaim bermasalah" (kosmetik) + "menunggu verifikasi aktif" (OT) + "limbo mati" (pangan). Setiap dashboard yang menjumlahkan VP tanpa split komoditi **menyesatkan**.

### Cross-tab Komoditi → Kesimpulan Pusat (dominan)

| Komoditi | Dominan | Pola |
|---|---|---|
| KOSMETIKA | MK (80,2%) | MK dominan. TMK 14,7%, VP 5,1% |
| PRODUK PANGAN | VP (71,5%) | VP dominan. MK 22,3%. TMK MAYOR 3,1% + TMK MINOR 3,0% |
| OBAT | TMK (90,5%) | TMK dominan. MK 9,5%. **VP = 0** |
| OT | MK (88,2%) | MK dominan. TMK 10,3% |
| ROKOK | MK (62,3%) | Campuran. TMK 37,7%. **VP = 0** |
| SUPLEMEN | MK (93,0%) | Paling patuh |
| OBAT KUASI | MK (92,7%) | |
| KEMASAN PANGAN | VP (99,7%) | VP nyaris total. MK 0,3% |

---

## Matriks Transisi Balai → Pusat (Pusat sebagai Moderator)

Bagaimana keputusan berubah dari balai ke pusat:

| Balai ↓ | Pusat → | n | % | Tafsir |
|---|---|---|---|---|
| MK | MK | 124.467 | 70,3% | Konsensus setuju |
| MK | VP | 47.611 | 26,9% | Pusat ragu → verifikasi (mostly pangan) |
| MK | TMK | 4.893 | 2,8% | Pusat lebih ketat: temukan yg balai lewatkan |
| (kosong) | TMK | 60.068 | — | OBAT/ROKOK: balai TIDAK menilai, pusat putuskan |
| (kosong) | MK | 25.044 | — | |
| TMK | TMK | 11.276 | 79,9% | Konsensus tolak |
| TMK | MK | 1.919 | 13,6% | Pusat "mengampuni" — balai over-reject |
| TMK | VP | 918 | 6,5% | |
| TMK MAYOR | VP | 4.250 | 64,5% | **Pusat turunkan ke verifikasi** |
| TMK MAYOR | TMK MAYOR | 2.266 | 34,4% | Konsensus mayor |
| TMK MINOR | VP | 5.062 | 68,3% | **Pusat turunkan ke verifikasi** |
| TMK MINOR | TMK MINOR | 2.156 | 29,1% | Konsensus minor |

### Tiga Pola Tata Kelola

1. **Pusat = moderator, bukan stempel.** 27% keputusan MK balai diubah pusat. Inter-rater disagreement terukur.
2. **"VP magnet"**: semua TMK Mayor/Minor tersedot ke VP (65-68%). VP = "keranjang penampung keraguan" — pusat enggan langsung menolak.
3. **Pengampunan:** TMK→MK 1.919 kasus (14%) — balai menolak tapi pusat meloloskan.

> **Insight:** Sistem punya **bias ke "verifikasi ulang"** ketimbang "tolak tegas". VP = instrumen politik-birokratis menghindari keputusan tolak final. **Konsekuensi → menumpuk di status 4 (limbo PANGAN).**

---

## Connecting Dots: Rantai Kausal Penyebab Limbo

```
PANGAN (75k sampel) → produsen kosong (form beda)
                   → balai nilai MK (82,9%)
                   → pusat set VP (72%) [instrumen "keraguan"]
                   → masuk antrian MT (status 4)
                   → workflow terputus (tak ada status 5/6/7/999 utk PANGAN)
                   → 76.249 sampel VP MANDEK di status 4 selamanya
```

Ini **satu fenomena tunggal** — utang proses struktural. Bukan kecelakaan, bukan kapasitas — **prosedur VP pangan tidak punya jalur penyelesaian**.

---

## Kode Regulasi di Catatan (terdecode sebagian)

| Kode | n | Komoditi dominan | Arti (perlu konfirmasi BPOM) |
|---|---|---|---|
| MKL.02 | 2.247 | KOSMETIKA | Klaim butuh data dukung |
| MKL.01 | 671 | KOSMETIKA | |
| TME.05 | 127 | ? | |
| TME.09 | 117 | ? | |
| TME.11 | 122 | ? | |
| NIK.01 | 61 | ? | Terkait NIE? |

> Kode lengkap butuh **kamus regulasi eksternal** dari BPOM. Lihat [16_informasi_eksternal](16_informasi_eksternal_dibutuhkan.md).

---

## Koneksi ke Pertanyaan User — Istilah & Disambiguasi

User (export KAI) memakai istilah yang perlu dipetakan ke 3 level verdict:

| Istilah user | Kolom yang dimaksud | n pertanyaan |
|---|---|---|
| "kesimpulan UPT" / "penilaian balai" | `kesimpulan_penilaian_balai` | ~5 |
| "kesimpulan pusat" / "hasil pengawasan akhir dari pusat" | `kesimpulan_penilaian_pusat` | ~10 |
| "kesimpulan akhir" | ⚠️ **TIDAK ada kolom** — user memaksudkan `kesimpulan_penilaian_pusat` | 11 |
| "gap/perbedaan antara pusat dan UPT" | `kesimpulan_balai <> kesimpulan_pusat` | 15 |
| "persamaan antara pusat dan UPT" | `kesimpulan_balai = kesimpulan_pusat` | 1–2 |
| "kesimpulan memenuhi ketentuan / tidak memenuhi ketentuan" | MK / TMK di level pusat (atau balai jika eksplisit) | 27 |

**Aturan disambiguasi kritis:**
1. "Kesimpulan akhir" = `kesimpulan_penilaian_pusat` (bukan kolom `kesimpulan_akhir` yang tidak ada).
2. Jika user menyebut "kesimpulan UPT" → pakai `kesimpulan_penilaian_balai`. Jika "kesimpulan akhir"/"pusat" → pusat.
3. Gap pusat-UPT = `<>`; persamaan = `=` (lihat [21_sql_pairs](21_sql_pairs_penandaan.md) SQL-02).

> Mapping lengkap istilah → kolom: [19_mapping_konsep_ke_schema](19_mapping_konsep_ke_schema.md).
> Gap "jenis kemasan" yang user tanya: [20_gap_schema_user](20_gap_schema_user.md) G1.

---

## Answer Contract untuk Verdict

- Setiap angka verdict **WAJIB** dilabel kode + kepanjangan: "`MK` (Memenuhi Ketentuan)"
- Sebut kolom verdict yang dipakai: balai vs pusat
- VP **WAJIB** split per komoditi (jangan agregat)
- TMK family closure: `{TMK, TMK MAYOR, TMK MINOR}` di balai; tambah `TMK KRITIKAL` di... (tidak ada KRITIKAL di penandaan, hanya di pengawasan)

---

Lanjut ke [07_temuan_kritis.md](07_temuan_kritis.md) untuk evidence lengkap 3 temuan material.
