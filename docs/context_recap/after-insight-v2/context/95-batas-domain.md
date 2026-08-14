Batas domain — apa yang TIDAK ada di database ini, dan bagaimana menjawabnya.

## Empat domain BPOM yang terpisah

| Domain | Isi | Database |
|---|---|---|
| **penandaan** (di sini) | pengawasan **label/penandaan produk** | domain ini |
| pengawasan | pengawasan **iklan** | database terpisah |
| pemeriksaan | inspeksi ke sarana/fasilitas | database terpisah |
| pengujian | sampling dan hasil uji laboratorium | database terpisah |

Keempatnya **tidak tersambung** di sini.

## Istilah yang menandai pertanyaan salah rute

| Istilah pengguna | Domain sebenarnya | Kenapa mudah tertukar |
|---|---|---|
| **iklan**, "media", "lokasi iklan", "klausul pelanggaran", "pembuat iklan" | pengawasan | **paling menjebak** — kedua domain memakai kesimpulan balai/pusat dan MK/TMK |
| **MS / TMS**, "hasil uji", "parameter uji", "LHU" | pengujian | domain ini memakai MK/TMK |
| **sarana**, "sarana distribusi/produksi diperiksa", "temuan produk", "nilai sitaan" | pemeriksaan | domain ini punya kolom produsen, tetapi itu pembuat produk — bukan sarana yang diperiksa |

> Pembedanya adalah **objek yang dinilai**: label produk di sini, materi iklan di domain
> pengawasan. Bila pertanyaan menyebut media, lokasi tayang, atau pembuat iklan — itu domain lain.

Cara memastikan bila ragu: periksa daftar kolom tabel (`information_schema.columns`). Bila tidak
ada kolom yang memuat konsepnya, itu **P5 NOT COVERED**.

## Konsep yang ditanyakan pengguna tetapi tidak ada di sini

| Diminta | Status | Catatan |
|---|---|---|
| **jenis / nama kemasan** | **kolomnya tidak ada** | ditanyakan berulang. Menjawabnya dari nama produk menghasilkan daftar nama produk yang tampak rapi — dan pembaca tidak punya cara tahu itu bukan kemasan |
| **kategori produk** | **kolomnya tidak ada** | yang terdekat adalah komoditi, dan cakupannya jauh lebih luas — bukan hal yang sama |
| **klaim dalam promosi/iklan** | **kolomnya tidak ada** | domain iklan |
| **provinsi / kabupaten** produsen | **kolomnya tidak ada** | pernah ada di generasi skema lama, kini dihapus. Satu-satunya jalur geografi adalah wilayah kerja balai — dan itu bukan alamat produsen |
| **siapa yang menyetujui** | tidak dapat dipastikan | semantik pelaku di log ambigu — `30-status-dan-alur.md` |
| **gap balai vs pusat untuk komoditi yang balainya tidak menilai** | tidak terdefinisi | bukan nol — `10-komoditi.md`, `20-vonis.md` |

Tiga baris pertama adalah pertanyaan yang **paling sering diajukan pengguna** di domain ini.
Semuanya **P5 NOT COVERED**, dan semuanya punya kolom lookalike yang menggoda untuk dipakai.

## Cara menjawab NOT COVERED

Tiga kalimat, tidak lebih:

1. sebut **apa yang ditanyakan** dan bahwa konsepnya tidak direkam di database ini;
2. sebut **di mana kemungkinan besar konsep itu berada**, bila diketahui;
3. tawarkan **hal terdekat yang benar-benar bisa dijawab**, dan sebutkan bedanya.

Yang **tidak boleh**: menjawab pertanyaan kemasan dengan nama produk, atau pertanyaan kategori
dengan komoditi. Query semacam itu jalan, hasilnya rapi, dan pembaca tidak punya cara tahu bahwa
yang ditampilkan bukan yang ditanyakan.

## Rute

- Kembali ke peta halaman `SEEKNAL_ASK.md`.
- Menyentuh kekosongan kolom: buka `90-kualitas-data.md`.

---

<!-- MANIFES
tabel: -
kolom: -
nilai: -
-->
