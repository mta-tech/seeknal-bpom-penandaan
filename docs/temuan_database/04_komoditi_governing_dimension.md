# 04 — Komoditi: Governing Dimension

> Komoditi bukan filter — ia adalah **variabel penentu** yang mengendalikan 5 hal sekaligus.

---

## 8 Komoditi & Kepribadian Masing-masing

| komoditi | n | MK% | TMK% | VP% | Completion | Karakter tersembunyi |
|---|---|---|---|---|---|---|
| KOSMETIKA | 78.495 | 80,2 | 14,7 | 5,1 | 95,5% | pendaftar 100% null; VP=klaim (MKL.02) |
| PRODUK PANGAN | 75.560 | 22,3 | 0 | 71,5 | **0,0%** | produsen 100% null; VP=limbo; Mayor/Minor |
| OBAT | 53.163 | 9,5 | 90,5 | 0 | 98,6% | balai 100% skip; biner; tanpa VP |
| OBAT TRADISIONAL (OT) | 39.170 | 88,2 | 10,3 | 1,5 | 96,5% | paling patuh setelah suplemen |
| ROKOK | 32.082 | 62,3 | 37,7 | 0 | 96,7% | ed_nie 70% null; balai skip; tanpa VP |
| SUPLEMEN KESEHATAN | 10.752 | 93,0 | 5,8 | 1,3 | 97,5% | paling patuh |
| OBAT KUASI | 2.687 | 92,7 | 5,8 | 1,5 | 97,0% | mirip suplemen |
| KEMASAN PANGAN | 689 | 0,3 | 0 | 99,7 | **0,0%** | hampir 100% VP; dead-end sama PANGAN |

---

## Komoditi Mengendalikan 5 Hal (Bukti Deterministik)

### (a) Form input — null deterministik per komoditi

| komoditi | kesimpulan_balai null | pendaftar null | produsen null | ed_nie null | catatan null |
|---|---|---|---|---|---|
| OBAT | **100%** | 3,7% | 5,1% | 4,2% | **100%** |
| ROKOK | **100%** | 1,4% | 0,1% | **69,8%** | **100%** |
| KOSMETIKA | 0% | **100%** | 1,3% | 0,8% | 2,3% |
| PRODUK PANGAN | 0% | 34,8% | **100%** | 4,1% | 71,6% |
| KEMASAN PANGAN | 0% | 29,3% | **100%** | 16,3% | 99,7% |
| OT | 0% | 12,1% | 0,6% | 1,6% | 71,7% |
| SUPLEMEN | 0% | 33,7% | 1,7% | 1,1% | 75,4% |
| OBAT KUASI | 0% | 20,8% | 1,0% | 1,3% | 81,9% |

**Pola bersih:**
- OBAT & ROKOK: `kesimpulan_balai` + `catatan` 100% null → **balai tidak menyentuh sama sekali**
- KOSMETIKA: `pendaftar` 100% null → form kosmetik tidak punya field pendaftar
- PANGAN & KEMASAN PANGAN: `produsen` 100% null → form pangan tidak punya field produsen
- ROKOK: `ed_nie` 69,8% null → rokok tidak pakai skema NIE yang sama

> **Setiap null = cetak biru form, BUKAN data hilang.** Membersihkan null = merusak informasi. Setiap komoditi punya sub-form berbeda.

### (b) Siapa menilai — balai vs pusat

- **OBAT & ROKOK:** balai SKIP (kb 100% null) → pusat nilai langsung → Direktorat KMEI ONPPZA
- **Lainnya:** balai nilai dulu → pusat verifikasi

### (c) Taksonomi keputusan

| Komoditi | Kode kesimpulan yang berlaku |
|---|---|
| OBAT, ROKOK | biner: MK / TMK saja (tidak ada VP) |
| KOSMETIKA, OT, SUPLEMEN, OBAT KUASI | MK / TMK / VP |
| PRODUK PANGAN, KEMASAN PANGAN | MK / VP / TMK MAYOR / TMK MINOR (TMK biasa = 0!) |

