# 23. Cakupan context terhadap kondisi database `penandaan`

Dokumen ini menjawab satu pertanyaan: **apakah yang diajarkan ke agent sudah menutup**
**kondisi database yang sebenarnya.** Sumbernya profiling langsung ke warehouse, bukan
pembacaan skema. Seluruh isi dokumen ini khusus domain `penandaan`; istilah, kode, dan
perilaku kolom di sini **tidak berlaku untuk domain lain** dan tidak boleh dipinjam.

## Cara membacanya

Tiap kolom data ditempatkan di salah satu dari empat kuadran, dari dua pertanyaan:
apakah **context/skill menyebutnya**, dan apakah **SQL sistem lama pernah memakainya**.

| Kuadran | Arti | Tindakan |
|---|---|---|
| **A — aman** | diajarkan, dan pernah dipakai | tidak ada |
| **B — berlebih** | diajarkan, tapi tak pernah dipakai | biarkan; menyiapkan pertanyaan yang belum muncul |
| **C — regresi** | **tidak** diajarkan, padahal SQL lama memakainya | tutup; kemampuan yang hilang saat migrasi |
| **D — titik buta** | tidak diajarkan, dan tak pernah dipakai siapa pun | nilai satu per satu; sebagian memang tak perlu |

Dari **60 kolom data** di **6 tabel** (kolom pembukuan ETL `sync`/`last_updated` tidak dihitung):

| Kuadran | Jumlah | Porsi |
|---|--:|--:|
| A | 28 | 46% |
| B | 8 | 13% |
| C | 6 | 10% |
| D | 18 | 30% |

> Angka di tabel ini menggambarkan **cakupan dokumen**, bukan isi data. Angka isi data
> tidak dibawa ke halaman `context/` — halaman itu mengajarkan pemetaan, bukan nilai.

## Batas alat ukur ini — wajib dibaca sebelum menindak kuadran C dan D

Penempatan kuadran dihitung dengan mencocokkan **nama kolom** ke teks context. Cara itu punya dua
kelemahan yang sudah terbukti, dan keduanya membuat kuadran C dan D **melebih-lebihkan** lubang.

**Pertama, aturan tingkat tabel tidak terdeteksi.** Kalau context mengajarkan sebuah aturan tentang
satu tabel tanpa menyebut kolomnya satu per satu, semua kolom tabel itu jatuh ke kuadran D seolah
tak dikenal. Kasus nyatanya di domain ini adalah aturan kubus agregasi — bahwa `periode_type`
bernilai dua dan wajib disaring salah satu — yang **sudah tertulis dengan benar** di
`00-menghitung.md`, namun kolom-kolom kubusnya tetap muncul di kuadran D. Itu artefak pengukuran,
**bukan lubang**.

**Kedua, alat ukur bisa gagal diam-diam.** Versi pertama pengukuran ini melaporkan seluruh kolom
tercakup — hasil yang mustahil. Sebabnya pemisah kolom tertulis sebagai teks literal, bukan tab,
sehingga nama kolom menjadi string kosong dan pola pencarian cocok ke apa saja. Setiap pengukuran
ulang wajib menyertakan **kontrol negatif**: nama kolom yang sengaja dikarang harus dilaporkan
tidak ditemukan. Tanpa itu, angka cakupan tidak boleh dipercaya.

**Karena itu:** perlakukan kuadran C dan D sebagai **daftar kandidat**, bukan vonis. Yang sudah
diverifikasi satu per satu terhadap warehouse ada di bagian *Lubang yang terbukti* di bawah —
hanya itu yang layak ditindak.

## C — Regresi: dipakai sistem lama, tidak diajarkan sekarang

Ini kelompok paling mendesak. SQL sistem lama membuktikan kolomnya **memang dipakai untuk**
**menjawab pertanyaan nyata**; kalau context sekarang tidak menyebutnya, kemampuan itu hilang
tanpa ada yang sadar.

| Tabel | Kolom | Kondisi data |
|---|---|---|
| `mv_penandaan_timeline` | `direktur_pusat` | berkode, ±3 nilai · kosong 30% |
| `mv_penandaan_timeline` | `kabalai_direktur` | ±224 nilai · kosong 30% |
| `mv_penandaan_timeline` | `mulai_kabalai` | ±268 nilai · kosong 4% |
| `mv_penandaan_timeline` | `tanggal_kirim_direktur` | ±884 nilai · kosong 30% |
| `mv_penandaan_timeline` | `tanggal_kirim_kabalai` | ±1872 nilai · kosong 4% |
| `mv_penandaan_timeline` | `tanggal_kirim_pusat` | ±1742 nilai · kosong 5% |

