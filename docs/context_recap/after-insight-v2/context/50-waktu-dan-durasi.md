Waktu, periode, durasi, dan ketepatan waktu pelaporan.

## Periode

`tgl_start` adalah tanggal kanonik untuk periode dan tren; `tgl_end` untuk pertanyaan penyelesaian.

Pakai **rentang berbatas**. Jangan `EXTRACT(YEAR …)` sebagai penyaring — tidak ada indeks.

**Periode berjalan selalu parsial** — setiap tren wajib menyebutnya. Di domain ini ada efek kedua:
periode terbaru punya porsi lebih besar berstatus **menunggu verifikasi** di kolom vonis pusat,
sehingga tren berbasis vonis makin bias di ujung (`20-vonis.md`).

## Tabel timeline

`mv_penandaan_timeline` memuat tanggal milestone (mulai, kirim ke kepala balai, kirim ke direktur,
kirim ke pusat) dan beberapa kolom selisih.

PENTING: **Tabel ini memuat jauh lebih banyak id daripada tabel fakta** — INNER JOIN dari fakta bila
jawabannya tentang populasi penandaan (`30-status-dan-alur.md`).

PENTING: Kolom selisih **kosong pada tahap yang belum tercapai**. Merata-ratakan tanpa menyaring
mencampur "cepat" dengan "belum sampai", dan hasilnya bias ke bawah.

> **Aturan:** sebelum menghitung durasi tahap mana pun, saring baris yang tanggal tahap tujuannya
> sudah terisi, dan sebutkan berapa bagian populasi yang belum mencapai tahap itu.

Periksa juga sebaran nilai kolom selisih sebelum meratakannya — bila nilainya hanya beberapa
kemungkinan, kolom itu penanda, bukan jumlah hari.

## Pertanyaan berbasis tanggal tahap

Banyak pertanyaan di domain ini berbentuk *"rekapitulasi ... berdasarkan tanggal proses direktur"*
atau *"yang telah dikirim ke pusat"*. Keduanya memakai **tanggal tahap di timeline**, bukan
`status`.

Baris yang tanggal tahapnya masih kosong berarti **belum mencapai tahap itu** — kelompokkan
terpisah, jangan digabungkan ke salah satu sisi.

## Ketepatan waktu pelaporan

Pertanyaan bentuk *"label yang dikirim ke pusat sebelum tanggal N bulan berikutnya"* dijawab dengan
membandingkan tanggal kirim terhadap batas yang diturunkan dari bulan kegiatannya.

Dua hal yang harus dinyatakan:

1. **Batas tanggalnya berasal dari aturan unit**, bukan dari data — sebutkan batas yang dipakai;
   bila pengguna tidak menyebutnya, tanya.
2. **Baris tanpa tanggal kirim bukan "terlambat"** — ia belum dikirim. Pisahkan menjadi kategori
   sendiri.


## Kolom tahap di tabel timeline — inilah yang menjawab "tertahan di tahap mana"

Tabel timeline bukan sekadar penyimpan tanggal mulai dan selesai. Ia memuat **tanggal tiap tahap**
dan **kolom selisih antar tahap**:

| Bentuk kolom | Contoh namanya | Isinya |
|---|---|---|
| Tanggal tahap | `tanggal_kirim_kabalai`, `tanggal_kirim_pusat`, `tanggal_kirim_direktur` | kapan berkas dikirim ke tahap berikutnya |
| Selisih antar tahap | `mulai_kabalai`, `kabalai_direktur`, `direktur_pusat` | jarak antar dua tahap |

> PENTING: **Kolom bernama seperti selisih belum tentu berisi jumlah hari.** Sebagian di antaranya hanya
> punya sedikit kemungkinan nilai — itu **penanda**, bukan durasi. Sebelum memakai kolom selisih
> untuk menghitung rata-rata lama proses, **periksa dulu sebaran nilainya**. Kalau nilainya hanya
> beberapa kemungkinan, ia menandai terjadi/tidaknya sesuatu, dan merata-ratakannya tidak berarti.

**Kekosongan di kolom tahap bersifat deterministik.** Berkas yang tidak pernah naik ke suatu tahap
memang tidak punya tanggal untuk tahap itu — bukan data yang hilang, melainkan tahap yang belum
terjadi.

> **Aturan:** rata-rata lama tahap **hanya dihitung dari berkas yang benar-benar melewati tahap
> itu**. Menyertakan baris kosong akan menurunkan rata-rata secara keliru. Dan karena porsi berkas
> yang mencapai tahap akhir jauh lebih kecil daripada yang mencapai tahap awal, **sebutkan populasi
> mana yang dihitung** di kalimat jawaban.
>
> Pertanyaan "di tahap mana berkas paling lama tertahan" dijawab dengan membandingkan antar tahap
> **pada populasi yang sama** — yaitu berkas yang melewati semua tahap yang dibandingkan.

## Tanggal epoch sebagai pengganti kosong

Sebagian baris di tabel timeline memakai `1970-01-01` sebagai pengganti tanggal kosong. Jumlahnya
tidak kecil.

Nilai itu sah secara tipe data, jadi ia **ikut terhitung**: masuk ke `MIN()`, masuk ke selisih
durasi, dan menciptakan bucket tahun tersendiri pada tren. Durasi yang dihitung dari baris itu akan
bernilai puluhan ribu hari.

Aturan: setiap perhitungan durasi dan setiap pertanyaan "paling awal" wajib membuang `1970-01-01`
lebih dulu. Perlakukan sebagai penanda kosong, bukan sebagai tanggal.

## Rute

- Menyebut alur/tahapan: buka `30-status-dan-alur.md`.
- Menyebut target/capaian: buka `85-target-capaian.md`.

---

<!-- MANIFES
tabel: mv_penandaan_timeline
kolom: direktur_pusat, kabalai_direktur, mulai_kabalai, status, tanggal_kirim_direktur, tanggal_kirim_kabalai, tanggal_kirim_pusat, tgl_end, tgl_start
nilai: -
-->
