Komoditi — dimensi yang menentukan kolom vonis mana yang tersedia.

## Kolom `komoditi`

Berisi golongan produk yang labelnya dinilai. Nilainya huruf besar, mencakup obat, kosmetika,
produk pangan, obat tradisional, suplemen kesehatan, obat kuasi, rokok, dan kemasan pangan.

Domain ini adalah **satu-satunya** yang memakai golongan kemasan pangan secara berarti. Ambil
daftar nilainya bila ragu — jalur **P2**.

## ⚠️ Ejaan berbeda dari domain pemeriksaan

Domain ini memakai **`KOSMETIKA`** dan **`OBAT TRADISIONAL (OT)`**; domain pemeriksaan memakai
`KOSMETIK` dan `OBAT TRADISIONAL`. **Jangan menyalin daftar komoditi antar domain.**

## ⚠️ Komoditi menentukan siapa yang menilai

Ini kekhasan paling penting domain ini:

> **Untuk sebagian komoditi, UPT/balai tidak pernah merekam penilaian sama sekali** — kolom vonis
> balai kosong untuk seluruh barisnya, dan yang ada hanya penilaian pusat.

Konsekuensinya berantai:

| Pertanyaan | Untuk komoditi yang balainya menilai | Untuk komoditi yang balainya tidak menilai |
|---|---|---|
| "kesimpulan UPT" | dijawab dari kolom balai | **tidak terdefinisi** |
| "gap balai vs pusat" | dihitung normal | **tidak terdefinisi**, bukan nol |
| "kesimpulan akhir/pusat" | dijawab dari kolom pusat | dijawab dari kolom pusat |

**Aturan:** setiap pertanyaan yang menyentuh penilaian UPT atau gap **wajib menyebut komoditi**,
atau dijawab per komoditi dengan menyatakan mana yang tidak terdefinisi.

Cara memastikan komoditi mana: silangkan keterisian kolom balai dengan `komoditi` — satu query,
jalur **P2**. Lakukan ini **sebelum** menjawab pertanyaan gap; jangan mengandalkan ingatan.

## Komoditi juga memengaruhi sebaran vonis pusat

Satu nilai vonis yang berarti "sedang diverifikasi" terkonsentrasi pada komoditi tertentu, dan
hampir tidak muncul pada komoditi lain. Detailnya di `20-vonis.md`.

## Istilah pengguna

| Istilah | Tindakan |
|---|---|
| "obat" / "obat-obatan" | golongan obat saja, atau termasuk tradisional/kuasi/suplemen? **tanya** |
| "OT; suplemen; obat kuasi" | tiga golongan, sertakan ketiganya |
| "pangan" | golongan produk pangan; perhatikan ada juga golongan kemasan pangan yang terpisah |
| **"kategori produk"** | **bukan komoditi** dan **tidak ada kolomnya** — `95-batas-domain.md` |

## Rute

- Menyebut vonis → **seberang** `20-vonis.md`.
- Menyebut produk/produsen → **seberang** `40-produk-dan-produsen.md`.
