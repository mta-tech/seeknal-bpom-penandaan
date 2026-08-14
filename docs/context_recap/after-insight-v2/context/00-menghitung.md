Cara menghitung — entity, grain, tanggal kanonik, eksklusi. Berlaku untuk SETIAP pertanyaan data.

Domain ini merekam **pengawasan penandaan/label produk**: penilaian label pada produk yang
disampling. Bukan pengawasan iklan, bukan pemeriksaan sarana, bukan pengujian laboratorium.
Batasnya di `95-batas-domain.md`.

## 1. Entity

| Subjek | Hitung dengan |
|---|---|
| Penandaan | `COUNT(*)` atau `COUNT(DISTINCT id)` |
| Surat | `COUNT(DISTINCT nomor_surat)` dengan sentinel dibuang |
| Produk unik | `COUNT(DISTINCT nama_produk)` |
| Produsen / pendaftar unik | `COUNT(DISTINCT ...)` — teks bebas, `40-produk-dan-produsen.md` |

`id` **unik penuh** — satu baris satu penandaan. `COUNT(*)` dan `COUNT(DISTINCT id)` memberi hasil
sama, keduanya sah. Ini berbeda dari domain pengawasan iklan, yang barisnya bisa berulang per id.

> **"Berapa total penandaan" masih bisa ambigu** antara penandaan dan surat — satu surat memuat
> banyak penandaan. Sebutkan yang dipakai, atau tanya.

PENTING: `nomorsampel` **bukan kunci** — ada nomor yang muncul lebih dari sekali. Satu sampel bisa dinilai
penandaannya lebih dari sekali. Jangan memakainya sebagai pengenal unik.

## 2. Tanggal kanonik

`tgl_start` adalah tanggal mulai — **default** untuk periode dan tren. `tgl_end` untuk pertanyaan
penyelesaian.

Tanggal tahap persetujuan ada di tabel timeline, dengan populasi yang **jauh lebih luas** dari
tabel fakta — lihat `30-status-dan-alur.md`.

`ed_nie` adalah tanggal berakhirnya izin edar produk, **bukan** tanggal kegiatan. Jangan dipakai
untuk periode kegiatan.

## 3. Eksklusi WAJIB

| Apa | Aturan |
|---|---|
| Unit pusat pada hitungan per-balai | `nama_balai` yang berupa direktorat bukan balai |
| Sentinel pada kolom yang dihitung unik | buang sebelum `COUNT(DISTINCT ...)` |

### Uji satu kalimat sebelum menambah klausa `WHERE` apa pun

> **Apakah baris yang dibuang klausa ini bisa menjadi jawaban yang benar?**
> **Bisa itu PENYEMPITAN, dan penyempitan hanya ada bila pertanyaan memintanya.**
> **Tidak bisa itu eksklusi, dan boleh selalu.**

**Kolom yang kosong tidak lolos uji itu** — kecuali kekosongannya deterministik per komoditi, dan
itu justru harus dinyatakan, bukan disaring diam-diam (`90-kualitas-data.md`).

## 4. Bentuk angka & eksekusi

- **Angka utama dari query global sendiri**, bukan penjumlahan partisi.
- Satu statement per panggilan, tanpa `;`.
- **Jangan `EXTRACT(YEAR ...)` untuk menyaring** — pakai rentang berbatas. Tidak ada indeks.
- `ILIKE '%…%'` memindai seluruh kolom — pakai **sekali** untuk menemukan nilai, lalu hitung
  dengan nilai persis.

## 5. Tabel pendamping — arah join

| Tabel | Kunci | Arah aman |
|---|---|---|
| `mv_penandaan_log` | `id_penandaan` = `mv_penandaan.id` | **dari fakta (INNER)** |
| `mv_penandaan_timeline` | idem | **dari fakta (INNER)** |
| `mv_penandaan_agg` | tanpa id | jangan dijoin; lihat §6 |
| `target_balai`, `coverage_balai` | `nama_balai` | `85-target-capaian.md` |

> **Log dan timeline memuat jauh lebih banyak id daripada tabel fakta.** Menghitung dari sana
> langsung akan melebihi populasi penandaan secara signifikan. Selalu mulai dari fakta.

## 6. Tabel `agg` — satu syarat

`mv_penandaan_agg` menyimpan kubus pra-agregasi dengan kolom `periode_type` bernilai dua (harian
dan bulanan). **Selalu saring satu `periode_type`**; tanpa itu angkanya tergandakan.

## Rute

- Konsep berkode belum teresolusi buka peta halaman di `SEEKNAL_ASK.md`.
- Menyebut periode / durasi: buka `50-waktu-dan-durasi.md`.
- Berkata "belum / tanpa / kosong": buka `90-kualitas-data.md`.

---

<!-- MANIFES
tabel: coverage_balai, mv_penandaan, mv_penandaan_agg, mv_penandaan_log, mv_penandaan_timeline, target_balai
kolom: ed_nie, id, id_penandaan, nama_balai, nomorsampel, periode_type, tgl_end, tgl_start
nilai: -
-->
