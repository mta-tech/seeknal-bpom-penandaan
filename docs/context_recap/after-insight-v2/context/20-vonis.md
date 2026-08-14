Dua kolom vonis — dan tiga jebakan yang masing-masing mengubah jawaban.

## Kolom

| Kolom | Menilai | Diisi oleh |
|---|---|---|
| `kesimpulan_penilaian_balai` | hasil penilaian UPT/balai | balai |
| `kesimpulan_penilaian_pusat` | hasil penilaian pusat | pusat |

Tidak ada kolom "kesimpulan akhir" terpisah di domain ini. Bila pengguna menyebut "kesimpulan
akhir", yang dimaksud hampir selalu **kolom pusat** — tetapi konfirmasi bila konteksnya ambigu.

## Jebakan 1 — sentinelnya STRING KOSONG, bukan SQL NULL

Kolom balai memakai **string kosong** sebagai penanda belum-dinilai.

> `WHERE kesimpulan_penilaian_balai IS NOT NULL` **tidak menyaring apa pun** — string kosong lolos.
> Dan karena string kosong tidak sama dengan nilai vonis mana pun, perbandingan
> `balai <> pusat` **mengembalikan TRUE** untuk seluruh baris yang balainya belum menilai.

Inilah penyebab paling umum angka "gap" yang jauh melebihi kenyataan. Aturan yang benar:
saring `kesimpulan_penilaian_balai <> ''` lebih dulu.

## Jebakan 2 — satu gradasi tersimpan dalam dua penulisan

Kolom balai memuat gradasi TMK yang **sama tetapi ditulis dengan kapitalisasi berbeda** pada
sebagian baris.

> `= 'TMK MINOR'` (atau penulisan mana pun yang persis) **melewatkan** baris bergradasi sama yang
> ditulis lain. Selalu bandingkan case-insensitively: `upper(kolom) LIKE 'TMK%'` untuk keluarga
> TMK, atau `upper(kolom) = 'TMK MINOR'` untuk gradasi tertentu.

Kolom pusat tidak punya masalah ini — nilainya bersih. Jadi aturan case-insensitive **wajib untuk
kolom balai**, dan tetap aman bila diterapkan ke kolom pusat.

## Jebakan 3 — satu nilai di kolom pusat adalah TAHAP PROSES, bukan putusan

Kolom pusat memuat satu nilai yang berarti **sedang/menunggu diverifikasi** — bukan vonis
memenuhi maupun tidak memenuhi ketentuan.

Buktinya ada di jalur waktunya: baris bernilai itu sudah sampai pusat, tetapi **hampir tidak pernah
punya tanggal proses direktur**, sementara baris ber-vonis MK/TMK hampir selalu punya.

> **Setiap hitungan MK/TMK wajib mengeluarkan nilai itu dan menyebut porsinya.** Ia terkonsentrasi
> pada komoditi tertentu, dan porsinya tumbuh dari waktu ke waktu — memasukkannya ke denominator
> membuat tingkat kepatuhan terlihat menurun padahal yang bertambah adalah antrean.

Cara mengenalinya: ambil daftar nilai kolom pusat (jalur **P2**) — nilai yang bukan MK dan bukan
keluarga TMK adalah kandidatnya. Silangkan dengan tanggal direktur di timeline untuk memastikan.

## Pertanyaan "gap balai vs pusat"

Gabungkan ketiga jebakan di atas. Populasi yang sah dibandingkan adalah baris yang:

1. kolom balainya **terisi** (bukan string kosong);
2. kolom pusatnya **sudah berkeputusan** (bukan nilai tahap proses, bukan kosong);
3. lalu bandingkan **case-insensitively**.

Dan sebelum itu semua: **periksa komoditinya**. Untuk komoditi yang balainya tidak pernah menilai,
gap **tidak terdefinisi** — bukan nol (`10-komoditi.md`).

> Jawaban yang benar untuk "gap penandaan obat" bisa berupa: *"untuk komoditi ini penilaian UPT
> tidak direkam, sehingga gap tidak terdefinisi"*. Itu jawaban, bukan kegagalan.

## Istilah pengguna

| Istilah | Cara mengikat |
|---|---|
| "kesimpulan UPT" / "hasil UPT" | kolom balai — ingat komoditinya |
| "kesimpulan akhir" / "hasil pusat" | kolom pusat |
| "memenuhi ketentuan" | vonis positif |
| "TMK" | keluarga TMK, pola awalan + case-insensitive |
| "gap" / "perbedaan" | lihat prosedur di atas |

## Rute

- Menyebut komoditi: buka `10-komoditi.md`.
- Menyebut catatan alasan TMK: buka `60-catatan.md`.
- Menyebut tanggal tahap: buka `50-waktu-dan-durasi.md`.

---

<!-- MANIFES
tabel: -
kolom: kesimpulan_penilaian_balai, kesimpulan_penilaian_pusat
nilai: -
-->
