Kualitas data — sentinel, kekosongan bermakna, dan teks bebas.

## 1. Sentinel: string kosong, bukan SQL NULL

Kolom vonis balai dan kolom catatan memakai **string kosong** sebagai penanda kosong.

> `IS NULL` / `IS NOT NULL` **tidak menyaring apa pun** pada kolom-kolom itu. Yang benar:
> `kolom <> ''` atau `coalesce(trim(kolom),'') = ''`.

Ini penyebab dua kesalahan yang paling sering terjadi di domain ini: angka "gap" yang melebihi
kenyataan (`20-vonis.md`) dan daftar "UPT tanpa catatan" yang kosong (`60-catatan.md`).

Tabel pendamping (log, timeline) punya SQL NULL pada sebagian kolom — jadi aturannya berbeda antar
tabel. Periksa per kolom, jalur **P2**.

Domain BPOM lain memakai ejaan sentinel yang berbeda lagi (`95-batas-domain.md`).

## 2. Kekosongan yang berkorelasi dengan makna = FILTER TERSEMBUNYI

| Kolom | Kosongnya berarti |
|---|---|
| `kesimpulan_penilaian_balai` | komoditi yang balainya memang tidak menilai — `10-komoditi.md` |
| `catatan` | penilai tidak mengisi keterangan — pola menyeluruh, bukan penyimpangan |
| `produsen` / `pendaftar` | komoditi yang tidak merekam peran itu |
| `ed_nie` | komoditi yang tidak merekam masa berlaku |
| kolom tanggal di timeline | tahap itu belum tercapai — `50-waktu-dan-durasi.md` |

**Cara mengenalinya pada kolom baru:** tanyakan **apa arti kosongnya**. Bila kosong berarti "tidak
berlaku bagi kelompok X", menyaringnya membuang kelompok X. Silangkan keterisiannya dengan
`komoditi` — satu query.

## 3. Satu nilai, dua penulisan

Kolom vonis balai memuat gradasi yang sama dengan kapitalisasi berbeda; kolom langkah alur di log
memuat satu nilai salah ketik. Keduanya membuat pencocokan persis melewatkan sebagian baris.

> **Bandingkan case-insensitively** pada kolom vonis, dan periksa daftar nilai langkah alur untuk
> menemukan salah ketik sebelum mengelompokkan.

## 4. Kunci yang bukan kunci

`nomorsampel` **tidak unik** — satu sampel bisa dinilai penandaannya lebih dari sekali. Pakai `id`.

## 5. Populasi log dan timeline jauh lebih luas dari fakta

Selisihnya besar, bukan marginal. Sifat id tambahan belum dipastikan — jangan menyimpulkan
sebabnya, dan selalu join dari fakta (`30-status-dan-alur.md`).

## 6. Label status dipinjam dari domain lain

`status_label` di log memakai kamus istilah domain pengujian. Untuk menceritakan alur, pakai
`trx_steps` (`30-status-dan-alur.md`).

## 7. Nomor surat bukan penanda komoditi

Awalan nomor surat tidak memetakan satu-ke-satu ke komoditi (`40-produk-dan-produsen.md`).

## 8. Schema `dimension`

Database ini punya schema kedua berisi proyeksi nilai distinct. **Isinya tidak sinkron** dengan
tabel utama — ia melewatkan nilai baru tanpa error. **Jangan memakainya untuk menemukan nilai.**

## 9. Menyebutkan cakupan di jawaban

Bila kolom yang dipakai hanya terisi untuk sebagian komoditi, **sebutkan porsinya sebelum
menyajikan peringkat atau persentase**. Satu baris kalimat cukup.

## Rute

- Kembali ke aturan hitung: buka `00-menghitung.md`.
- Menyebut batas domain: buka `95-batas-domain.md`.

---

<!-- MANIFES
tabel: -
kolom: catatan, ed_nie, kesimpulan_penilaian_balai, komoditi, nomorsampel, pendaftar, produsen, status_label, trx_steps
nilai: -
-->
