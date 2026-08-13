# 19 — Mapping Konsep User → Kolom Database

> Jembatan antara istilah yang dipakai user (dari pertanyaan KAI) dan nama kolom sebenarnya di database `penandaan`.
> Tujuan: agent/analis bisa menerjemahkan pertanyaan bahasa bebas → SQL yang benar TANPA menebak kolom.

---

## Pemetaan Utama (Istilah User → Kolom)

| # | Istilah user | Kolom DB | Tabel | Konfidensi | Catatan |
|---|---|---|---|---|---|
| 1 | **UPT** / balai / nama UPT | `nama_balai` | mv_penandaan | ✅ | "UPT" = balai BPOM. User tidak pernah bilang "balai" |
| 2 | **Kesimpulan UPT** / kesimpulan balai / penilaian balai | `kesimpulan_penilaian_balai` | mv_penandaan | ✅ | Level penilaian oleh UPT |
| 3 | **Kesimpulan akhir** / kesimpulan pusat / hasil akhir pusat | `kesimpulan_penilaian_pusat` | mv_penandaan | ✅ | Level final; nilai MK/TMK/VP |
| 4 | **Tanggal proses direktur** / tanggal direktur | `tanggal_kirim_direktur` | mv_penandaan_timeline | ✅ | Dimensi grouping user paling sering (11×) |
| 5 | **Tanggal laporan dikirim ke pusat** / dikirim pusat | `tanggal_kirim_pusat` | mv_penandaan_timeline | ✅ | Marker "sudah dikirim ke pusat" |
| 6 | **Tanggal pemeriksaan** / tanggal mulai | `tgl_start` | mv_penandaan / timeline | ✅ | Awal pengawasan |
| 7 | **Sarana produksi** / industri farmasi / pelaku usaha | `produsen` | mv_penandaan | ⚠️ 75% | Proxy terbaik; perlu konfirmasi apakah = nama sarana |
| 8 | **Pendaftar** / pemohon | `pendaftar` | mv_penandaan | ⚠️ | String corrupt-trap — lihat [11](11_data_quality_anomali.md) |
| 9 | **Nama produk** | `nama_produk` | mv_penandaan | ✅ | Free text |
| 10 | **Komoditi** / kategori produk / kategori | `komoditi` | mv_penandaan | ✅ | 8 nilai, lihat [04](04_komoditi_governing_dimension.md) |
| 11 | **MK** / Memenuhi Ketentuan | `'MK'` (nilai) | — | ✅ | Nilai kolom kesimpulan |
| 12 | **TMK** / Tidak Memenuhi Ketentuan | `'TMK'` (nilai) | — | ✅ | Nilai kolom kesimpulan |
| 13 | **VP** / verifikasi | `'VP'` (nilai) | — | ⚠️ | Multi-makna — lihat [06](06_penilaian_keputusan.md) |
| 14 | **Nomor surat** | `nomor_surat` | mv_penandaan | ✅ | Kode direktorat di segmen — lihat [12](12_kode_berstruktur.md) |
| 15 | **No. sampel** | `nomorsampel` | mv_penandaan | ✅ | Prefix kode balai — lihat [12](12_kode_berstruktur.md) |
| 16 | **NIE / ed_nie** | `ed_nie` | mv_penandaan | ⚠️ | Ada outlier tanggal 1970 — lihat [11](11_data_quality_anomali.md) |
| 17 | **Catatan TMK** | `catatan` | mv_penandaan | ✅ | 62% kosong — lihat [06](06_penilaian_keputusan.md) |
| 18 | **Status** / proses | `status` | mv_penandaan_timeline | ✅ | 999 = selesai |
| 19 | **Tanggal selesai** / tgl end | `tgl_end` | mv_penandaan / timeline | ⚠️ | Hubungan ke durasi belum 100% — lihat [09](09_waktu_durasi.md) |
| 20 | **Pelaku usaha** | `produsen` (proxy) | mv_penandaan | ⚠️ 75% | Sama dgn #7 |
| 21 | **Obat bahan alam** | `komoditi` IN (`OBAT TRADISIONAL (OT)`, dll) | mv_penandaan | ✅ | "Obat Bahan Alam" = OT (Obat Tradisional) |
| 22 | **Label** / hasil label | Baris `mv_penandaan` | mv_penandaan | ⚠️ | User kadang bilang "label" = produk diperiksa |
| 23 | **Timeline** | `mv_penandaan_timeline` | — | ✅ | Tabel terpisah, join via `id_penandaan` |
| 24 | **Pemenuhan timeline** | `tanggal_kirim_pusat - tgl_start` | timeline | ✅ | SLA user: ≤ tanggal 15 bulan berikutnya |
| 25 | **Jumlah laporan** | `COUNT(*)` | mv_penandaan | ✅ | Konteks laporan pengawasan |

---

## Disambiguasi Kritis — 3 Level Kesimpulan

User membedakan **tiga** level yang sering tertukar:

| Level | Istilah user | Kolom | Makna |
|---|---|---|---|
| Level 1 — UPT | "kesimpulan UPT", "penilaian balai" | `kesimpulan_penilaian_balai` | Hasil penilaian oleh balai/UPT |
| Level 2 — Pusat | "kesimpulan pusat", "hasil pengawasan akhir dari pusat" | `kesimpulan_penilaian_pusat` | Keputusan final pusat |
| Level 3 — Akhir | "kesimpulan akhir" | ⚠️ **tidak ada kolom terpisah** | Umumnya user = `kesimpulan_penilaian_pusat` |

> ⚠️ **"Kesimpulan akhir" ≠ kolom khusus.** User memakai "kesimpulan akhir" untuk merujuk hasil final — yang secara data
> adalah `kesimpulan_penilaian_pusat`. Jangan mencari kolom bernama "kesimpulan_akhir".

---

## Disambiguasi "Tanggal Proses Direktur"

User memakai **`tanggal proses direktur`** sebagai dimensi GROUP BY (11×) — ini kolom **`tanggal_kirim_direktur`**
di `mv_penandaan_timeline`, BUKAN kolom tanggal di fact. Wajib join timeline.

```sql
SELECT date_trunc('month', t.tanggal_kirim_direktur)::date AS bulan, count(*)
FROM mv_penandaan p
JOIN mv_penandaan_timeline t ON t.id_penandaan = p.id
WHERE t.tanggal_kirim_direktur IS NOT NULL
GROUP BY 1 ORDER BY 1;
```

---

## Disambiguasi "UPT" = Balai

| Istilah user | Kolom | Konfidensi |
|---|---|---|
| UPT | `nama_balai` | ✅ 100% |
| "masing-masing UPT" | `GROUP BY nama_balai` | ✅ |
| "UPT mana saja" | `nama_balai` + filter kesimpulan | ✅ |
| Balai Besar / Loka POM | Nilai di `nama_balai` (prefiks "BALAI BESAR POM DI..." / "LOKA POM DI...") | ✅ |

> Total 83 nilai aktif. Lihat [10_geografi_kapasitas](10_geografi_kapasitas.md).

---

## Istilah yang TIDAK ADA di Schema (Gap)

| Istilah user | n pertanyaan | Status | Rekomendasi |
|---|---|---|---|
| **Jenis kemasan** / nama kemasan | 3–5 | ❌ tidak ada kolom | Jangan berhalusinasi — jawab "tidak tersedia di DB" |
| **Kabupaten/kota/provinsi produsen** | 2–4 | ❌ tidak ada kolom lokasi | Butuh data eksternal |
| **Media publikasi** | 3 | ❌ domain `pengawasan` | Alihkan ke DB pengawasan |
| **Klaim dalam promosi/iklan** | 3–4 | ❌ domain `pengawasan` | Alihkan ke DB pengawasan |
| **Golongan obat** | 1 | ❌ tidak ada | Jangan tebak |
| **e-labeling / peserta pilot** | 1 | ❌ tidak ada flag | Butuh data eksternal |
| **Sumber pemeriksaan** (pengaduan/rutin) | 0 | ❌ tidak ada | Lihat [16](16_informasi_eksternal_dibutuhkan.md) |

Detail lengkap: [20_gap_schema_user.md](20_gap_schema_user.md).

---

## Aturan Terjemahan (Rules of Thumb)

1. **"dikirim ke pusat"** → filter `tanggal_kirim_pusat IS NOT NULL`.
2. **"dengan kesimpulan X"** → `kesimpulan_penilaian_pusat = 'X'` UNLESS user eksplisit "kesimpulan UPT" → pakai `kesimpulan_penilaian_balai`.
3. **"berdasarkan tanggal proses direktur"** → `GROUP BY date_trunc(tanggal_kirim_direktur)`, bukan tgl_start.
4. **"obat tradisional; suplemen kesehatan; obat kuasi"** → `komoditi IN ('OBAT TRADISIONAL (OT)','SUPLEMEN KESEHATAN','OBAT KUASI')`.
5. **"top N / paling banyak"** → `ORDER BY count DESC LIMIT n`.
6. **"gap/perbedaan pusat vs UPT"** → `WHERE kesimpulan_penilaian_balai <> kesimpulan_penilaian_pusat`.
7. **"persamaan"** → `=` (bukan `<>`).
8. **"tidak input catatan TMK"** → `kesimpulan_penilaian_pusat='TMK' AND (catatan IS NULL OR TRIM(catatan)='')`.
9. **Jangan gunakan kolom yang tidak ada** (jenis kemasan, provinsi, media) — laporkan gap.
10. **"timeline"** selalu butuh join `mv_penandaan_timeline` via `id_penandaan`.

---

## Konvensi Join

```
mv_penandaan p
  JOIN mv_penandaan_timeline t ON t.id_penandaan = p.id   -- tanggal proses, timeline
  JOIN mv_penandaan_log l ON l.id_penandaan = p.id        -- status code, fullname user
```

> `mv_penandaan_timeline` grain 1:1 ke fact (id_penandaan unik). `mv_penandaan_log` 1:many (banyak event per sampel).

---

Lanjut ke [20_gap_schema_user.md](20_gap_schema_user.md) untuk daftar gap lengkap.
