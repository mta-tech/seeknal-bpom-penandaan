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

## Rute

- Menyebut komoditi → **seberang** `10-komoditi.md`.
- Menyebut periode → **seberang** `50-waktu-dan-durasi.md`.