## D — Titik buta: tak dikenal context maupun sistem lama

Sebagian memang tidak perlu diajarkan (id internal, indeks posisi array, stempel waktu baris).
Sisanya adalah kemampuan yang belum pernah dipakai siapa pun.

| Tabel | Kolom | Kondisi data | Perlu? |
|---|---|---|---|
| `coverage_balai` | `id_balai` | kardinalitas 13% dari baris | tidak — teknis |
| `coverage_balai` | `id_kabupaten` | kardinalitas 77% dari baris | tidak — teknis |
| `coverage_balai` | `kabupaten_kota` | kardinalitas 77% dari baris | nilai manual |
| `mv_penandaan_agg` | `avg_durasi_hari` | ±461 nilai | nilai manual |
| `mv_penandaan_agg` | `jumlah_produk_unik` | ±113 nilai | nilai manual |
| `mv_penandaan_agg` | `jumlah_surat_unik` | berkode, ±26 nilai | nilai manual |
| `mv_penandaan_agg` | `max_durasi_hari` | berkode, ±46 nilai | nilai manual |
| `mv_penandaan_agg` | `min_durasi_hari` | berkode, ±40 nilai | nilai manual |
| `mv_penandaan_log` | `fullname` | ±1459 nilai | nilai manual |
| `mv_penandaan_log` | `status_code` | berkode, ±16 nilai | nilai manual |
| `mv_penandaan_log` | `tanggal_proses` | ±130993 nilai · kosong 8% | nilai manual |
| `target_balai` | `target_penandaan` | kardinalitas 48% dari baris | nilai manual |
| `target_balai` | `target_pengawasan` | berkode, ±53 nilai | nilai manual |
| `target_balai` | `target_pengujian` | kardinalitas 48% dari baris | nilai manual |
| `target_balai` | `target_pengujian_pangan` | kardinalitas 13% dari baris · kosong 3% | nilai manual |
| `target_balai` | `target_pengujian_pangan_fortifikasi` | berkode, ±19 nilai · kosong 3% | nilai manual |
| `target_balai` | `target_sarana_distribusi` | kardinalitas 33% dari baris | nilai manual |
| `target_balai` | `target_sarana_produksi` | kardinalitas 14% dari baris | nilai manual |

## Katalog nilai kolom berkode

Kolom yang nilainya terbatas dikatalogkan penuh di bawah — inilah "kode filter" yang boleh
diajarkan. Yang tidak boleh dibawa ke `context/` adalah **cacah barisnya**, karena itu
bergeser tiap ETL; karena itu di sini hanya nilainya yang didaftar, tanpa jumlah.

### `mv_penandaan` . `kesimpulan_penilaian_balai`  ·  kuadran A

`MK`, _(string kosong)_, `TMK`, `TMK MINOR`, `TMK MAYOR`, `TMK Minor`

⚠️ Penanda kosong di kolom ini: string kosong — perlakukan sebagai "belum diisi", bukan sebagai kategori.

### `mv_penandaan` . `kesimpulan_penilaian_pusat`  ·  kuadran A

`MK`, `TMK`, `VP`, `TMK MAYOR`, `TMK MINOR`

### `mv_penandaan` . `komoditi`  ·  kuadran A

`KOSMETIKA`, `PRODUK PANGAN`, `OBAT`, `OBAT TRADISIONAL (OT)`, `ROKOK`, `SUPLEMEN KESEHATAN`, `OBAT KUASI`, `KEMASAN PANGAN`

### `mv_penandaan_agg` . `kesimpulan_penilaian_balai`  ·  kuadran A

`MK`, _(string kosong)_, `TMK`, `TMK MINOR`, `TMK MAYOR`, `TMK Minor`

⚠️ Penanda kosong di kolom ini: string kosong — perlakukan sebagai "belum diisi", bukan sebagai kategori.

### `mv_penandaan_agg` . `kesimpulan_penilaian_pusat`  ·  kuadran A

`MK`, `TMK`, `VP`, `TMK MINOR`, `TMK MAYOR`

### `mv_penandaan_agg` . `komoditi`  ·  kuadran A

