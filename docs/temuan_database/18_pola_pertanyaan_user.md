# 18 — Pola Pertanyaan User (dari Export KAI)

> Taksonomi pertanyaan asli yang diajukan user ke database `penandaan` — diekstrak dari `kai_question_sql_pairs.csv`
> (filter `db_alias LIKE 'penandaan%'`). Hanya **teks pertanyaan** yang dipakai; kolom `sql` hasil AI diabaikan (belum valid).
> Sumber data: 144 pertanyaan mentah → **101 unik** (setelah dedup + buang sampah "This is a TEST!").

---

## Konteks Sumber

| Item | Nilai |
|---|---|
| Sumber | `data_output/kai_sql_pairs/kai_question_sql_pairs.csv` |
| Filter | `db_alias LIKE 'penandaan%'` |
| Total pertanyaan mentah | 144 |
| Unik (dedup) | 101 |
| Periode pertanyaan | Didominasi konteks pengawasan 2024–2025 |
| Istilah kunci user | **UPT** (= `nama_balai`), **kesimpulan akhir** (= `kesimpulan_penilaian_pusat`), **tanggal proses direktur** (= `tanggal_kirim_direktur`) |

> ⚠️ **Terminologi penting:** User menyebut "UPT" (Unit Pelaksana Teknis = balai BPOM). Di database kolomnya bernama
> `nama_balai`. Ini jembatan istilah yang wajib dipahami agent — user TIDAK pernah menulis "balai".

---

## Taksonomi Pola Pertanyaan (10 Kategori)

### 1. Agregasi Total / Filter Waktu (17)

"Berapa jumlah penandaan..." / "total penandaan tahun/bulan X".

| # | Contoh verbatim |
|---|---|
| 1 | "berikan total penandaan tahun 2025" |
| 2 | "Berapakah Jumlah Penandaan pada bulan Juni 2025?" |
| 3 | "Berapakah total jumlah penandaan pada bulan Juli 2026?" |
| 4 | "Berapa total penandaan untuk tahun sebelumnya" |
| 5 | "Berapa jumlah penandaan selama tahun 2024" |
| 6 | "Berapakah total jumlah penandaan untuk kategori OBAT pada bulan Juli 2025?" |

**Mapping:** `COUNT(*) FROM mv_penandaan WHERE tgl_start` + filter tahun/bulan. Dimensi umum: komoditi.

---

### 2. Gap / Persamaan Pusat vs UPT (7–15)

"tampilkan data gap/perbedaan hasil pengawasan antara pusat dan UPT". **Kategori bisnis paling khas** — user membandingkan
keputusan balai (`kesimpulan_penilaian_balai`) dengan keputusan pusat (`kesimpulan_penilaian_pusat`).

| # | Contoh verbatim |
|---|---|
| 1 | "tampilkan data gap/perbedaan hasil pengawasan penandaan obat antara pusat dan upt untuk periode pengawasan tertentu, dengan opsi filter berdasarkan nama upt maupun keseluruhan upt." |
| 2 | "tampilkan data persamaan hasil pengawasan penandaan obat antara pusat dan upt untuk periode pengawasan 2025." |
| 3 | "Berdasarkan kabupaten/kota, tampilkan jumlah perbedaan kesimpulan balai dengan pusat saat kesimpulan penilaian pusat = TMK untuk tahun 2025" |
| 4 | "Berdasarkan produsen dua kelinci, tampilkan berapa jumlah label yang dilaporkan MK atau TMK beserta jumlah perbedaan antara kesimpulan balai dengan pusat" |
| 5 | "apa perbedaan Kesimpulan Penilaian Balai dan Kesimpulan Penilaian pusat pada tabel diatas?" |

**Mapping:** `WHERE kesimpulan_penilaian_balai <> kesimpulan_penilaian_pusat` (perbedaan) ATAU `=` (persamaan).

---

### 3. Rekapitulasi per UPT (14)

"rekapitulasi jumlah laporan pengawasan penandaan masing-masing UPT yang telah dikirim ke pusat pada periode X".

