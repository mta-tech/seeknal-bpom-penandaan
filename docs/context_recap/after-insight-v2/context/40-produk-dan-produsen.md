Produk, produsen, pendaftar, dan penomoran.

## Kolom

| Kolom | Isi |
|---|---|
| `nama_produk` | nama produk yang labelnya dinilai |
| `produsen` | pembuat produk |
| `pendaftar` | pemilik izin edar |
| `nomorsampel` | penomoran sampel — **bukan kunci** |
| `nomor_surat` | penomoran surat pengawasan |
| `ed_nie` | tanggal berakhir izin edar |

## `produsen` vs `pendaftar` — dua peran berbeda

Pertanyaan tentang **"sarana produksi"** atau **"industri farmasi"** yang membuat produk menargetkan
`produsen`. Pertanyaan tentang **pemilik izin edar** menargetkan `pendaftar`. Keduanya sering
berbeda untuk produk yang sama.

Bila pengguna menulis "produsen" tetapi maksudnya pemegang izin (atau sebaliknya), hasilnya akan
berbeda — klarifikasi bila konteksnya tidak jelas.

Keduanya **teks bebas**: satu perusahaan bisa muncul dengan beberapa penulisan. Pola kerja jalur
**P3**: `ILIKE` sekali untuk menemukan varian, lalu hitung dengan himpunan varian itu, lalu
sebutkan penggabungannya.

Keterisian kedua kolom **tidak merata antar komoditi** — sebagian komoditi tidak merekam salah
satunya. Silangkan dengan `komoditi` sebelum memeringkat (`90-kualitas-data.md`).

## `nomor_surat` tidak menandai komoditi

Nomor surat berpola berawalan kode. Menggoda untuk menyimpulkan bahwa awalan tertentu berarti
komoditi tertentu — **itu tidak benar**. Satu awalan dipakai oleh beberapa komoditi, dan satu
komoditi tersebar di beberapa awalan.

> **Jangan menurunkan komoditi dari nomor surat.** Pakai kolom `komoditi` langsung.

## `ed_nie` bukan tanggal kegiatan

Kolom ini adalah tanggal berakhirnya izin edar produk, bukan tanggal penandaan. Jangan dipakai
untuk periode kegiatan.

Ia berguna untuk pertanyaan tentang masa berlaku izin — tetapi keterisiannya tidak merata antar
komoditi, dan sebagian nilainya jauh di luar rentang wajar. Periksa sebarannya sebelum memakainya.

## `nomorsampel` bukan kunci

Ada nomor sampel yang muncul lebih dari sekali — satu sampel bisa dinilai penandaannya lebih dari
sekali. Jangan memakainya sebagai pengenal unik; pakai `id`.

## Nama pendaftar punya kembaran karena spasi

Kolom pendaftar diisi bebas. Bentuk kembaran yang paling sering: spasi ganda sesudah bentuk badan
usaha, misalnya `PT` diikuti dua spasi alih-alih satu. Ada juga nilai yang berspasi di ujung.

Akibatnya satu perusahaan tersimpan sebagai beberapa entri berbeda, sehingga cacah perusahaan unik
menjadi lebih tinggi dari kenyataan dan peringkat pendaftar terpecah.

Aturan: cacah perusahaan unik dan peringkat pendaftar wajib memakai
`btrim(regexp_replace(pendaftar, '\s+', ' ', 'g'))`. Sebutkan di jawaban bahwa varian penulisan
digabungkan.

## Rute

- Menyebut komoditi: buka `10-komoditi.md`.
- Menyebut vonis: buka `20-vonis.md`.

---

<!-- MANIFES
tabel: -
kolom: ed_nie, komoditi, nama_produk, nomor_surat, nomorsampel, pendaftar, produsen
nilai: -
-->
