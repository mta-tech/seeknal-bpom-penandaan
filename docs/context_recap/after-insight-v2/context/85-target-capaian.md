Target, capaian, dan pertanyaan "UPT mana yang belum melaporkan".

## `target_balai`

Memuat target tahunan per `(nama_balai, komoditi, tahun)`, dengan kolom target terpisah per jenis
kegiatan — pakai kolom target penandaan untuk domain ini.

### Tiga batas struktural

**1. Hanya satu tahun.** Periksa dulu (`SELECT DISTINCT tahun FROM target_balai`). Untuk realisasi
di tahun yang tidak ada targetnya: sajikan realisasi **tanpa** persentase capaian, atau bandingkan
terhadap tahun yang tersedia **sambil menyatakan bahwa tahunnya berbeda**.

**2. Nama balai beda kapitalisasi.** Tabel fakta menulis huruf besar, tabel target huruf campuran.
**Join persis akan gagal** untuk sebagian besar nama — selalu `lower(trim(...))` di kedua sisi.

**3. Unit pusat tidak punya target.** Nilai `nama_balai` yang berupa direktorat adalah unit pusat.
Laporkan terpisah; jangan menghitung capaian nol untuknya, dan jangan membuangnya diam-diam.

Komoditi di kedua sisi memakai penulisan berbeda (tabel target memakai huruf campuran) —
normalisasi juga di sisi komoditi.

## Bentuk jawaban capaian

Sebelum menyajikan capaian:

1. sebutkan **tahun target** yang dipakai;
2. sebutkan **entity realisasi** — penandaan atau surat (`00-menghitung.md` §1);
3. keluarkan unit pusat dari agregat nasional, dan katakan;
4. bila periodenya berjalan, sebutkan bahwa realisasinya belum lengkap.

## Pertanyaan "UPT mana yang TIDAK melaporkan"

Bentuk anti-join sering menghasilkan **himpunan kosong** — praktis setiap balai melaporkan sesuatu.
Ubah menjadi **peringkat porsi** dan sampaikan keduanya (`60-catatan.md` memakai pola yang sama).

## Peringkat "UPT paling banyak melaporkan MK/TMK"

Dua hal yang wajib diperhatikan:

1. **Kolom vonis mana** — balai atau pusat. Untuk sebagian komoditi kolom balai kosong seluruhnya,
   sehingga peringkat "kesimpulan UPT" hanya sah untuk sebagian komoditi (`10-komoditi.md`).
2. **Keluarga TMK dicocokkan dengan pola awalan dan case-insensitive** (`20-vonis.md`).

Peringkat mentah juga bias terhadap balai bervolume besar. Bila pertanyaannya tentang kinerja,
sajikan **porsi**, bukan hanya cacah — atau sertakan keduanya.

## Cakupan wilayah

`coverage_balai` memuat wilayah kerja balai. Jumlah balai di cakupan dan di fakta **tidak sama** —
balai tanpa penandaan pada periode itu harus tampil sebagai nol, bukan hilang. LEFT JOIN dari sisi
cakupan.


## Tabel target memuat TUJUH kolom target — hanya sebagian milik domain ini

Ini penyebab kesalahan yang paling mudah terjadi di halaman ini, karena semua kolomnya bernama
mirip dan semuanya berisi angka yang masuk akal.

`target_balai` melayani beberapa kegiatan pengawasan sekaligus. Kolom targetnya:
`target_penandaan`, `target_pengawasan`, `target_pengujian`, `target_pengujian_pangan`,
`target_pengujian_pangan_fortifikasi`, `target_sarana_distribusi`, `target_sarana_produksi`.

**Untuk domain ini yang dipakai adalah `target_penandaan`.** Satu kolom saja untuk domain ini.

> **Aturan:** kolom target dipilih berdasarkan **kegiatan yang ditanya**, bukan berdasarkan angka
> mana yang terlihat wajar. Kolom milik kegiatan lain **tidak boleh dipakai di sini** meskipun
> terisi — angkanya nyata, tetapi menjawab pertanyaan yang berbeda.

## Grain tabel target: satu baris = satu balai × satu komoditi × satu tahun

Bukan satu baris per balai. Setiap balai punya beberapa baris, satu untuk tiap komoditi.

Konsekuensinya:

| Yang ingin dijawab | Yang harus dilakukan |
|---|---|
| Target satu balai untuk satu komoditi | ambil barisnya langsung |
| Target satu balai keseluruhan | jumlahkan seluruh komoditinya |
| Target nasional | jumlahkan seluruh balai **dan** seluruh komoditi |
| Membandingkan dengan capaian per komoditi | agregasi capaian juga harus per komoditi |

> **Aturan:** menjumlahkan kolom target tanpa menyadari grain-nya akan **melipatgandakan** hasilnya
> sebanyak jumlah komoditi. Selalu tentukan lebih dulu apakah pertanyaannya per komoditi atau
> gabungan, lalu samakan tingkat agregasi kedua sisi — target dan capaian.

## Tabel target tidak mencakup semua tahun

Kolom `tahun` di tabel ini **tidak berisi seluruh tahun operasional**. Jangan berasumsi tahun yang
diminta pengguna tersedia.

> **Aturan:** sebelum menjawab pertanyaan capaian, **periksa dulu tahun apa saja yang ada** di
> tabel target. Bila tahun yang diminta tidak ada, jawab bahwa pembandingnya tidak tersedia untuk
> tahun itu — **jangan** menjawab capaian nol, dan **jangan** diam-diam memakai tahun lain sebagai
> pengganti.
>
> Ini pemeriksaan, bukan fakta yang dihafal: isi tabel bisa bertambah kapan saja, jadi periksa
> setiap kali alih-alih mengandalkan apa yang pernah benar.

## Nama balai punya spasi tersembunyi di ujung

Sebagian nilai nama balai tersimpan dengan spasi menempel di belakang. Spasi itu konsisten di semua
tabel yang memuat nama balai, jadi join antar tabel tetap jalan. Yang gagal adalah filter kesamaan
persis.

Menulis `nama_balai = 'BALAI POM DI ...'` dengan nama yang disalin dari dokumen atau diketik dari
ingatan akan mengembalikan nol baris tanpa pesan kesalahan, seolah balai itu tidak punya data.

Aturan: filter kesamaan persis pada nama balai wajib memakai `trim()` di kedua sisi, atau memakai
nilai hasil probe `SELECT DISTINCT` apa adanya termasuk spasinya. Jangan mengetik nama balai dari
ingatan.

## Rute

- Menyebut komoditi: buka `10-komoditi.md`.
- Menyebut periode: buka `50-waktu-dan-durasi.md`.

---

<!-- MANIFES
tabel: coverage_balai, target_balai
kolom: nama_balai, tahun, target_penandaan, target_pengawasan, target_pengujian, target_pengujian_pangan, target_pengujian_pangan_fortifikasi, target_sarana_distribusi, target_sarana_produksi
nilai: -
-->
