# Katalog Nilai Unik — database `penandaan` (live 2026-08-13)

Dihasilkan dengan `GROUP BY` penuh atas seluruh baris (bukan sampel `categories` KAI).
Angka absolut bergeser karena ETL harian; **struktur nilainya** yang stabil.
Tanda ⟵ menandai nilai sentinel (kosong / `-` / `Null` / SQL NULL).

---
### mv_penandaan

  **komoditi** — 8 nilai unik
      `KOSMETIKA` = 78,550 (26.8%)
      `PRODUK PANGAN` = 75,629 (25.8%)
      `OBAT` = 53,175 (18.2%)
      `OBAT TRADISIONAL (OT)` = 39,189 (13.4%)
      `ROKOK` = 32,081 (11.0%)
      `SUPLEMEN KESEHATAN` = 10,757 (3.7%)
      `OBAT KUASI` = 2,688 (0.9%)
      `KEMASAN PANGAN` = 689 (0.2%)

  **kesimpulan_penilaian_balai** — 6 nilai unik
      `MK` = 177,440 (60.6%)
      `` = 85,257 (29.1%) ⟵ kosong/sentinel
      `TMK` = 14,140 (4.8%)
      `TMK MINOR` = 7,424 (2.5%)
      `TMK MAYOR` = 6,607 (2.3%)
      `TMK Minor` = 1,890 (0.6%)

  **kesimpulan_penilaian_pusat** — 5 nilai unik
      `MK` = 151,777 (51.8%)
      `TMK` = 76,547 (26.1%)
      `VP` = 59,831 (20.4%)
      `TMK MAYOR` = 2,367 (0.8%)
      `TMK MINOR` = 2,236 (0.8%)

### mv_penandaan_log

  **trx_steps** — 19 nilai unik
      `draft` = 576,586 (16.3%)
      `pusat` = 536,586 (15.2%)
      `spv_1` = 516,563 (14.6%)
      `kepala_balai` = 489,702 (13.9%)
      `spv_1_pusat` = 359,921 (10.2%)
      `direktur` = 354,181 (10.0%)
      `selesai` = 350,804 (9.9%)
      `spv_2_pusat` = 251,602 (7.1%)
      `spv_2` = 56,843 (1.6%)
      `ditolak_spv_1` = 38,479 (1.1%)
      `ditolak_pusat` = 1,461 (0.0%)
      `ditolak_spv_1_pusat` = 1,416 (0.0%)
      `ditolak_kepala_balai` = 637 (0.0%)
      `ditolak_spv_2` = 78 (0.0%)
      `ditolak_spv_2_pusat` = 63 (0.0%)
      `ditolak_direktur` = 43 (0.0%)
      `<SQL NULL>` = 7 (0.0%) ⟵ kosong/sentinel
      `receive_tps` = 2 (0.0%)
      `spv1` = 1 (0.0%)

  **status_label** — 12 nilai unik
      `Operator - Draft Sampling` = 576,135 (16.3%)
      `MT - Pembuatan SPK` = 536,084 (15.2%)
      `Supervisor - Verifikasi` = 518,409 (14.7%)
      `TPS - Penerimaan SPU` = 490,508 (13.9%)
      `Deputi MT - Pembuatan SPK` = 359,974 (10.2%)
      `Penguji - Entri Hasil Pengujian` = 354,065 (10.0%)
      `Sampel Rujukan Selesai` = 350,735 (9.9%)
      `Penyelia - Pembuatan SPP` = 251,411 (7.1%)
      `Supervisor 2 - Verifikasi` = 56,841 (1.6%)
      `<SQL NULL>` = 40,020 (1.1%) ⟵ kosong/sentinel
      `Operator - Perbaikan Sampel` = 758 (0.0%)
      `Kepala Balai` = 35 (0.0%)
