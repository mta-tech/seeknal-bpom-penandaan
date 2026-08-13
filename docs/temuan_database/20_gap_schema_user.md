# 20 — Gap Schema vs Pertanyaan User

> Pertanyaan asli user yang **TIDAK bisa dijawab** dari schema database `penandaan` saat ini — gap schema, domain boundary,
> dan kebutuhan data eksternal. Dokumen ini = anti-halusinasi: agent harus JELAS bilang "tidak tersedia" daripada menebak kolom.

---

## Daftar Gap (Urut Prioritas Berdasarkan Frekuensi User)

### G1. Jenis / Nama Kemasan (3–5 pertanyaan) 🔴

**Contoh pertanyaan user:**
1. "urutkan nama kemasan yang memiliki kesimpulan akhir pengawasan penandaan tmk paling banyak sampai dengan paling rendah."
2. "tampilkan rekapitulasi jumlah laporan pengawasan penandaan untuk masing-masing jenis kemasan dengan hasil kesimpulan akhir yaitu memenuhi ketentuan berdasarkan tanggal proses direktur."
3. "tampilkan rekapitulasi jumlah laporan pengawasan penandaan untuk masing-masing jenis kemasan dengan hasil kesimpulan akhir yaitu tidak memenuhi ketentuan berdasarkan tanggal proses direktur."

**Status:** ❌ **TIDAK ADA kolom** `jenis_kemasan` / `nama_kemasan` di mv_penandaan.

**Yang ADA:** `komoditi = 'KEMASAN PANGAN'` — INI BUKAN jenis kemasan, melainkan komoditi (produk kemasan pangan yang
diperiksa). User yang bertanya "jenis kemasan" memaksudkan atribut kemasan dari produk (botol, blister, sachet, dll) —
data ini tidak direkam.

**Rekomendasi agent:** Jawab "Kolom jenis kemasan tidak tersedia di database e-Penandaan. Data yang ada hanya komoditi
KEMASAN PANGAN (produk kemasan yang diperiksa), bukan atribut kemasan." Jangan menebak.

---

### G2. Kabupaten / Kota / Provinsi Produsen (2–4 pertanyaan) 🔴

**Contoh pertanyaan user:**
1. "Berdasarkan kabupaten kota dan provinsi alamat produsen, tampilkan berapa jumlah label yang dilaporkan MK atau TMK, beserta penilaian produk UPTnya Jika TMK."
2. "Tolong buatkan query untuk: tampilkan data label yang dilaporkan mk/tmk dari masing-masing upt yang dikategorikan berdasarkan nama produk, jenis pangan, kategori pangan, produsen, kabupaten/provinsi produsen, pada rentang waktu antara tanggal 1 januari 2025 hingga 31 desember 2025."

**Status:** ❌ **TIDAK ADA kolom lokasi produsen** (alamat/kabupaten/provinsi) di schema.

**Yang ADA:** `produsen` = nama (free text). Lokasi alamat produsen tidak direkam.

**Rekomendasi agent:** Bisa answer per `produsen`, tapi tidak per kabupaten/provinsi. Butuh data eksternal master
produsen → alamat.

---

### G3. Media Publikasi / Iklan / Klaim Promosi (7 pertanyaan) 🔴

**Contoh pertanyaan user:**
1. "Tampilkan data iklan yang dilaporkan MK/TMK dari masing-masing UPT yang dikategorikan berdasarkan nama produk, Sarana Produksi, media publikasi, dan klaim dalam promosi/iklan dengan status selesai pada rentang waktu 1 tahun"
2. "Tampilkan data iklan mk/tmk yang dilaporkan dengan status selesai dari 25 Juni 2024 hingga 31 Juni 2024 berdasarkan media publikasi."
3. "Tampilkan data iklan mk/tmk yang dilaporkan dengan status selesai ... berdasarkan nama produk dan golongan obat."

**Status:** ❌ **Domain boundary** — data iklan/media publikasi/klaim ada di database **`pengawasan`**, BUKAN `penandaan`.

**Mengapa ini muncul di db_alias penandaan:** Banyak pertanyaan iklan di-export KAI ke db_alias `penandaan` karena
konfigurasi alias, padahal secara domain termasuk pengawasan iklan/promosi.

**Rekomendasi agent:** 
- Jika pertanyaan eksplisit soal iklan/media/klaim/golongan obat → arahkan ke database `pengawasan`.
- Di `penandaan`, yang bisa dijawab: MK/TMK produk (bukan iklan), nama produk, sarana produksi.
- Jangan paksakan jawab dengan data penandaan.

