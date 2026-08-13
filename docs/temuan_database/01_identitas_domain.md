# 01 — Identitas Domain

> Sistem apa ini? Bagaimana kita tahu?

---

## Kesimpulan Domain

Database ini = **sistem e-Penandaan BPOM** — aplikasi alur kerja untuk memeriksa apakah **label/penandaan produk** yang beredar sesuai regulasi. Setiap sampel produk diambil balai daerah, dinilai, lalu diverifikasi pusat.

Ini BUKAN:
- ❌ Registrasi produk pangan (itu di `seeknal-bpom-neo`, tabel `t_produk_3_*`)
- ❌ Pengawasan iklan (itu di `seeknal-bpom-pengawasan`, tabel `mv_pengawasan*`)
- ❌ Pemeriksaan/pengujian lab (domain terpisah)

Ini ADALAH pemeriksaan **label/kemasan/penandaan** produk yang sudah beredar — apakah sesuai izin edar.

---

## Bukti Petunjuk (kepingan → kesimpulan)

| Petunjuk (dari data) | Kesimpulan |
|---|---|
| 83 `nama_balai` ("BALAI BESAR POM DI...", "BALAI POM DI...", "LOKA POM DI...") | Institusi BPOM daerah |
| 8 komoditi: KOSMETIKA, OBAT, PRODUK PANGAN, ROKOK, OT, SUPLEMEN, OBAT KUASI, KEMASAN PANGAN | Ranah pengawasan BPOM |
| Kolom `ed_nie` (Expiry Date NIE) | NIE = Nomor Izin Edar — izin resmi BPOM |
| Kesimpulan = MK/TMK/VP | MK = Memenuhi Ketentuan, TMK = Tidak Memenuhi Ketentuan, VP = Verifikasi Produk |
| Catatan: "2D barcode", "klaim data dukung", "MKL.02", "halal", "ING" | Pemeriksaan label/kemasan |
| Workflow: Operator → Supervisor → Kepala Balai → Pusat → Deputi → Penyelia → Penguji → Selesai | Struktur birokrasi berjenjang balai → pusat |
| 5 Direktorat (KMEI ONPPZA, Kosmetik, OTSK, Peredaran Pangan Olahan, Distribusi ONPP) | Direktorat teknis di BPOM Pusat Jakarta |
| `nomor_surat` prefix `PW.dd.bb...` (PW = Penandaan) | Penomoran surat berkode direktorat |

---

## Model Mental Singkat

```
SEBUAH PRODUK BEREDAR
  │
  ├─ Balai daerah ambil sampel ──► nomorsampel (berkode tahun+balai)
  │                                 nomor_surat (berkode direktorat+bidang)
  │
  ├─ [FORM per komoditi BERBEDA]
  │    KOSMETIKA: tidak isi pendaftar
  │    PANGAN/KEMASAN: tidak isi produsen
  │    ROKOK: 70% tidak isi ed_nie
  │
  ├─ WORKFLOW (log mencatat tiap langkah, timeline hitung durasi)
  │    Operator → Spv1 → [reject 36k di sini]
  │            → KepalaBalai → Pusat/MT → Deputi → Penyelia → Penguji → Selesai
  │
  ├─ PENILAIAN
  │    OBAT/ROKOK: balai SKIP → pusat nilai langsung (→ 90% TMK, standar ketat)
  │    KOSMETIKA/PANGAN/dll: balai nilai → pusat verifikasi (27% diubah)
  │
  ├─ KEPUTUSAN (kesimpulan_pusat)
  │    MK (lolos) | TMK (tolak) | VP (verifikasi) | TMK MAYOR/MINOR (khusus pangan)
  │    ⚠ VP pangan = limbo → 76k sampel stuck selamanya di MT
  │
  └─ HASIL tersimpan di FACT, diringkas di AGG
```

---

## Lingkup Cakupan Database

**Yang TERCATAT detail:**
- Proses administratif penandaan (draft → approval)
- Keputusan penilaian (balai, pusat, akhir)
- Audit trail (siapa-kapan-catatan)
- Durasi per tahap workflow
- Target & realisasi 2024

**Yang TIDAK tercatat (batas — lihat [16_informasi_eksternal](16_informasi_eksternal_dibutuhkan.md)):**
- Hasil uji laboratorium (status 7 ada, hasil tak tersimpan)
- Tindak lanjut TMK (tarik produk? peringatan?)
- Pemicu pemeriksaan (pengaduan vs rutin)
- Geolokasi produk (kabupaten ditemukan beredar)
- Kamus kode regulasi (MKL.02, TME.09 — arti di Peraturan BPOM)
- Target SLA per tahap
- Target 2023/2025/2026 (hanya 2024 ada)

---

Lanjut ke [02_arsitektur_database.md](02_arsitektur_database.md) untuk peta tabel & relasi.
