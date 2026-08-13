# 02 — Arsitektur Database

> 6 tabel public + 4 tabel dimension. ERD, grain, peran, relasi implicit.

---

## Lapisan Dimensional

```
                    ┌─────────────────────────────────────────┐
                    │       LAPISAN REFERENSI (WHO/WHERE)      │
                    │  coverage_balai (668)   target_balai(532)│
                    │  siapa awasi mana        target 2024      │
                    └───────────────┬──────────────────────────┘
                                    │ nama_balai (natural key)
                    ┌───────────────▼──────────────────────────┐
                    │       FACT CORE — "APA HASILNYA"          │
                    │       mv_penandaan (292.598)              │
                    │       1 baris = 1 sampel + keputusan final│
                    └───────┬──────────────────────┬───────────┘
                            │ id                    │ id
              ┌─────────────▼──────────┐  ┌─────────▼────────────┐
              │ AUDIT — "SIAPA-KAPAN"  │  │ TIMELINE — "BERAPA   │
              │ mv_penandaan_log       │  │  LAMA per tahap"     │
              │ (3.533.299)            │  │ mv_penandaan_timeline│
              │ 1 baris = 1 event      │  │ (500.717) 1:1        │
              └────────────────────────┘  └──────────────────────┘
                            │
              ┌─────────────▼──────────┐
              │ ROLLUP — "RINGKASAN"   │
              │ mv_penandaan_agg(94.683)│
              └────────────────────────┘
```

**Tiga tabel `mv_*` (fact/log/timeline) = 3 sudut pandang atas objek yang sama:** satu "penandaan". `id` di fact = `id_penandaan` di log & timeline.

---

## Inventaris Tabel

### Schema `public` (sumber kebenaran)

| Tabel | Baris | Grain | Peran |
|---|---|---|---|
| `mv_penandaan` | **292.598** | 1 baris = 1 sampel penandaan | **FACT CORE**. Jawab "produk ini lulus/tidak?" |
| `mv_penandaan_log` | **3.533.299** | 1 baris = 1 event workflow | **AUDIT**. Jawab "siapa proses, kapan, catatan apa?" |
| `mv_penandaan_timeline` | **500.717** | 1 baris = 1 sampel lifecycle | **TIMELINE**. Jawab "berapa lama tiap tahap?" |
| `mv_penandaan_agg` | **94.683** | rollup periode×komoditi×balai×kesimpulan | **ROLLUP** dashboard (hindari scan 292k) |
| `coverage_balai` | **668** | 1 baris = 1 balai×1 kabupaten | Dimensi geografi. 88 balai ↔ 514 kabupaten |
| `target_balai` | **532** | 1 baris = 1 balai×1 komoditi×1 tahun | Dimensi KPI. **2024 saja**. 76 balai × 7 komoditi |

### Schema `dimension` (SNAPSHOT STALE — bukan sumber kebenaran)

| Tabel | Baris | vs public | Catatan |
|---|---|---|---|
| `dimension.mv_penandaan` | 218.472 | tertinggal 74.126 | Tanpa id, tanpa tanggal — export beku |
| `dimension.mv_penandaan_log` | 33.137 | subset kecil | Tanpa id_penandaan, status_code |
| `dimension.coverage_balai` | 513 | 81 balai (vs 88) | Tanpa id_balai |
| `dimension.target_balai` | 76 | pivot | Tanpa kolom target_* numerik |

> ⚠️ **`dimension` = snapshot BI yang di-flatten** (buang PK, tanggal, kode). Karena tak ada id, tak bisa di-join balik ke public. **Selalu pakai `public` sebagai sumber kebenaran.** Lihat [11_data_quality_anomali.md](11_data_quality_anomali.md) §dimension-stale.

---

## Relasi Implicit (TIDAK ada FK di schema)

Semua join bersifat **logical** — harus diingat, tidak diharapkan dari schema.

```
mv_penandaan.id  ──1:N──▶  mv_penandaan_log.id_penandaan
                            (208.119 orphan — id log tak ada di fact, era 2020-2022)

mv_penandaan.id  ──1:1──▶  mv_penandaan_timeline.id_penandaan
                            (208.119 orphan yang sama)

mv_penandaan.nama_balai  ──N:1──▶  coverage_balai.nama_balai
                                   (83/83 = 100% match)

mv_penandaan.nama_balai  ──N:1──▶  target_balai.nama_balai
                                   (54/83 = 65% match — 29 balai tanpa target 2024)
                                   ⚠ casing mismatch: target "Kosmetika" vs fact "KOSMETIKA"

mv_penandaan.komoditi  ──N:1──▶  target_balai.komoditi
                                  (case mismatch sama — pakai lower())
```

### Kebijakan Orphan (KRITIS)

Log & timeline punya **208.119 `id_penandaan` yang tak ada di fact** (era 2020-2022, fact mulai 2023). Pilihan:

| Skenario | Pola SQL | Hasil |
|---|---|---|
| **Fact-scoped (default)** | `FROM fact JOIN log ON log.id = fact.id` | Excludes orphan |
| **Full history** | `FROM log` (tanpa join fact) | Includes orphan |
| **Investigasi orphan** | `FROM log WHERE NOT EXISTS (fact match)` | Hanya orphan |

> Selalu sebut scope di jawaban: "Termasuk data 2020-2022" atau "Terbatas 2023+".
> Lihat detail orphan di [11_data_quality_anomali.md](11_data_quality_anomali.md).

---

## Naming Lie: `mv_*` = Tabel Biasa, BUKAN Materialized View

Semua 6 relasi di schema `public` punya `relkind='r'` (regular table) — verifikasi:
```sql
SELECT relname, relkind FROM pg_class c JOIN pg_namespace n ON n.oid=c.relnamespace
WHERE n.nspname='public' AND relkind IN ('r','m','v');
```

**Konsekuensi:**
- Tidak ada `REFRESH MATERIALIZED VIEW` — isi di-update oleh ETL lewat kolom `sync`.
- Tidak ada index, FK, atau constraint — semua join logical.
- Jangan asumsi `id` adalah PK hanya karena namanya.

---

## Validasi Integritas Rollup (agg vs fact)

| Sumber | Total |
|---|---|
| `mv_penandaan` (fact) | 292.598 |
| `agg` periode `day` | 292.598 ✅ |
| `agg` periode `month` | 292.598 ✅ |

**agg identik sempurna dengan fact** di grand total — satu-satunya tabel yang integritasnya terbukti. Tapi agg hanya me-rollup era 2023+ (mewarisi bias scope fact). Lihat [13_konsistensi_integritas.md](13_konsistensi_integritas.md).

---

## Database Connection

```
WAREHOUSE_URL=postgresql://postgres:***@localhost:5533/penandaan
```

- PostgreSQL, port 5533 (via SSH tunnel ke host)
- Schema default: `public`
- User: `postgres` (full read access untuk `public` + `dimension`)
- Snapshot `sync = 2026-08-11 22:53:20` seragam di semua tabel utama

> Live SQL authoritative — ETL bisa refresh harian. Angka di dokumen ini = anchor snapshot, bukan kebenaran abadi.

---

Lanjut ke [03_anatomi_per_tabel.md](03_anatomi_per_tabel.md) untuk detail kolom per tabel.