---

### G4. Golongan Obat (1 pertanyaan) 🟠

**Contoh:** "Tampilkan data iklan mk/tmk yang dilaporkan dengan status selesai ... berdasarkan nama produk dan golongan obat."

**Status:** ❌ Tidak ada kolom "golongan obat" (bebas/keras/terbatas). Ada `komoditi` (OBAT vs OT) dan `nomor_surat` prefix
yang bisa menunjukkan jenis, tapi bukan golongan obat farmakologis. Lihat [12](12_kode_berstruktur.md).

---

### G5. e-Labeling / Peserta Pilot (1 pertanyaan) 🟠

**Contoh:** "Tampilkan data obat hasil pengawasan penandaan yang merupakan peserta pilot project e-labeling tahun 2025"

**Status:** ❌ Tidak ada flag/kolom e-labeling. Data pilot di luar DB (sistem terpisah).

---

### G6. Kesimpulan "Akhir" Terpisah (implisit) 🟠

**Contoh:** "hasil kesimpulan akhir" — user mengira ada kolom "kesimpulan akhir".

**Status:** ⚠️ Tidak ada kolom terpisah; "akhir" = `kesimpulan_penilaian_pusat`. Jelaskan mapping ini (lihat
[19](19_mapping_konsep_ke_schema.md)).

---

### G7. Pemicu / Sumber Pemeriksaan (implisit, 0 langsung) ⚪

Tidak ada field "pengaduan vs rutin". Lihat [16](16_informasi_eksternal_dibutuhkan.md) §10.

---

## Tabel Rekap Gap

| Gap | n pertanyaan | Kolom ada? | Solusi |
|---|---|---|---|
| G1 Jenis/nama kemasan | 3–5 | ❌ | Laporkan tidak tersedia; data KEMASAN PANGAN ≠ jenis kemasan |
| G2 Kabupaten/provinsi produsen | 2–4 | ❌ | Jawab per `produsen`; butuh master alamat eksternal |
| G3 Iklan/media/klaim | 7 | ❌ domain pengawasan | Arahkan ke DB `pengawasan` |
| G4 Golongan obat | 1 | ❌ | Tidak tersedia |
| G5 e-labeling | 1 | ❌ | Data eksternal |
| G6 "Kesimpulan akhir" | implisit | ⚠️ = kesimpulan_pusat | Mapping eksplisit |
| G7 Pemicu pemeriksaan | 0 | ❌ | Data eksternal |

---

## Prinsip Anti-Halusinasi

1. **Jika kolom tidak ada → KATAKAN "tidak tersedia"** dengan penjelasan, JANGAN tebak pakai kolom lain yang salah makna.
2. **Jika domain salah (iklan→pengawasan) → arahkan ke DB yang benar**, jangan paksakan.
3. **Sediakan alternatif terdekat** yang valid (mis. per `produsen` saat kabupaten tidak ada).
4. **Bedakan komoditi vs atribut**: `KEMASAN PANGAN` (komoditi) ≠ "jenis kemasan" (atribut).

---

## Yang BISA Dijawab (Kontrol Positif)

| Pertanyaan user | Solusi |
|---|---|
| Jumlah penandaan per tahun/bulan | ✅ `COUNT(*)` + filter tgl_start |
| Gap pusat vs UPT | ✅ `kesimpulan_balai <> kesimpulan_pusat` |
| Rekap per UPT | ✅ `GROUP BY nama_balai` |
| Rekap per tanggal direktur | ✅ `GROUP BY date_trunc(tanggal_kirim_direktur)` |
| Ranking UPT/produsen | ✅ `GROUP BY ... ORDER BY count DESC` |
| MK/TMK per komoditi | ✅ `WHERE kesimpulan_pusat IN ('MK','TMK')` |
| UPT tak input catatan TMK | ✅ `catatan IS NULL/empty AND kesimpulan='TMK'` |
| Timeline pemenuhan | ✅ `tanggal_kirim_pusat - tgl_start` |
| Triplet OT/Supkes/Kuasi | ✅ `komoditi IN (...)` |
| Raw export | ✅ `SELECT * LIMIT n` |

Lihat [21_sql_pairs_penandaan.md](21_sql_pairs_penandaan.md) untuk SQL tervalidasi.