| # | Contoh verbatim |
|---|---|
| 1 | "tampilkan rekapitulasi jumlah laporan pengawasan penandaan masing-masing upt yang telah dikirim ke pusat pada periode waktu antara tanggal 1 januari 2025 hingga 31 desember 2025." |
| 2 | "Tampilkan rekapitulasi jumlah laporan pengawasan penandaan masing-masing UPT yang telah dikirim ke Pusat pada tahun 2025" |

**Mapping:** `GROUP BY nama_balai` + filter `tanggal_kirim_pusat` dalam rentang.

---

### 4. Rekapitulasi Berdasarkan Tanggal Proses Direktur (11)

"rekapitulasi ... dengan hasil kesimpulan akhir MK/TMK berdasarkan tanggal proses direktur". **Tanggal direktur adalah
dimensi grouping utama** user (bukan tgl_start).

| # | Contoh verbatim |
|---|---|
| 1 | "tampilkan rekapitulasi jumlah laporan pengawasan penandaan dengan hasil kesimpulan akhir yaitu memenuhi ketentuan berdasarkan tanggal proses direktur." |
| 2 | "tampilkan rekapitulasi jumlah laporan pengawasan penandaan dengan hasil kesimpulan akhir yaitu tidak memenuhi ketentuan berdasarkan tanggal proses direktur." |
| 3 | "Tampilkan rekapitulasi jumlah laporan pengawasan penandaan dengan hasil kesimpulan akhir yaitu memenuhi ketentuan berdasarkan tanggal proses direktur untuk tahun 2025" |

**Mapping:** `GROUP BY date_trunc('month', tanggal_kirim_direktur)` + filter `kesimpulan_penilaian_pusat = 'MK'/'TMK'`.
Lihat [09_waktu_durasi](09_waktu_durasi.md) — kolom di `mv_penandaan_timeline`.

---

### 5. Rekap / Ranking per Jenis Kemasan (3)

"urutkan nama kemasan yang memiliki kesimpulan akhir TMK paling banyak". 

| # | Contoh verbatim |
|---|---|
| 1 | "urutkan nama kemasan yang memiliki kesimpulan akhir pengawasan penandaan tmk paling banyak sampai dengan paling rendah." |
| 2 | "tampilkan rekapitulasi jumlah laporan pengawasan penandaan untuk masing-masing jenis kemasan dengan hasil kesimpulan akhir yaitu memenuhi ketentuan berdasarkan tanggal proses direktur." |

> ⚠️ **GAP SCHEMA:** Tidak ada kolom "jenis/nama kemasan" di schema. Jangan berhalusinasi — lihat [20_gap_schema_user](20_gap_schema_user.md).

---

### 6. Ranking UPT (Top/Bottom) (6–10)

"Balai/UPT mana yang jumlahnya tertinggi/terendah" / "top 5 UPT yang melaporkan MK/TMK".

| # | Contoh verbatim |
|---|---|
| 1 | "Balai mana yang memiliki jumlah penandaan tertinggi?" |
| 2 | "Balai mana yang memiliki jumlah penandaan terendah?" |
| 3 | "berikan top 5 upt yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt tidak memenuhi ketentuan?" |
| 4 | "upt mana saja yang paling banyak melaporkan hasil pemeriksaan penandaan dengan kesimpulan upt memenuhi ketentuan?" |

**Mapping:** `GROUP BY nama_balai ORDER BY count DESC/ASC LIMIT n`.

---

### 7. Ranking Produsen / Industri Farmasi / Sarana Produksi (4)

"urutan data nama industri farmasi / sarana produksi dengan hasil TMK paling banyak".

