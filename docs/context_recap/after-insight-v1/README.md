# Varian `after-insight-v1` — domain `penandaan`

**Dibuat:** 14 Agustus 2026 · **Status:** belum dijalankan terhadap suite.
**Basis:** seluruh temuan di `docs/temuan_database/` — profiling live, eksekusi 43 pair
`context_stores`, dan 8 `SQL Training` dari `BPOM User Relevant Query`.
**Pola penyusunan:** mengikuti `seeknal-bpom-neo/docs/context_recap/after-chart-route/route-context-070826-v6`
— dokumen orkestrator yang **merutekan dan menggerbang**, aturan data dipecah ke halaman topik kecil.

Menggantikan struktur v1 (`predikat.md` + `filter_code_reference.md` + `data_architecture.md` +
`workflow_guide.md` + `forecast_guide.md`), bukan menambahinya.

---

## Struktur

```
after-insight-v1/
├── SEEKNAL_ASK.md          orkestrator: PAGE MAP + Gate 0-5
├── seeknal_agent.yml       hanya blok prompt.custom yang diubah dari v1
├── context/                11 halaman topik
│   ├── 00-menghitung.md            WAJIB tiap pertanyaan data
│   ├── 10-komoditi.md              dimensi yang menentukan siapa yang menilai
│   ├── 20-vonis.md                 dua kolom vonis + tiga jebakan yang mengubah jawaban
│   ├── 30-status-dan-alur.md       label dipinjam domain lain, id di luar fakta
│   ├── 40-produk-dan-produsen.md   produsen vs pendaftar, nomor surat bukan komoditi
│   ├── 50-waktu-dan-durasi.md      tanggal tahap, ketepatan waktu, durasi kosong
│   ├── 60-catatan.md               sentinel string kosong, anti-join yang selalu kosong
│   ├── 85-target-capaian.md        join beda kapitalisasi, peringkat yang bias volume
│   ├── 90-kualitas-data.md         string kosong sebagai sentinel, satu nilai dua penulisan
│   ├── forecast_guide.md           SALINAN VERBATIM dari v1 — aturan forecast tidak diubah
│   └── 95-batas-domain.md          tiga pertanyaan tersering yang kolomnya tidak ada
└── skills/
    ├── penandaan-analyst/    DITULIS ULANG (v2.0.0)
    ├── penandaan-visualize/  salinan verbatim dari v1
    ├── penandaan-forecast/   salinan verbatim dari v1
    └── detect-anomaly/       salinan verbatim dari v1
```

**Aturan chart, ekspor S3, forecast, dan anomaly tidak diubah sama sekali.** Ketiga skill disalin
byte-identik dari v1, dan blok terkait di `SEEKNAL_ASK.md` Gate 0 & Gate 5 disalin verbatim.
Pada `seeknal_agent.yml`, hanya `prompt.custom` yang diubah; blok `agent`, `sources`, dan
`agent_harness` byte-identik dengan v1 (terverifikasi lewat diff).

---

## Prinsip yang membedakannya dari v1

**1. Merutekan, bukan menimbun.** `00-menghitung.md` selalu dibuka; sisanya hanya bila kondisinya
menyala.

**2. Mengajarkan pemetaan, bukan angka.** Halaman-halaman ini **tidak memuat satu pun cacah baris,
persentase, atau nilai agregat**. Angka bergeser tiap ETL; context berangka menua menjadi salah dan
mengundang agent menjawab dari ingatan.

**3. Sentinel diperlakukan sebagai jebakan utama, bukan catatan kaki.** Domain ini memakai **string
kosong** sebagai penanda kosong, dan itu penyebab dua kesalahan paling sering: angka "gap" yang
melebihi kenyataan, dan daftar "UPT tanpa catatan" yang kosong. Aturannya ditulis di tiga tempat —
Gate 5, `20-vonis.md`, dan `60-catatan.md`.

**4. "Tidak terdefinisi" dibedakan dari "nol".** Untuk sebagian komoditi, UPT tidak pernah merekam
penilaian — sehingga gap balai-pusat **tidak punya definisi**, bukan bernilai nol. Perbedaan ini
menentukan jawaban pertanyaan yang paling sering diajukan di domain ini.

**5. Batas domain ditulis di depan.** Tiga konsep yang paling sering ditanyakan — jenis kemasan,
kategori produk, klaim promosi — **tidak punya kolom sama sekali**, dan masing-masing punya kolom
lookalike yang menggoda. `95-batas-domain.md` menamainya beserta bahaya spesifik tiap substitusi.

---

## Temuan yang menjadi alasan tiap halaman

| Halaman | Temuan yang mendasarinya |
|---|---|
| `00-menghitung` | `id` unik penuh (berbeda dari domain pengawasan); `nomorsampel` bukan kunci; log dan timeline jauh lebih luas dari fakta |
| `10-komoditi` | Untuk sebagian komoditi kolom vonis balai kosong seluruhnya — bukan sebagian, melainkan setiap baris; ini mengunci pertanyaan gap dan pertanyaan "kesimpulan UPT" |
| `20-vonis` | Sentinel string kosong membuat `IS NOT NULL` dan `balai <> pusat` melaporkan seluruh baris belum-dinilai sebagai gap; satu gradasi tersimpan dalam dua kapitalisasi; satu nilai kolom pusat terbukti tahap proses, bukan putusan |
| `30-status-dan-alur` | `status_label` menyalin kamus istilah domain pengujian sehingga narasi alur menjadi tidak masuk akal; ada satu nilai langkah salah ketik |
| `40-produk-dan-produsen` | Awalan nomor surat tidak memetakan satu-ke-satu ke komoditi; produsen dan pendaftar peran berbeda dengan keterisian tidak merata |
| `50-waktu-dan-durasi` | Kolom durasi kosong pada tahap yang belum tercapai; banyak pertanyaan berbasis tanggal tahap, bukan status |
| `60-catatan` | Sentinelnya string kosong sehingga `IS NULL` selalu nol baris; anti-join "UPT tidak mengisi" praktis kosong; porsi tanpa catatan tinggi secara nasional sehingga peringkat butuh konteks |
| `85-target-capaian` | Join gagal karena beda kapitalisasi; unit pusat tidak bertarget; peringkat mentah bias terhadap balai bervolume besar |
| `90-kualitas-data` | String kosong sebagai sentinel; satu nilai dua penulisan; schema dimension basi |
| `95-batas-domain` | Tiga pertanyaan tersering meminta kolom yang tidak ada; kolom wilayah sudah dihapus dari skema; kemiripan dengan domain pengawasan paling menjebak |

---

## Yang wajib diperiksa saat pilot

| Metrik | Gerbang |
|---|---|
| PASS suite yang ada | **tidak turun** |
| Jawaban pada skenario yang sudah benar | **nol yang bergerak** |
| SQL per turn | **tidak naik** |
| Pertanyaan gap balai-pusat | menyaring sentinel dan menyebut komoditi yang tidak terdefinisi |
| Filter keluarga TMK | pola awalan **dan** case-insensitive |
| Pertanyaan kemasan / kategori / klaim | dijawab NOT COVERED, **bukan** diganti nama produk atau komoditi |
| Peringkat "UPT tidak mengisi catatan" | disajikan sebagai porsi dengan konteks nasional |

⚠️ Dua hal yang tidak akan tertangkap suite mana pun: apakah kolom vonis yang dipilih benar ketika
keduanya sama-sama menghasilkan angka yang masuk akal, dan apakah jawaban NOT COVERED benar-benar
diberikan alih-alih substitusi yang terlihat rapi. Keduanya perlu pembacaan manual.
