#FORECAST GUIDE (PENANDAAN-FORECASTER) — ETS HOLT-WINTERS METHOD, SQL TEMPLATES, ELIGIBILITY GATES, AND QUALITY LABELS#

> **v1.0 — 2026-08-11.** Engine: **ETS seasonal (Holt-Winters)** — deliberate tradeoff,
> forecast varies per month at the cost of lower accuracy than a flat median (FC2f r3
> spec). Computed on-demand from `mv_penandaan_agg`, no pre-computed table.
> **Consistency is critical:** an inconsistently-built SQL across turns produces a
> visibly different answer, not just a different level — follow §2's reuse rule for
> follow-ups, and the template below exactly for new questions.

## §1 Data Source & SQL Template

| Param | Value |
|---|---|
| Table | `mv_penandaan_agg` (pre-aggregated rollup) |
| Date col | `tanggal_periode` |
| Baseline | `>= '2023-01-01'` |
| Aggregation | `periode_type = 'month'` for monthly grain |
| Grain | 1 row = 1 month × 1 komoditi × 1 balai × 1 kesimpulan |

**Production table only.** `mv_penandaan_agg` has 94.517 rows with composite key
(periode_type, tanggal_periode, komoditi, nama_balai, kesimpulan_penilaian_balai,
kesimpulan_penilaian_pusat). sum(jumlah_penandaan) = 292.169 = fact total ✅.

```sql
SELECT tanggal_periode AS x,
       SUM(jumlah_penandaan)                     AS y
FROM   mv_penandaan_agg
WHERE  periode_type = 'month'
  AND  tanggal_periode >= '2023-01-01'
  AND  tanggal_periode < date_trunc('month', CURRENT_DATE)
  AND  <series filter from §3, if any>
GROUP BY 1 ORDER BY 1
```

Default Y = `SUM(jumlah_penandaan)`; use `SUM(jumlah_sampel_unik)` if the user says
"jumlah sampel unik". Default grain = monthly. No JOINs, no `generate_series`, no
recursive CTE (tool rejects these). Univariate only.

> **Why `mv_penandaan_agg` instead of `mv_penandaan`?** The fact table has 292.169
> rows with per-sample granularity. Aggregating on-the-fly is expensive and risks
> inconsistent GROUP BY across turns. The pre-aggregated table ensures same query
> → same numbers.

## §2 Method — ETS Seasonal, Deterministic

Single fit (trend + seasonal, `seasonal_periods=12`) on a **fixed 36-month
trailing window** (non-adaptive — consistency prioritised over marginal MAPE;
shorter series just use all their data), forecast natively — never
recursive self-append. Same input → same output, always. Point **varies per
month** (intentional, see §6). Bounds: `sigma * sqrt(min(h,2))`, z=1.2816
(80%)/1.9600 (95%) — widens period 1→2, then flat. Never show `sigma`,
`sub_type`, or raw field names; say "Rentang Realistis" (80%) / "Rentang
Ekstrem" (95%).

**Follow-up consistency:** same series, different horizon/clarifying question
→ reuse the prior turn's exact SQL, don't rebuild it. Rebuilding risks a
different-but-"valid" query and a visibly different answer for what the user
considers the same question. Only rebuild for an explicitly different
series/grain/filter.

## §3 Eligibility & Series Registry

Refuse if history < 10 periods or CV > 0.8. Volume alone never refuses — it
widens the range and lowers the quality label instead.

| Series | Filter | Notes |
|---|---|---|
| Total Penandaan | none | All penandaan across all komoditi and balai |
| Per Komoditi | `komoditi = '<KOMODITI>'` | 8 series: KOSMETIKA, PRODUK PANGAN, OBAT, OBAT TRADISIONAL (OT), ROKOK, SUPLEMEN KESEHATAN, OBAT KUASI, KEMASAN PANGAN |
| Per Balai | `nama_balai = '<BALAI>'` | 83 series (one per balai) |
| MK Only | `kesimpulan_penilaian_pusat = 'MK'` | Compliant only |
| TMK Only | `kesimpulan_penilaian_pusat IN ('TMK','TMK MAYOR','TMK MINOR')` | Non-compliant only |
| VP Only | `kesimpulan_penilaian_pusat = 'VP'` | Verification needed only |