`KOSMETIKA`, `PRODUK PANGAN`, `OBAT TRADISIONAL (OT)`, `OBAT`, `ROKOK`, `SUPLEMEN KESEHATAN`, `OBAT KUASI`, `KEMASAN PANGAN`

### `mv_penandaan_agg` . `periode_type`  ·  kuadran B

`day`, `month`

### `mv_penandaan_log` . `status_label`  ·  kuadran A

`Operator - Draft Sampling`, `MT - Pembuatan SPK`, `Supervisor - Verifikasi`, `TPS - Penerimaan SPU`, `Deputi MT - Pembuatan SPK`, `Penguji - Entri Hasil Pengujian`, `Sampel Rujukan Selesai`, `Penyelia - Pembuatan SPP`, `Supervisor 2 - Verifikasi`, _(SQL NULL)_, `Operator - Perbaikan Sampel`, `Kepala Balai`

⚠️ Penanda kosong di kolom ini: SQL NULL — perlakukan sebagai "belum diisi", bukan sebagai kategori.

### `mv_penandaan_log` . `trx_steps`  ·  kuadran B

`draft`, `pusat`, `spv_1`, `kepala_balai`, `spv_1_pusat`, `direktur`, `selesai`, `spv_2_pusat`, `spv_2`, `ditolak_spv_1`, `ditolak_pusat`, `ditolak_spv_1_pusat`, `ditolak_kepala_balai`, `ditolak_spv_2`, `ditolak_spv_2_pusat`, `ditolak_direktur`, _(SQL NULL)_, `receive_tps`, `spv1`

⚠️ Penanda kosong di kolom ini: SQL NULL — perlakukan sebagai "belum diisi", bukan sebagai kategori.

### `target_balai` . `komoditi`  ·  kuadran A

`Obat Kuasi`, `Produk Pangan`, `Obat Tradisional (OT)`, `Obat`, `Kosmetika`, `Suplemen Kesehatan`, `Rokok`


---

## Apa yang diceritakan database ini

`penandaan` merekam **penilaian terhadap label dan kemasan produk** — apakah penandaan sebuah produk
memenuhi ketentuan menurut balai dan menurut pusat.

| Lapis | Cerita | Tabel |
|---|---|---|
| Peristiwa | produk apa yang dinilai penandaannya, oleh balai mana, vonisnya apa | `mv_penandaan` |
| Perjalanan berkas | tahap demi tahap sampai disetujui | `mv_penandaan_log` + `mv_penandaan_timeline` |
| Rencana vs capaian | target dan cakupan wilayah | `target_balai` + `coverage_balai` |
| Kubus | agregasi siap pakai dari lapis peristiwa | `mv_penandaan_agg` |

Ini domain paling ramping dari keempatnya: satu tabel peristiwa, tanpa tabel anak yang memecah
grain. Karena itu satu penandaan tetap satu baris, dan pertanyaan "berapa" jauh lebih jarang ambigu
di sini dibanding domain lain.

**Ciri khas yang menentukan hampir semua jawaban:** kekosongan di database ini ditulis sebagai
**string kosong**, bukan SQL NULL. Setiap filter `IS NULL` di sini tidak menyaring apa pun, dan
setiap perbandingan "balai berbeda dengan pusat" akan melaporkan seluruh baris yang belum dinilai
sebagai perbedaan — padahal yang terjadi hanyalah penilaian itu belum ada.

Ciri kedua: untuk sebagian komoditi, kolom vonis balai **tidak pernah terisi sama sekali** — bukan
jarang, melainkan setiap baris. Untuk komoditi itu, selisih balai-pusat **tidak terdefinisi**, bukan
bernilai nol. Perbedaan itulah yang menentukan benar-salahnya pertanyaan paling sering di domain ini.

---

## Lubang yang terbukti — dan sifatnya: PENYALURAN, bukan penemuan

Satu hal yang harus dibaca sebelum daftar di bawah, karena ia mengubah ke mana perbaikan diarahkan.

Sebagian besar lubang di bawah **bukan** berarti temuannya belum pernah dibuat. Diperiksa ulang
14 Agustus 2026, mayoritasnya **sudah terdokumentasi dengan benar di direktori ini sejak awal** —
lengkap dengan katalog nilai, grain, dan jebakannya. Yang gagal adalah **penyalurannya ke
`context/`**: halaman context menyebut nama tabelnya lalu berhenti, sehingga pengetahuan yang sudah
dimiliki repositori ini tidak pernah sampai ke agent yang menjawab pertanyaan.

