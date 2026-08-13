# 16 — Informasi Eksternal Dibutuhkan

> Yang TIDAK bisa dijawab dari database — butuh dokumen/sistem/sumber lain BPOM.

---

## Inventaris Informasi Hilang

Database e-Penandaan merekam **proses administratif** sangat detail, TAPI buta terhadap hulu (kenapa diperiksa) dan hilir (apa akibatnya). Informasi berikut hidup di dokumen kebijakan/sistem lain BPOM.

---

## 1. Kamus Kode Regulasi (catatan)

| Kode di DB | n | Komoditi dominan | Arti (perlu konfirmasi) | Sumber |
|---|---|---|---|---|
| MKL.02 | 2.247 | KOSMETIKA | Klaim butuh data dukung? | Peraturan BPOM |
| MKL.01 | 671 | KOSMETIKA | | |
| TME.05 | 127 | ? | | |
| TME.09 | 117 | ? | | |
| TME.11 | 122 | ? | | |
| TME.14 | 80 | ? | | |
| NIK.01 | 61 | ? | Terkait NIE? | |

> **Tanpa kamus ini**, kita hanya tahu "ada pelanggaran pasal X" tanpa tahu pasal X tentang apa. Decoding kode = prerequisite untuk audit pelanggaran substantif.

**Sumber:** Peraturan BPOM, SOP e-Penandaan, atau database kode regulasi internal BPOM.

---

## 2. Definisi & SOP Resmi VP/MK/TMK

| Konsep | Tafsir data kita | Perlu konfirmasi |
|---|---|---|
| MK | "Memenuhi Ketentuan" — label comply | ✅ jelas |
| TMK | "Tidak Memenuhi Ketentuan" — violation | ✅ jelas |
| VP | Inferensi pola: "verifikasi ulang" / "keranjang keraguan" | ❓ **butuh definisi resmi** |
| TMK MAYOR / MINOR | Gradasi keparahan (PANGAN only) | ❓ butuh SOP kapan mayor vs minor |

**Pertanyaan kritis untuk BPOM:**
- Apa definisi resmi VP? Apakah benar "perlu verifikasi lanjutan"?
- Mengapa VP PANGAN tidak pernah selesai? Adakah SOP verifikasi lanjutan?
- Kapan sebuah pelanggaran diklasifikasi MAYOR vs MINOR?

**Sumber:** SOP e-Penandaan, Pedoman Penilaian Penandaan BPOM.

---

## 3. SLA Target per Tahap Workflow

| Tahap | Median aktual | Target SOP? |
|---|---|---|
| mulai_kabalai | 6 hari | ❓ |
| **kabalai_direktur** | **21 hari** | ❓ **butuh target** |
| direktur_pusat | 0 hari | ❓ |

> Kita bisa ukur durasi aktual (21 hari), tapi **tak tahu berapa seharusnya** tanpa SOP SLA. Bottleneck "21 hari" — apakah overload atau within SLA?

**Sumber:** SOP workflow penandaan, Peraturan SLA BPOM.

---

## 4. Target Multi-Tahun (2023, 2025, 2026)

`target_balai` hanya punya **2024**. Untuk KPI 2023/2025/2026:

| Tahun | Target | Status |
|---|---|---|
| 2023 | ❌ Tidak ada | Butuh backfill |
| 2024 | ✅ Ada | Terlampaui 101-116% |
| 2025 | ❌ Tidak ada | Butuh backfill |
| 2026 | ❌ Tidak ada | Butuh backfill |

**Sumber:** Renstra BPOM, DIPA tahunan, dokumen perencanaan direktorat.

---

## 5. Master Kode Balai (3-digit di nomorsampel)

`nomorsampel` prefix `yy.bbb.` mengandung kode balai 3-digit. 84% prefix konsisten 1:1 ke nama_balai, tapi 12 prefix ambigu (kode 094→3 balai, 9.297 sampel).

**Butuh:**
- Tabel master resmi kode balai 3-digit ↔ nama balai
- Riwayat reorganisasi balai (kode dipakai ulang kapan?)

**Sumber:** Master data balai BPOM Pusat, riwayat struktur organisasi.

---

## 6. Mengapa PANGAN Workflow Terputus? (ROOT CAUSE aplikasi)

Data tunjukkan **gejala**: 0% completion PANGAN, pipa terputus setelah MT. **Penyebab** di aplikasi/SOP BPOM:

- Adakah state 5/6/7 untuk PANGAN di konfigurasi workflow?
- Apakah direktorat pangan punya tim Deputi/Penyelia/Penguji?
- Apakah ada SOP verifikasi lanjutan VP yang tak pernah dijalankan?
- Apakah ini bug aplikasi ATAU kebijakan eksplisit (PANGAN sengaja tidak selesai)?

> **Ini temuan paling material yang tak bisa dijawab dari DB.** Butuh akses aplikasi e-Penandaan ATAU wawancara tim Direktorat Peredaran Pangan Olahan.

