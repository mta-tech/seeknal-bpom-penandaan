# 15 — Gap: Yang Belum Ditemukan

> Open questions, aspek belum didetailkan, hipotesis tertunda, batas cakupan.

---

## A. Hipotesis Terverifikasi ✅ (sudut jelas)

| # | Hipotesis | Status |
|---|---|---|
| 1 | Null deterministik per komoditi (form berbeda) | ✅ Terverifikasi penuh |
| 2 | VP-limbo permanen (PANGAN dead-end) | ✅ Terverifikasi penuh |
| 3 | Key-person dependency ekstrem (3 approver = 72%) | ✅ Terverifikasi penuh |
| 4 | 41,6% produk identik beda keputusan lintas balai | ✅ Terverifikasi penuh |
| 5 | direktur_pusat = hari (bukan flag) | ✅ Terverifikasi — koreksi asumsi awal |
| 6 | Target 2024 dapat di-join (bukan "KPI buta") | ✅ Terverifikasi — koreksi asumsi awal |
| 7 | trx_steps = edge, status_code = node | ✅ Terverifikasi |
| 8 | Kode PW.XX di nomor_surat = direktorat | ✅ Terverifikasi |

---

## B. Hipotesis Tertunda 🟡 (perlu query lanjut)

### B1. Dari mana kolom durasi dihitung?
- **Pengamatan:** `mulai_kabalai` cocok `tanggal_kirim_kabalai - tgl_start` hanya 4,5%
- **Hipotesis:** Business days (exclude weekend) ATAU anchor event berbeda
- **Query verifikasi:** [17_rencana_investigasi](17_rencana_investigasi_lanjut.md) §K5

### B2. Apakah sampel VP PANGAN pernah bergerak di log setelah status 4?
- **Pengamatan:** Status 4 PANGAN median umur 1.128 hari
- **Hipotesis:** Benar-benar abandoned (event terakhir juga lama)
- **Query:** §K2 (last event tanggal_proses untuk status-4)

### B3. Pola disagreement produk identik — arah bias mana?
- **Pengamatan:** 5.277 produk beda keputusan, 99,3% lintas balai
- **Hipotesis:** Transisi dominan MK→TMK atau TMK→MK? Korelasi waktu (regulasi berubah)?
- **Query:** §K4 (transition pattern + time span)

### B4. Apakah anomali Q1 2025 sistemik di SEMUA direktorat atau sebagian?
- **Pengamatan:** 76 balai aktif tapi volume anjlok
- **Hipotesis:** Migrasi sistem (semua direktorat) vs masalah lokal
- **Query:** §K11 (per direktorat per bulan)

### B5. Apakah kolom durasi benar = business days?
- Belum diverifikasi — perlu generate_series hitung hari kerja

### B6. Profil 18.666 orphan status-4 tahun 2021 — komoditinya apa?
- Orphan tidak punya komoditi di timeline. Bisa rekonstruksi dari log?
- **Query:** §K7

### B7. Apakah 12 prefix kode balai ambigu = reorganisasi atau salah input?
- **Hipotesis:** Kode dipakai ulang saat balai direorganisasi
- Perlu data eksternal master balai untuk konfirmasi

### B8. Konsistensi agg di level detail (per balai × bulan)?
- Grand total cocok, tapi apakah per cell juga cocok?
- **Query:** §K9

---

## C. Aspek yang BELUM Didetailkan

### C1. Analisis produsen sebagai aktor perilaku
- Top produsen (Bernofarm, DEXA, Kimia Farma — semua OBAT, 90%+ TMK)
- **TAPI TMK% confounded komoditi** (OBAT dinilai ketat pusat)
- Belum dianalisis: produsen yang TMK-nya turun lintas tahun (efek jera) vs residivis
- Belum kontrol: produsen × komoditi × siapa-menilai

### C2. Nama produk free-text analysis
- 84.693 produk unik, free text
- Belum dianalisis: produk paling sering diperiksa, pola klaim di nama (mis. "halal", "herbal")

### C3. Halal / ING / 2D barcode claims
- Catatan sebut "2D barcode", "halal", "ING" — tapi belum diquantify per klaim
- **Query potensial:** ILIKE pattern di catatan

### C4. Tren multi-tahun per komoditi (completion rate)
- PANGAN 0% setiap tahun — konstan atau ada periode pernah selesai?
- **Query:** §K1 (PANGAN per tahun)

### C5. Beban approver per direktorat
- 22 approver final distribusinya per direktorat apa?
- Apakah direktorat Kosmetik lebih timpang dari KMEI?

### C6. Pendaftar cleansed count
- 4.314 distinct pendaftar raw, tapi corrupt-string trap
- Belum ada normalization mapping → belum ada "pendaftar unik cleansed"

### C7. Hubungan komoditi × media pengiriman × durasi
- Apakah komoditi tertentu (volume besar) punya durasi lebih lambat karena antrian?