Karena itu tiap butir di bawah mencantumkan **berkas topik** tempat rinciannya tinggal. Dokumen ini
mencatat **pengukurannya**; rincian datanya tetap di berkas topiknya masing-masing, dan di sanalah
pembaruan berikutnya harus ditulis.

Implikasinya untuk cara kerja: menambah dokumen temuan **tidak dengan sendirinya** menutup lubang.
Setiap temuan yang mengubah cara menjawab harus punya pasangan di `context/` atau di skill.

### Daftar temuan

### 1. Kolom tahap di `mv_penandaan_timeline` tidak diajarkan

> 📄 Rincian datanya tinggal di `09_waktu_durasi.md` — kolom tahap dan kekosongan deterministiknya

`tanggal_kirim_kabalai`, `tanggal_kirim_pusat`, `tanggal_kirim_direktur`, `mulai_kabalai`,
`kabalai_direktur`, `direktur_pusat` — inilah kolom yang menjawab "berkas tertahan di tahap mana"
dan "berapa lama sampai selesai". SQL sistem lama memakai seluruhnya; context sekarang hanya
menyebut nama tabelnya.

**Bukti kondisi:** kolom tahap direktur kosong pada sekitar sepertiga baris, sedangkan tahap awal
hampir selalu terisi. Kekosongan itu **deterministik** — berkas yang tidak pernah naik ke tahap itu
memang tidak punya tanggalnya. Rata-rata durasi yang menyertakan baris kosong akan salah, dan
karena porsi kosongnya besar, salahnya besar.

Ini kelompok regresi terbesar di domain ini: **enam dari enam** kolom tahap hilang dari context.

### 2. `target_balai` — tabelnya disebut, kolom targetnya tidak

> 📄 Rincian datanya tinggal di `14_kpi_target_2024.md` — grain, batas tahun, dan pemilihan kolom target

Halaman `85-target-capaian.md` menyebut nama tabel dan kunci join, lalu berhenti.

**Bukti kondisi:** grain-nya **balai × komoditi** (76 balai × 7 komoditi), bukan satu baris per
balai. `tahun` hanya berisi **2024**. Ada **tujuh kolom target berbeda**; untuk domain ini yang
relevan adalah `target_penandaan`, sisanya milik kegiatan lain dan **tidak boleh dipakai di sini**.

**Akibatnya:** pertanyaan capaian tidak bisa dijawab tanpa menebak kolom, dan menjumlahkan tanpa
sadar grain-nya akan melipatgandakan target tujuh kali. Untuk tahun selain 2024 tidak ada
pembanding sama sekali — itu harus dikatakan, bukan dijawab nol.

### 3. `status_code` di log tidak diajarkan

> 📄 Rincian datanya tinggal di `05_workflow_state_machine.md` — blok kode tahap versus blok kode penolakan

Log memakai `status_code` numerik dengan blok nilai kecil untuk tahap normal dan **blok nilai besar
yang terpisah jauh** untuk jalur penolakan. Domain ini punya beberapa nilai tahap yang tidak muncul
di domain lain. Pemisahan blok itulah yang membuat pertanyaan "berapa yang ditolak dan di tahap
mana" bisa dijawab. Context tidak menyebut kolom ini sama sekali.

---

## Yang sudah benar dan tidak perlu diubah

Agar audit ini jujur dua arah: aturan kubus agregasi di domain ini **sudah diajarkan dengan benar**.
Context sudah menyatakan bahwa `periode_type` bernilai dua dan wajib disaring salah satu.
Pemeriksaan ulang terhadap warehouse membenarkannya, dan sekaligus menunjukkan bahwa kubus
beragregasi berdasarkan **tanggal selesai** sementara tanggal kanonik yang diajarkan adalah tanggal
mulai — jadi tren dari kubus tidak sebanding dengan tren dari tabel fakta. Yang terakhir itu belum
tertulis dan perlu ditambahkan.

---

## Yang TIDAK ditutupi oleh daftar pertanyaan mana pun

Ini menjawab langsung pertanyaan "apakah pertanyaan yang ada sudah mencakup semua kondisi": **tidak.**

Terbukti nol penyebutan di kedua korpus pertanyaan — seluruh kolom durasi ringkas di kubus
(`avg_durasi_hari`, `min_durasi_hari`, `max_durasi_hari`), hitungan unik di kubus, identitas pelaku
di log, serta stempel waktu proses di log.