**Sumber:** Tim IT e-Penandaan, Direktorat Peredaran Pangan Olahan BPOM.

---

## 7. Singkatan Direktorat (Kepanjangan)

| Singkatan di DB | Kepanjangan (perlu konfirmasi) |
|---|---|
| KMEI ONPPZA | ❓ |
| OTSK | ❓ |
| ONPP | ❓ |
| Distribusi dan Pelayanan ONPP | ❓ |

> Decoding struktur organisasi BPOM Pusat.

**Sumber:** Struktur organisasi BPOM, SK Sekretariat.

---

## 8. Hasil Uji Laboratorium

Workflow menyebut status 7 "Penguji - Entri Hasil Pengujian" (190.104 event), TAPI fact tidak menyimpan hasil uji lab.

**Pertanyaan:**
- Ke mana perginya hasil uji?
- Apakah ada sistem terpisah (LIMS BPOM)?
- Catatan di log status-7 = satu-satunya jejak?

**Sumber:** Sistem LIMS BPOM, direktorat pengujian.

---

## 9. Tindak Lanjut TMK

Setelah TMK, apa yang terjadi pada produk? Database berhenti di "keputusan":

- Produk ditarik dari peredaran?
- Produsen diperingatkan?
- Sanksi administratif?
- Tidak ada konsekuensi yang terekam

> **Efektivitas pengawasan ada di hilir yang tak terekam.** Tidak bisa klaim "pengawasan efektif" hanya dari data ini.

**Sumber:** Sistem penindakan BPOM, direktorat penyidikan.

---

## 10. Pemicu Pemeriksaan (pengaduan vs rutin)

Tidak ada field "sumber pemeriksaan". Tak tahu apakah pemeriksaan dipicu:
- Pengaduan masyarakat
- Sampling rutin
- Risalah koordinasi
- Audit tematis

> Mempengaruhi interpretasi: TMK tinggi pada pengaduan = berbeda dengan TMK tinggi pada rutin.

**Sumber:** Aplikasi sumber (RPO), modul pengaduan BPOM.

---

## 11. Geolokasi Produk

`coverage_balai` tahu yurisdiksi balai (84 balai ↔ 514 kabupaten), TAPI tidak tahu **di kabupaten mana produk ditemukan beredar**.

> Balai pemeriksa ≠ lokasi produk. Untuk peta pelanggaran geografis, butuh kolom lokasi penemuan.

**Sumber:** Aplikasi sumber, modul sampling lapang.

---

## 12. Gap yang User Tanyakan tapi Butuh Data Eksternal (baru)

Validasi pertanyaan user (export KAI) menambah daftar kebutuhan eksternal:

| # | Gap user | n pertanyaan | Kebutuhan eksternal |
|---|---|---|---|
| 12.1 | **Jenis/nama kemasan** (botol/blister/sachet) | 3–5 | Master atribut kemasan produk — TIDAK ada di DB penandaan |
| 12.2 | **Kabupaten/kota/provinsi produsen** | 2–4 | Master produsen → alamat → wilayah (belum ada di DB) |
| 12.3 | **Media publikasi / klaim iklan / golongan obat** | 7 | Data domain `pengawasan` (DB lain) |
| 12.4 | **e-labeling / peserta pilot** | 1 | Daftar peserta pilot e-labeling (sistem terpisah) |

> Detail: [20_gap_schema_user](20_gap_schema_user.md). Ini kebutuhan baru di luar 11 daftar di atas — berasal dari
> analisis pertanyaan user, bukan dari analisis internal.

---

## Prioritas Pengumpulan Dokumen Eksternal

| Prioritas | Dokumen | Untuk |
|---|---|---|
| 🔴 Kritis | SOP workflow PANGAN + wawancara direktorat pangan | Root cause temuan #1 |
| 🔴 Kritis | Struktur organisasi + wawenang approver | Root cause temuan #2 |
| 🟠 Tinggi | Kamus kode MKL/TME/NIK | Decode pelanggaran |
| 🟠 Tinggi | SOP VP/MK/TMK + Mayor/Minor | Konfirmasi tafsir |
| 🟠 Tinggi | **Master atribut kemasan + alamat produsen** (gap G1/G2 user) | Jawab pertanyaan user |
| 🟡 Sedang | SLA per tahap | Konteks durasi |
| 🟡 Sedang | Target 2023/2025/2026 | KPI lengkap |
| 🟡 Sedang | Master kode balai | Validasi silang |
| ⚪ Rendah | Singkatan direktorat | Housekeeping |
| ⚪ Rendah | Hasil lab / tindak lanjut / pemicu / geolokasi | Cakupan luas |

---

Lanjut ke [17_rencana_investigasi_lanjut.md](17_rencana_investigasi_lanjut.md) untuk SQL siap eksekusi.
