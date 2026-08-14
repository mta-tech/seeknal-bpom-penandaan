Kolom catatan — dan cara menjawab pertanyaan "UPT yang tidak mengisi catatan".

## Kolom

`catatan` berisi keterangan bebas yang menyertai penilaian — misalnya alasan sebuah label dinilai
tidak memenuhi ketentuan.

## ⚠️ Sentinelnya string kosong, bukan SQL NULL

> `WHERE catatan IS NULL` **mengembalikan nol baris**. Yang benar:
> `coalesce(trim(catatan),'') = ''`.

Menulisnya dengan `IS NULL` adalah kesalahan yang paling sering terjadi pada pertanyaan ini, dan
hasilnya nol baris yang terlihat seperti "semua UPT sudah mengisi".

## ⚠️ Bentuk anti-join hampir selalu kosong

Pertanyaan *"tampilkan nama UPT yang tidak melakukan input catatan TMK"* biasanya diterjemahkan
menjadi anti-join: UPT yang **tidak pernah** mengisi catatan pada baris TMK.

Himpunan itu praktis kosong — setiap UPT punya minimal satu baris bercatatan.

> Nol baris di sini **adalah jawaban yang benar**, tetapi jarang berguna. Bentuk yang menjawab
> maksud penanya adalah **peringkat porsi**: per UPT, cacah baris TMK dan berapa persen di
> antaranya tanpa catatan, diurutkan dari porsi tertinggi.

Sampaikan keduanya: "tidak ada UPT yang sama sekali tidak mengisi; berikut yang porsi kosongnya
tertinggi."

## Konteks yang wajib disertakan

Porsi baris TMK tanpa catatan **tinggi secara nasional** — ini bukan penyimpangan segelintir UPT
melainkan pola menyeluruh. Menyajikan peringkat tanpa menyebut konteks nasionalnya membuat UPT
teratas terlihat menyimpang padahal ia hanya sedikit di atas rata-rata.

> **Aturan:** sertakan angka nasional sebagai pembanding sebelum menyajikan peringkat per UPT.

## Jangan pakai `catatan` sebagai kolom vonis

Pada generasi skema lama, kolom bernama sama pernah memuat nilai vonis. **Di skema sekarang ia teks
bebas.** Aturan atau SQL warisan yang memfilter `catatan` dengan nilai vonis sudah tidak berlaku.

## Rute

- Menyebut vonis → **seberang** `20-vonis.md`.
- Menyebut target/capaian per UPT → **seberang** `85-target-capaian.md`.