### (d) Completion (pola dead-end PANGAN)

- PANGAN & KEMASAN: **0% completion** (lihat [07_temuan_kritis](07_temuan_kritis.md))
- Lainnya: 95-99%

### (e) Direktorat pemroses

| Direktorat | Komoditi rumah |
|---|---|
| KMEI ONPPZA | OBAT, ROKOK (590.086 event, 146.822 selesai) |
| Kosmetik | KOSMETIKA + 4 lain (372.752 event, 33.890 selesai) |
| OTSK | lintas (routing, 243.923 event) |
| Peredaran Pangan Olahan | PRODUK PANGAN (2.519 event, **0 selesai**) |
| Distribusi dan Pelayanan ONPP | (1.928 event, 0 selesai) |

---

## Sumbu Kepatuhan (MK%)

```
SUPLEMEN      93,0%  ████████████████████████████████▏
OBAT KUASI    92,7%  ████████████████████████████████▏
OT            88,2%  ██████████████████████████████▏
KOSMETIKA     80,2%  ██████████████████████████▏
ROKOK         62,3%  █████████████████████▏
PANGAN        22,3%  ███████▏
OBAT           9,5%  ███▏
KEMASAN        0,3%  ▏
```

**Dua kutub ekstrem:** OBAT (9,5% MK) vs SUPLEMEN (93% MK) — selisih 10× lipat.

⚠️ **TAPI ini BUKAN berarti obat lebih nakal.** OBAT dinilai pusat langsung (standar ketat), SUPLEMEN dinilai balai (lebih longgar). **MK% tergantung SIAPA yang menilai**, bukan kualitas produk. Lihat [06_penilaian_keputusan](06_penilaian_keputusan.md).

---

## Sumbu Keputusan: Biner vs Bergradasi

- **Biner (MK/TMK saja):** OBAT, ROKOK → tidak ada jalan tengah, keputusan tegas
- **Bergradasi (punya VP):** KOSMETIKA, OT, SUPLEMEN, OBAT KUASI → ada verifikasi
- **Paling bergradasi:** PANGAN (5 kategori: MK/VP/MAYOR/MINOR + TMK biasa = 0!)

---

## Koneksi ke Pertanyaan User

User (dari export KAI) sering bertanya berdasarkan komoditi — pola ini wajib dipetakan benar:

| Istilah user | Mapping komoditi | Catatan |
|---|---|---|
| "obat tradisional; suplemen kesehatan; obat kuasi" (triplet, 6 pertanyaan) | `IN ('OBAT TRADISIONAL (OT)','SUPLEMEN KESEHATAN','OBAT KUASI')` | Triplet khas diminta BERSAMA |
| "obat bahan alam" (2 pertanyaan) | `OBAT TRADISIONAL (OT)` | "Obat Bahan Alam" = OT dalam tata bahasa BPOM |
| "pangan olahan" / "label pangan" (3 pertanyaan) | `PRODUK PANGAN` | ⚠️ PANGAN 0% completion — hati-hati saat lapor |
| "kosmetik" | `KOSMETIKA` | — |

> **Konsep "UPT" = `nama_balai`** (lihat [19_mapping_konsep_ke_schema](19_mapping_konsep_ke_schema.md)) — user
> memakai istilah "UPT", database memakai "balai". Jangan salah terjemah saat filter per wilayah.

> ⚠️ **Gap schema user:** "jenis/nama kemasan" (3–5 pertanyaan) ≠ komoditi `KEMASAN PANGAN`. User memaksudkan atribut
> kemasan produk (botol/blister/sachet) yang TIDAK ada kolomnya. Lihat [20_gap_schema_user](20_gap_schema_user.md) G1.

---

## Connecting Dots: Mengapa Komoditi Penting

Komoditi adalah **governing dimension** — variabel penentu yang mengontrol:
1. Form input mana yang dipakai (null deterministik)
2. Apakah balai menilai atau skip
3. Taksonomi keputusan mana yang berlaku
4. Apakah sampel bisa selesai (PANGAN = dead-end)
5. Direktorat pusat mana yang menangani

