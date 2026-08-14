Status dan alur persetujuan.

## `status` di timeline — angka

Kode status bertipe **angka**, berjalan dari tahap draft sampai selesai, ditambah blok kode
penolakan di rentang atas. Ambil daftarnya lewat `SELECT DISTINCT` — jalur **P2**.

## ⚠️ Label status di log DIPINJAM dari domain lain

`mv_penandaan_log` punya kolom `trx_steps` (nama langkah) dan `status_label` (label berbahasa
manusia).

> **`status_label` di domain ini menyalin kamus label domain pengujian.** Langkah persetujuan
> direktur berlabel istilah pengujian laboratorium, langkah kepala balai berlabel istilah
> penerimaan sampel, dan langkah selesai berlabel istilah sampel rujukan.

Akibatnya, menceritakan alur penandaan dengan `status_label` menghasilkan kalimat yang tidak masuk
akal bagi pengguna — misalnya laporan penandaan disebut masuk tahap entri hasil pengujian.

> **Aturan: pakai `trx_steps` untuk menceritakan alur.** `status_label` hanya boleh dipakai bila
> pertanyaannya memang tentang label mentahnya.

⚠️ `trx_steps` juga memuat **satu nilai salah ketik** — penulisan langkah tanpa pemisah yang
seharusnya sama dengan langkah lain. Periksa daftar nilainya; bila menemukan dua nilai yang jelas
sama maksudnya, gabungkan dan sebutkan.

## ⚠️ Log dan timeline jauh lebih luas dari fakta

Keduanya memuat **jauh lebih banyak** id daripada tabel fakta — selisihnya besar, bukan marginal.

> Menghitung dari log atau timeline **langsung** akan melebihi populasi penandaan secara
> signifikan. Bila jawabannya berbicara tentang populasi penandaan, **INNER JOIN dari fakta**.

Sifat id tambahan itu belum dipastikan — jangan menyimpulkan sebabnya.

## Batas yang harus dihormati

Pertanyaan *"siapa yang menyetujui"*, *"apakah pemisahan tugas berjalan"* → **P5 NOT COVERED**.
Kolom pelaku di log tidak dapat dipastikan artinya dari database ini sendirian, dan kesimpulannya
bersifat tuduhan.

Yang **boleh** dijawab dari log: kapan berkas berpindah tahap, berapa lama tersangkut, dan tahap
mana yang paling banyak menahan berkas.

## Istilah pengguna

| Istilah | Cara mengikat |
|---|---|
| "sudah selesai" | kode tahap akhir |
| "berdasarkan tanggal proses direktur" | tanggal tahap direktur di timeline, bukan status |
| "sudah dikirim ke pusat" | tanggal tahap pusat di timeline |
| "ditolak" | langkah penolakan di `trx_steps` |

## Rute

- Menyebut durasi/ketepatan waktu → **seberang** `50-waktu-dan-durasi.md`.
- Menyebut vonis → **seberang** `20-vonis.md`.
