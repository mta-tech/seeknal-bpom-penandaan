# Test case — domain `penandaan`

**Dibuat:** 14 Agustus 2026 · **Total:** 110 test dalam 8 folder.
**Skema berkas:** mengikuti pola UAT `seeknal-bpom-neo/seeknal/tests/v1/singleturn/UAT-v2-compact`.

## Tujuan

Mengevaluasi apakah agent mampu menjawab **dengan membaca context dan skill** yang ada — bukan
menebak. Angka pada `assert_any_of` seluruhnya diambil dari eksekusi SQL langsung ke database
domain ini pada tanggal verifikasi, bukan ditulis tangan.

## Dua kelas test

**Kelas A — regresi.** Diambil dari SQL pair sistem lama yang sudah terbukti jalan, SQL-nya
dijalankan ulang untuk mendapat nilai assert. Tugasnya membuktikan context baru **tidak merusak**
jawaban yang selama ini benar. Folder bernomor 11 ke atas.

**Kelas B — diskriminasi.** Ditulis dari temuan audit, menyasar aturan yang versi context sekarang
belum atau salah mengajarkan. Ciri khasnya: versi lama gagal, versi baru harus lolos. Folder
berawalan `0x-B-`.

Kelas A saja tidak cukup: pemetaan menunjukkan SQL pair nyaris tidak menyentuh target/capaian,
kode tahap log, sentinel, maupun kolom tahap timeline — persis yang diperbaiki versi baru. Tanpa
kelas B, seluruh test akan lolos di kedua versi dan tidak membuktikan apa pun.

## Semantik assert

| Kunci | Arti |
|---|---|
| `assert_contains` | semua token wajib muncul (DAN). Token bertanda `\|` = daftar sinonim, cukup salah satu |
| `assert_any_of` | daftar grup; lolos bila **minimal satu grup** cocok penuh |
| `tolerance_pct` | hanya berlaku untuk token numerik di `assert_any_of`. `0` = cocok persis |

Angka tidak pernah ditaruh di `assert_contains` karena di sana toleransi tidak berlaku.

Beberapa test sengaja **tanpa angka**: yang diuji apakah agent menyatakan keterbatasan data dengan
benar. Perilaku itu tidak menua ketika data bertambah.

## Isi `note`

Tiap `note` memuat SQL yang menghasilkan angka assert, kode filter beserta tabel dan kolomnya,
sebab jebakannya, dan — untuk kelas B — apa yang membuat versi lama gagal.

## Folder

| Folder | Kelas | Test | Menguji |
|---|---|--:|---|
| `01-aturan-baru-balai-dan-epoch` | A | 14 | Cacah penandaan untuk satu balai pada satu tahun; menguji normalisasi nama balai dan filt... |
| `02-bulan-dan-balai` | A | 14 | Cacah penandaan untuk satu balai pada satu tahun; menguji normalisasi nama balai dan filt... |
| `03-kesimpula-dan-gap-pusat-upt` | A | 14 | Cacah penandaan untuk satu bulan; menguji pembatasan rentang tanggal yang benar. |
| `04-kesimpulan-penilaian-pusat-dan-kesimpulan-pe` | A | 14 | Cacah penandaan untuk satu nilai `kesimpulan_penilaian_balai`; menguji pemetaan istilah k... |
| `05-komoditi-dan-kom-produk-panga` | A | 14 | Silang komoditi dengan periode; menguji apakah kedua komponen pertanyaan masuk ke filter. |
| `06-komoditi-dan-peringkat-upt` | A | 14 | Silang `komoditi` dengan periode; menguji kelengkapan filter dari pertanyaan majemuk. |
| `07-rekap-laporan-upt-dan-tahun` | A | 13 | Regresi dari pertanyaan nyata sistem lama; angka diverifikasi ulang ke database. |
| `08-x-dan-timeline-ketepatan-waktu` | A | 13 | Cacah penandaan untuk satu tahun; menguji pemilihan kolom tanggal kanonik. |

## Catatan pemeliharaan

Angka kelas A akan bergeser seiring ETL. `verification_date` menandai kapan diverifikasi;
`tolerance_pct` menyerap pergeseran wajar. Bila sebuah test gagal, periksa dulu apakah datanya
yang bergerak atau jawabannya yang salah — jangan langsung menurunkan toleransi.