Konsekuensinya untuk cara kerja kita: **daftar pertanyaan tidak bisa dipakai sebagai ukuran
kelengkapan context.** Kalau context hanya menutupi yang pernah ditanyakan, ia akan gagal pada
pertanyaan pertama yang keluar dari kebiasaan. Ukuran yang benar adalah kuadran C dan D di atas.

---

## Pencocokan temuan konsistensi terhadap `context/` yang hidup sekarang

Diverifikasi 14 Agustus 2026, dengan **kondisi database sebagai acuan mutlak**. Tiap temuan
konsistensi penulisan dan anomali tanggal — rinciannya di berkas kualitas data domain ini —
dicocokkan ke apa yang benar-benar tertulis di `context/` dan `skills/` saat ini.

Kolom **Status** memakai tiga nilai, dan bedanya penting:

| Status | Arti |
|---|---|
| **SUDAH** | aturannya ada dan benar — jangan diubah |
| **BELUM** | aturannya tidak ada di mana pun — perlu ditambahkan |
| **SALAH ARAH** | ada aturan, tetapi isinya menyesatkan terhadap kondisi database — **perbaiki lebih dulu daripada menambah apa pun** |


| Temuan | Status | Yang tertulis sekarang | Perubahan yang dibutuhkan |
|---|---|---|---|
| `TMK MINOR` versus `TMK Minor` — dua kapitalisasi satu gradasi | **SUDAH** | `90-kualitas-data.md` dan `85-target-capaian.md` sudah mewajibkan pencocokan pola awalan dan case-insensitive | — |
| `trx_steps` memuat satu baris salah ketik (`spv1`) | **SUDAH sebagian** | `90-kualitas-data.md` menyuruh memeriksa daftar nilai langkah alur | Pertegas aturannya: nilai bervolume sangat kecil yang merupakan varian penulisan dari nilai bervolume besar diabaikan saat menyusun daftar tahap |
| `pendaftar` punya puluhan kembaran dari spasi ganda sesudah `PT`/`CV` | **BELUM** | tidak ada | Tambahkan ke `40-produk-dan-produsen.md`: cacah perusahaan unik dan peringkat pendaftar terlalu tinggi tanpa `btrim(regexp_replace(...))`, dan penggabungan varian harus disebut di jawaban |
| Spasi ekor pada nama balai membuat filter kesamaan persis nol baris | **BELUM** | `85-target-capaian.md` membahas beda kapitalisasi antar tabel, bukan spasi ekor | Tambahkan aturan filter kesamaan persis pada `nama_balai` wajib lewat `trim()` |
| Epoch `1970-01-01` mengisi ribuan baris kolom timeline | **BELUM** | tidak ada | Tambahkan ke `50-waktu-dan-durasi.md`: setiap perhitungan durasi wajib membuang `1970-01-01` lebih dulu — ia tanggal yang sah secara tipe data, jadi ikut terhitung dan menghasilkan durasi puluhan ribu hari |
| `ed_nie` memuat tahun terpotong dan tahun mustahil | **BELUM** | tidak ada | Tambahkan ke `90-kualitas-data.md`: pertanyaan "paling awal / paling lama" wajib membatasi tahun; tahun 2027+ pada `ed_nie` **wajar** karena itu masa berlaku izin edar |
| Nama pelaku di log terpecah oleh cara menulis gelar | **BELUM** | tidak ada | Tambahkan sebagai keterbatasan pada `30-status-dan-alur.md` |

### Urutan yang disarankan

**SALAH ARAH lebih dulu.** Aturan yang menyesatkan lebih berbahaya daripada aturan yang tidak ada:
kalau tidak ada aturan, agent akan memakai kuota probe dan sering menemukan sendiri; kalau ada
aturan yang salah, ia akan mengikutinya dengan yakin dan hasilnya terlihat masuk akal.

Sesudah itu baru **BELUM**, didahulukan yang paling sering mengubah angka jawaban.

Yang berstatus **SUDAH** tidak boleh disentuh — daftar ini juga berfungsi melindunginya dari
perubahan yang tidak perlu.

> Dokumen ini adalah **acuan perubahan** untuk `context/` dan `skills/`. Perubahan itu sendiri
> belum dikerjakan; tidak ada satu pun berkas context atau skill yang diubah saat dokumen ini
> ditulis.