| # | Contoh verbatim |
|---|---|
| 1 | "tampilkan urutan data nama industri farmasi dengan hasil pengawasan akhir dari pusat yang memiliki obat dengan penandaan yang tidak memenuhi ketentuan paling banyak sampai dengan paling rendah." |
| 2 | "tampilkan urutan data nama sarana produksi dengan hasil pengawasan akhir dari pusat yang memiliki obat tradisional; suplemen kesehatan; obat kuasi dengan penandaan yang tidak memenuhi ketentuan diurutkan dari yang paling banyak sampai dengan paling sedikit." |
| 3 | "Tolong bikinkan 10 peringkat teratas Pelaku usaha Obat Bahan Alam dengan pelanggaran penandaan tertinggi" |

**Mapping:** `GROUP BY produsen` (nama sarana produksi ≈ `produsen`) + filter kesimpulan pusat.

---

### 8. Kesimpulan MK / TMK per Kategori (27)

"kategori produk mana yang TMK-nya tertinggi" / filter "kesimpulan akhir memenuhi ketentuan".

| # | Contoh verbatim |
|---|---|
| 1 | "Kategori 'Kesimpulan' manakah yang memiliki 'Jumlah Penandaan' tertinggi?" |
| 2 | "Kategori produk apa yang memiliki jumlah 'Tidak Memenuhi Ketentuan TMK' tertinggi?" |
| 3 | "Berapa banyak laporan penandaan obat dengan hasil kesimpulan UPT Memenuhi Ketentuan dari Balai Besar POM di Banda Aceh tahun 2025" |

**Mapping:** `WHERE kesimpulan_penilaian_pusat IN ('MK','TMK')` + `GROUP BY komoditi`. Perhatikan dua level:
"kesimpulan UPT" = balai, "kesimpulan akhir" = pusat.

---

### 9. Investigasi Catatan TMK (5)

"nama UPT yang tidak melakukan input catatan TMK pada hasil pengawasan dengan kesimpulan TMK".

| # | Contoh verbatim |
|---|---|
| 1 | "tampilkan nama upt yang tidak melakukan input catatan tmk pada hasil pengawasan penandaan dengan kesimpulan tmk." |
| 2 | "Tampilkan nama UPT yang tidak melakukan input catatan TMK ... berdasarkan prosedur input catatan TMK yang berlaku." |

**Mapping:** `WHERE kesimpulan_penilaian_pusat='TMK' AND (catatan IS NULL OR TRIM(catatan)='')`. Terkait temuan
"catatan ceremonial" di [06_penilaian_keputusan](06_penilaian_keputusan.md).

---

### 10. Timeline Pemenuhan (5)

"pemenuhan timeline pengawasan diukur sejak tanggal pemeriksaan sampai tanggal laporan dikirim ke pusat oleh kepala UPT".

| # | Contoh verbatim |
|---|---|
| 1 | "tampilkan data pemenuhan timeline pengawasan oleh masing-masing upt yang dapat diukur sejak tanggal pemeriksaan sampai dengan tanggal laporan dikirim ke pusat oleh kepala upt." |
| 2 | "Tampilkan jumlah label tepat waktu yang telah dikirimkan ke pusat pada UPT tertentu pada tahun 2025 Hasil pengawasan label dikirimkan ke pusat maksimal tanggal 15 bulan berikutnya" |
| 3 | "Tampilkan data pemenuhan timeline pengawasan oleh Balai Besar POM di jakarta ..." |

**Mapping:** `tanggal_kirim_pusat - tgl_start` (dari `mv_penandaan_timeline` join `mv_penandaan`). SLA user:
**maksimal tanggal 15 bulan berikutnya** — lihat [09_waktu_durasi](09_waktu_durasi.md).

---

## Pola Lain (Minor tapi Penting)

### Iklan / Klaim / Media Publikasi (7) — DOMAIN LAIN
"Tampilkan data iklan yang dilaporkan MK/TMK ... media publikasi, dan klaim dalam promosi/iklan".
> ⚠️ Ini domain **`pengawasan`** (iklan/promosi), BUKAN penandaan. Banyak pertanyaan ini di-export ke db_alias penandaan,
> tapi data iklan/media publikasi/klaim **tidak ada** di database penandaan. Lihat [20_gap_schema_user](20_gap_schema_user.md).