> **Setiap analisis lintas-komoditi HARUS mengontrol komoditi.** Membandingkan MK% OBAT vs SUPLEMEN tanpa kontrol penilai = bias. Membandingkan completion PANGAN vs OBAT tanpa konteks dead-end = menyesatkan.

---

Lanjut ke [05_workflow_state_machine.md](05_workflow_state_machine.md).

---

## Populasi yang Sah Dibandingkan per Komoditi (verifikasi 2026-08-13)

Tabel di §atas sudah mencatat "OBAT & ROKOK: balai 100% skip". Konsekuensi analitiknya perlu
dinyatakan sebagai angka, karena **pertanyaan tersering domain ini adalah pertanyaan gap**
(17× di log KAI: *"data gap hasil pengawasan penandaan obat antara pusat dan UPT"*).

Gap hanya terdefinisi bila **balai sudah menilai** DAN **pusat sudah berkeputusan** (bukan `VP`,
yang merupakan tahap menunggu — lihat `06_penilaian_keputusan.md`):

```sql
SELECT komoditi, count(*) AS n,
       count(*) FILTER (WHERE kesimpulan_penilaian_balai <> ''
                          AND kesimpulan_penilaian_pusat NOT IN ('','VP')) AS bisa_dibandingkan,
       count(*) FILTER (WHERE kesimpulan_penilaian_balai <> ''
                          AND kesimpulan_penilaian_pusat NOT IN ('','VP')
                          AND kesimpulan_penilaian_balai <> kesimpulan_penilaian_pusat) AS beda
FROM mv_penandaan GROUP BY 1 ORDER BY 3 DESC;
```

| Komoditi | Baris | Bisa dibandingkan | Benar-benar beda | % beda dari yang dibandingkan |
|---|--:|--:|--:|--:|
| KOSMETIKA | 78.550 | 74.436 | **6.336** | 8,5% |
| OBAT TRADISIONAL (OT) | 39.189 | 38.537 | 643 | 1,7% |
| PRODUK PANGAN | 75.629 | 21.443 | 374 | 1,7% |
| SUPLEMEN KESEHATAN | 10.757 | 10.607 | 109 | 1,0% |
| OBAT KUASI | 2.688 | 2.646 | 28 | 1,1% |
| KEMASAN PANGAN | 689 | 2 | 0 | — |
| **OBAT** | 53.175 | **0** | **0** | **tidak terdefinisi** |
| **ROKOK** | 32.081 | **0** | **0** | **tidak terdefinisi** |

Tiga hal yang langsung terbaca:

1. **OBAT dan ROKOK: gap tidak terdefinisi.** Bukan "nol gap" — memang tidak ada penilaian balai
   untuk dibandingkan. Pertanyaan gap untuk OBAT harus dijawab dengan menyatakan batas ini, bukan
   dengan angka.
2. **PRODUK PANGAN kehilangan 71%** populasinya karena `VP` (54.186 dari 75.629). Angka gap 374
   berlaku atas 21.443 baris yang sudah berkeputusan, bukan atas seluruh pangan.
3. **KOSMETIKA adalah satu-satunya komoditi dengan gap material** (6.336 baris, 8,5%). Kalau user
   bertanya "gap penandaan" tanpa menyebut komoditi, inilah yang mendominasi jawabannya — dan itu
   harus disebut, bukan disajikan sebagai angka nasional.

> ⚠️ **Jebakan `IS NOT NULL`.** Sentinel di kedua kolom kesimpulan adalah **string kosong `''`**,
> bukan SQL NULL. `WHERE kesimpulan_penilaian_balai IS NOT NULL` **tidak menyaring apa pun** — dan
> karena `'' <> 'TMK'` bernilai TRUE, SQL yang memakai pola itu melaporkan seluruh 48.122 baris
> OBAT ber-`pusat='TMK'` sebagai "gap". Bukti lengkap: `21_sql_pairs_penandaan.md` §21.D.