### C8. Tindak lanjut TMK — TIDAK ADA di DB
- Setelah TMK, produk ditarik? Diperingatkan? Tidak terekam → batas cakupan

---

## D. Batas Cakupan Database (yang TIDAK bisa dijawab dari data)

Lihat [16_informasi_eksternal](16_informasi_eksternal_dibutuhkan.md) untuk detail. Ringkasan:

1. **Hasil uji lab** — status 7 ada, hasil tak tersimpan
2. **Tindak lanjut TMK** — tarik? peringatan? tidak terekam
3. **Pemicu pemeriksaan** — pengaduan vs rutin, tak ada field
4. **Geolokasi produk** — kabupaten ditemukan beredar, tak terekam
5. **Kamus kode regulasi** (MKL.02, TME.09) — kode ada, arti di Peraturan BPOM
6. **Target SLA per tahap** — bisa ukur aktual, tak tahu target tanpa SOP
7. **Target multi-tahun** — hanya 2024
8. **SOP workflow PANGAN** — kenapa pipa terputus? Di luar data

---

## E. Pertanyaan Investigatif Lanjut (prioritas)

### E1. Mengapa PANGAN terputus di MT? (root cause di aplikasi/SOP)
- Data tunjukkan GEJALA (0 completion). Penyebab di SOP/aplikasi BPOM.
- **Butuh:** akses aplikasi e-Penandaan ATAU wawancara tim direktorat pangan

### E2. Berapa biaya waktu sesungguhnya dari rework?
- Reject spv_1 tidak terlihat di mulai_kabalai
- Perlu ukur `tgl_end - tgl_start` untuk sampel pernah-ditolak vs lolos
- **Query:** §K6

### E3. Bisakah orphan 2021 di-backfill ke fact?
- 18.666 orphan status-4 2021 = backlog pra-fact
- Bisa direkonstruksi dari log? Perlu keputusan tim data

### E4. Apakah completion PANGAN bisa "dipaksa selesai" via batch update?
- 76.249 sampel stuck — apakah ada SOP untuk closure manual?
- **Butuh:** kebijakan BPOM (bukan data)

---

## G. Gap dari Perspektif Pertanyaan User (baru)

Validasi terhadap **101 pertanyaan user asli** mengungkap gap yang user tanyakan tapi DB tidak punya:

| # | Gap | n pertanyaan | Kolom DB? | Solusi |
|---|---|---|---|---|
| G1 | **Jenis / nama kemasan** (atribut botol/blister/sachet) | 3–5 | ❌ tidak ada | Laporkan tidak tersedia; bedakan dari komoditi KEMASAN PANGAN |
| G2 | **Kabupaten/kota/provinsi produsen** | 2–4 | ❌ tidak ada kolom lokasi | Jawab per `produsen`; butuh master alamat eksternal |
| G3 | **Media publikasi / iklan / klaim promosi / golongan obat** | 7 | ❌ domain `pengawasan` | Arahkan ke DB pengawasan |
| G4 | **e-labeling / peserta pilot** | 1 | ❌ tidak ada flag | Data eksternal |
| G5 | **"Kesimpulan akhir"** dianggap kolom | implisit | ⚠️ = `kesimpulan_penilaian_pusat` | Mapping eksplisit |

> Detail lengkap dengan contoh pertanyaan verbatim: [20_gap_schema_user](20_gap_schema_user.md).
> Ini memperbarui daftar open-question di atas: gap G1–G5 = pertanyaan user yang harus dijawab dengan
> "tidak tersedia" (anti-halusinasi), bukan hipotesis analisis internal.

---

## F. Daftar Query Prioritas (link ke [17_rencana_investigasi](17_rencana_investigasi_lanjut.md))

| Prioritas | Kelompok | Pertanyaan inti |
|---|---|---|
| 🔴 K1 | Akar PANGAN dead-end | Adakah PANGAN yg pernah 6/7/999? |
| 🔴 K2 | VP-limbo movement | Apakah sampel VP PANGAN masih disentuh? |
| 🟠 K3 | Kamus kode regulasi | MKL/TME/NIK per komoditi |
| 🟠 K4 | Disagreement pattern | Transisi dominan + korelasi waktu |
| 🟠 K5 | Metodologi durasi | Business days vs calendar |
| 🟠 K6 | Biaya rework | Total durasi pernah-ditolak vs lolos |
| 🟡 K7 | Orphan 2021 | Komoditi rekonstruksi |
| 🟡 K8 | Geografi | Korelasi coverage × durasi |
| 🟡 K9 | Konsistensi detail | agg per cell vs fact |
| 🟡 K10 | Struktur kode | Segmen ke-3 nomor_surat |
| ⚪ K11 | Eksternal | Dokumen BPOM (lihat [16](16_informasi_eksternal_dibutuhkan.md)) |

---

Lanjut ke [16_informasi_eksternal_dibutuhkan.md](16_informasi_eksternal_dibutuhkan.md).