### Triplet OT / Supkes / Kuasi (6)
"obat tradisional; suplemen kesehatan; obat kuasi" — triplet khas diminta bersama sebagai satu group filter.
> Mapping: `komoditi IN ('OBAT TRADISIONAL (OT)','SUPLEMEN KESEHATAN','OBAT KUASI')`.

### Pangan / Label Pangan (3)
"ketidaksesuaian pelabelan dan periklanan pangan olahan", "data label yang dilaporkan MK/TMK ... jenis pangan, kategori pangan".
> Perhatikan: user menyebut "label" yang di sini berarti produk pangan. Mapping ke `komoditi='PRODUK PANGAN'`.

### Tren Multi-Tahun (2)
"apakah trend pelanggaran penandaan obat tradisional", "pelanggaran penandaan obat bahan alam paling banyak di tahun 2024".
> Mapping: `GROUP BY year(tgl_start)` + filter komoditi.

### Raw Export (3)
"Berikan 10000 rows penandaan select * limit 10000" → `SELECT * FROM mv_penandaan LIMIT 10000`.

### Grafik / Visualisasi (1)
"apa bisa di buat dalam bentuk grafik untuk jumlah penandaan setiap wilayah selama tahun 2024" → agregat per balai → grafik.

### Penjelasan Konsep (1–2)
"apa perbedaan Kesimpulan Penilaian Balai dan Kesimpulan Penilaian pusat?" → butuh jawaban konseptual (bukan SQL),
paling baik menjawab dari [06_penilaian_keputusan](06_penilaian_keputusan.md).

### Kabupaten / Kota / Provinsi Produsen (2)
"Berdasarkan kabupaten kota dan provinsi alamat produsen, tampilkan berapa jumlah label yang dilaporkan MK atau TMK".
> ⚠️ **GAP SCHEMA:** kolom lokasi/almar produsen tidak ada. Lihat [20_gap_schema_user](20_gap_schema_user.md).

---

## Ringkasan Frekuensi

| Peringkat | Pola | n | Bisa dijawab DB? |
|---|---|---:|---|
| 1 | Kesimpulan MK/TMK per kategori | 27 | ✅ |
| 2 | Agregasi total / filter waktu | 17 | ✅ |
| 3 | Gap / persamaan pusat vs UPT | 7–15 | ✅ |
| 4 | Rekap per UPT | 14 | ✅ |
| 5 | Rekap tanggal proses direktur | 11 | ✅ |
| 6 | Ranking UPT | 6–10 | ✅ |
| 7 | Iklan/klaim/media | 7 | ❌ (domain pengawasan) |
| 8 | Triplet OT/Supkes/Kuasi | 6 | ✅ |
| 9 | Timeline pemenuhan | 5 | ✅ |
| 10 | Investigasi catatan TMK | 5 | ✅ |
| 11 | Ranking produsen | 4 | ✅ |
| 12 | Jenis kemasan | 3 | ❌ (gap schema) |
| 13 | Pangan/label pangan | 3 | ✅ |
| 14 | Raw export | 3 | ✅ |
| 15 | Kabupaten/provinsi produsen | 2 | ❌ (gap schema) |
| 16 | Tren multi-tahun | 2 | ✅ |

> **Insight kunci:** ±80% pertanyaan user bisa dijawab dari 4 tabel inti (`mv_penandaan`, `mv_penandaan_timeline`,
> `mv_penandaan_log`). Gap utama: **jenis kemasan** & **lokasi produsen** (tidak ada kolom), dan **iklan/media/klaim**
> (domain database lain).

---

Lanjut ke [19_mapping_konsep_ke_schema.md](19_mapping_konsep_ke_schema.md) untuk pemetaan istilah → kolom.

---

## Batch pertanyaan tambahan — diuji ke DB live 2026-08-14

