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

⚠️ **Tabel ini memuat jauh lebih banyak id daripada tabel fakta** — INNER JOIN dari fakta bila
jawabannya tentang populasi penandaan (`30-status-dan-alur.md`).

⚠️ Kolom selisih **kosong pada tahap yang belum tercapai**. Merata-ratakan tanpa menyaring
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

## Rute

- Menyebut alur/tahapan → **seberang** `30-status-dan-alur.md`.
- Menyebut target/capaian → **seberang** `85-target-capaian.md`.
