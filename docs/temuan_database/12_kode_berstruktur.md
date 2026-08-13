# 12 — Kode Berstruktur (Data dalam Data)

> Decode nomor_surat & nomorsampel — kode tersembunyi yang bermakna.

---

## 12.1 `nomor_surat` — Kode Direktorat Terdecode

### Struktur
```
PW.{direktorat}.{bidang}.{...}.{seq}.{bulan}.{tahun}
 ^    ^           ^
 |    |           bidang/sub-direktorat
 |    direktorat (2-digit)
 prefix "Penandaan"
```

### Decode direktorat → komoditi (terverifikasi)

| Kode direktorat | Komoditi dominan | n surat |
|---|---|---|
| **PW.01** | **OBAT** | 40.567 |
| **PW.02** | **OBAT TRADISIONAL (OT)** | 17.903 |
| **PW.03** | **KOSMETIKA** | 44.175 |
| **PW.04** | **PRODUK PANGAN** | 55.010 |

> **Kode direktorat tersembunyi di nomor surat = terexpos.** PW.04 (pangan) paling banyak mengeluarkan surat (55k), padahal direktorat pangan **0 completion** — direktorat paling produktif mengeluarkan surat TAPI paling tidak produktif menyelesaikan. Ironi struktural.

**Query verifikasi:**
```sql
WITH sur AS (
  SELECT split_part(nomor_surat,'.',2) AS dir, komoditi, count(*) AS n
  FROM mv_penandaan WHERE nomor_surat ~ '^PW\.[0-9]+\.[0-9]+' AND komoditi<>''
  GROUP BY 1,2
)
SELECT dir, komoditi, n FROM (
  SELECT dir, komoditi, n, row_number() OVER (PARTITION BY dir ORDER BY n DESC) AS rk FROM sur
) x WHERE rk=1 ORDER BY n DESC;
```

### Distribusi prefix
| Prefix | n | % |
|---|---|---|
| PW.01 | 101.430 | 38,6% |
| PW.04 | 69.302 | 26,4% |
| PW.03 | 63.217 | 24,1% |
| PW.02 | 29.203 | 11,1% |

> Sisanya: variasi ketik (PW.0110, PW.0403, dll) — minor, perlu normalisasi input.

---

## 12.2 Surat Besar vs Kecil (Bimodal)

| Metrik | Angka |
|---|---|
| Total surat non-empty | ~22.443 |
| Avg sampel/surat | 13 |
| Surat dengan 1 sampel (kasus individual) | 4.243 |
| Surat placeholder `-` (draft) | 7.858 |
| Surat terbesar nyata | 572 sampel |

### Surat terbesar nyata (top 5)
| nomor_surat | n | Dari | Sampai |
|---|---|---|---|
| PW.03.12.10A.01.24.36 | 572 | 2024-01-16 | 2024-12-04 |
| PW.03.12.14A.14A2.01.23.14 | 469 | 2023-01-17 | 2023-12-04 |
| PW.03.12.14A.14A2.01.23.13 | 387 | 2023-01-13 | 2024-03-07 |
| PW.01.12.10A.01.24.03 | 372 | 2024-01-16 | 2024-12-02 |
| PW.04.03.14A.14A2.01.23.0076 | 342 | 2023-02-08 | 2023-12-04 |

> **Surat terbesar spanning Jan-Des 2024** (hampir 1 tahun penuh). Ini bukan operasi tunggal — "surat tahunan" / batch tematis. Bukan razia serentak.

**Implikasi:** 292k sampel BUKAN 292k peristiwa independen. Ini ~22k operasi dengan skala sangat bervariasi (1-572 sampel/surat). Volume = fungsi dari kampanye bertema.

---

## 12.3 `nomorsampel` — Kode Balai Tersembunyi

### Struktur
```
yy.bbb.ttt.xx.xx.seq.KK
 ^   ^
 |   kode balai (3-digit)
 tahun (2-digit)
```

### Validasi kode balai (84% andal)

| Kategori | n prefix | % |
|---|---|---|
| Konsisten (1 prefix → 1 balai) | 61 | **84%** |
| Ambigu (1 prefix → >1 balai) | 12 | 16% |

### Redundansi terstruktur (aset kualitas data)
**Balai tersimpan di DUA tempat:**
1. Kolom `nama_balai` (teks)
2. Embedded di `nomorsampel` (kode 3-digit)

> **Validasi silang gratis:** bila kode balai di nomorsampel ≠ nama_balai, ada kesalahan input. Ini aset kualitas data yang belum dimanfaatkan. Lihat [17_rencana_investigasi](17_rencana_investigasi_lanjut.md) §K4.

### Prefix ambigu (12 kode, identitas ragu)
| Kode | n_balai | n_sampel | Hipotesis |
|---|---|---|---|
| 094 | 3 | 9.297 | Reorganisasi balai |
| 119 | 3 | 5.844 | Kode dipakai ulang |
| 093, 098, 100, 102, 104, 108 | 2 | 5.000-10.000 | |

---

## 12.4 `id` — PK Sempurna tapi Tidak Di-enforce

| Metrik | Angka |
|---|---|
| Baris fact | 292.598 |
| `id` distinct | 292.598 |
| Null | 0 |
| Duplikat | 0 |

**PK sempurna** tapi tidak dideklarasikan di schema → sistem sumber (aplikasi) yang menjamin, bukan DB constraint.

### Ketimpangan dengan log/timeline
| Tabel | distinct id |
|---|---|
| fact | 292.598 |
| log | 500.717 |
| timeline | 500.717 |
| Selisih (orphan) | **208.119 (41,6%)** |

> Hampir separuh riwayat proses tak punya induk fact. Populasi proses sesungguhnya = **500.717**, bukan 292.598.

---

## Implikasi

1. **nomor_surat prefix PW.XX = direktorat** — bisa decode birokrasi tanpa kolom direktorat
2. **nomorsampel prefix = kode balai** — validasi silang 84% andal
3. **Surat bimodal** — kasus individual (1 sampel) vs surat tahunan (hingga 572)
4. **id = PK logis** tapi schema tak enforce — andalkan aplikasi sumber
5. **Populasi proses 500k** (bukan 292k fact) — selalu klarifikasi scope

---

Lanjut ke [13_konsistensi_integritas.md](13_konsistensi_integritas.md).