| Pertanyaan | Pemetaan | Catatan hasil |
|---|---|---|
| UPT paling banyak kesimpulan **UPT** MK | `kesimpulan_penilaian_balai='MK'` | jalan — tapi hanya sah untuk komoditi yang balainya menilai |
| Top 5 UPT kesimpulan **UPT** TMK | `upper(balai) LIKE 'TMK%'` | ⚠️ `= 'TMK'` melewatkan MAYOR/MINOR dan varian huruf kecil |
| Sarana produksi OT/SK/OK dengan hasil **pusat** TMK | `produsen` + `komoditi` + `pusat LIKE 'TMK%'` | jalan |
| **Gap penandaan OBAT pusat vs UPT 2025** | uji ketersediaan | ⛔ **tidak terdefinisi** — lihat di bawah |
| **Kategori produk** dengan TMK tertinggi | tidak ada kolom kategori | proksi terdekat `komoditi`; **bukan hal yang sama** |
| Total penandaan Juli 2025 | rentang `tgl_start` | ⚠️ **dua jawaban sah** (penandaan vs surat) |
| UPT tidak input catatan TMK | `catatan` sentinel `''` | anti-join kosong → sajikan sebagai **peringkat porsi** |
| Data OT/SK/OK **kategori + klaim promosi/iklan** | uji kolom | ⛔ **tidak ada kolomnya** |
| Urutkan **nama kemasan** TMK terbanyak | uji kolom | ⛔ **tidak ada kolomnya** |

### ⛔ Gap penandaan OBAT: pertanyaan tersering, dan tidak terdefinisi

```sql
SELECT komoditi, count(*) AS baris,
       count(*) FILTER (WHERE kesimpulan_penilaian_balai <> '') AS balai_terisi,
       count(*) FILTER (WHERE kesimpulan_penilaian_balai <> ''
                          AND kesimpulan_penilaian_pusat NOT IN ('','VP')
                          AND kesimpulan_penilaian_balai <> kesimpulan_penilaian_pusat) AS gap_sah
FROM mv_penandaan WHERE komoditi='OBAT' AND extract(year FROM tgl_start)=2025 GROUP BY 1;
```

Barisnya ada, **`balai_terisi` nol, `gap_sah` nol**. Untuk komoditi OBAT (dan ROKOK) UPT memang
tidak pernah merekam penilaian — jadi "gap antara pusat dan UPT" **tidak punya definisi**, bukan
"nol perbedaan". Rinciannya di `04_komoditi_governing_dimension.md`.

Jawaban yang benar menyebut batas ini. Jawaban yang memakai `IS NOT NULL` justru melaporkan seluruh
baris OBAT sebagai "gap" — lihat `21_sql_pairs_penandaan.md` §21.D.

### ⛔ Tiga pertanyaan meminta kolom yang tidak ada

```sql
SELECT column_name FROM information_schema.columns
WHERE table_schema='public' AND table_name='mv_penandaan'
  AND (column_name ILIKE '%kategori%' OR column_name ILIKE '%klaim%'
       OR column_name ILIKE '%iklan%'  OR column_name ILIKE '%kemasan%');
--  (0 baris)
```

| Diminta | Ada? | Bahaya kalau dipaksakan |
|---|---|---|
| **jenis/nama kemasan** | tidak | jatuh ke `nama_produk` → yang muncul nama obat, bukan kemasan |
| **kategori produk** | tidak | jatuh ke `komoditi` → cakupannya jauh lebih luas |
| **klaim dalam promosi/iklan** | tidak | domain iklan ada di database `pengawasan`, bukan di sini |

Ketiganya **P5 NOT COVERED**. Yang paling menyesatkan adalah "nama kemasan": query-nya jalan,
hasilnya rapi, dan pembaca tidak punya cara tahu bahwa yang ditampilkan bukan kemasan.

### ⚠️ "Total penandaan Juli 2025" — sebutkan entity-nya

`COUNT(*)` dan `COUNT(DISTINCT id)` memberi angka sama (id unik penuh di tabel ini), tetapi
`COUNT(DISTINCT nomor_surat)` jauh lebih kecil — satu surat memuat banyak penandaan. Keduanya sah;
sebutkan yang dipakai.