**Never eligible** (CV too high, event-driven not recurring):
- Individual balai with < 500 total penandaan (too sparse)
- KEMASAN PANGAN (only 689 total rows — insufficient history)
- Rejection events (status 990-997) — these are in log, not agg

Multi-series question → check each independently; one ineligible sub-series
doesn't block an eligible one.

## §4 Quality Label (walk-forward backtest MAPE)

| MAPE | Label | Keyakinan |
|---|---|---|
| ≤15% | BAIK | Tinggi |
| 15–25% | CUKUP | Sedang |
| 25–35% | LEMAH | Rendah — anomaly auto-attached |
| >35% | TOLAK | Rendah, strong warning — anomaly auto-attached |
| n/a | UNKNOWN | Sedang — don't overstate either way |

One label for the whole projection (MAPE is one number for the fit), even
though the point varies by month.

## §5 Output & Limitations

Read `run_forecast`'s markdown and compose prose around it, don't invent a new
structure. Never say "Forecast" (use "Proyeksi"), never show raw CV/`sigma`/
`sub_type`/`modified_z`/MAD or `window_selection` internals — a one-line
Metodologi summary is enough unless the user asks for detail.

- Seasonal shape isn't guaranteed correct — a historically-high month may not
  be high this year; the quality label is the honest trust signal, shape included.
- Bounds cap at h=2, don't keep widening — validated, not a shortcut.
- **CSV Store Contract:** one question = one stored CSV = the data behind the
  answer. On success `run_forecast` self-uploads ONE **combined** CSV (historical
  + projection in one file, `kind` column) — tool-owned and complete; the agent
  does NOT export separately. Only on refusal/fallback (no combined CSV) does the
  agent export the historical series. The 36-step horizon cap is silent in the
  tool — stating it in the answer is the agent's job.
- **Transparency & consistency (general):** every projected period appears as
  its own row (point + bounds); multi-series → per-series labelled blocks; the
  CSVs cover the same full horizon as the answer. Same question, same series,
  same horizon → identical numbers in any session (deterministic engine);
  the only legitimate difference is data drift — stamp the as-of date.

## §6 Anomaly Detection (separate tool, awareness only)

`detect_anomaly(sql)` — same 2-column contract, never removes data, never
forecasts. Auto-fires when a forecast's MAPE > 25%; or on direct request
("apakah ada anomali..."). Always state flagged points were **not removed**.

## §7 Komoditi-Specific Forecast Notes

| Komoditi | Total rows | Seasonality | Notes |
|---|---|---|---|
| KOSMETIKA | 78.432 | Moderate | Largest sample. MK dominant (80%). |
| PRODUK PANGAN | 75.367 | Strong | VP dominant (71%). Seasonal food patterns. |
| OBAT | 53.034 | Low | TMK dominant (90.5%). Less seasonal. |
| OBAT TRADISIONAL (OT) | 39.138 | Low | MK dominant (88%). |
| ROKOK | 32.078 | Moderate | Mixed MK/TMK. High ed_nie null (69.8%). |
| SUPLEMEN KESEHATAN | 10.745 | Moderate | MK dominant (93%). |
| OBAT KUASI | 2.686 | Low | Small sample. MK dominant. |
| KEMASAN PANGAN | 689 | **Insufficient** | Too small for reliable forecast. Refuse. |

## §8 Balai-Specific Forecast Notes

| Volume tier | Balai count | Forecast eligible? |
|---|---|---|
| >5.000 penandaan | ~20 | ✅ Yes — sufficient history |
| 1.000-5.000 | ~30 | ⚠️ Marginal — check CV |
| 500-1.000 | ~15 | ⚠️ Marginal — check CV |
| <500 | ~18 | ❌ No — insufficient history |

> When forecasting per balai, check eligibility for each balai independently.
> A balai with < 500 total penandaan should be refused even if the aggregate
> across all balai is eligible.
