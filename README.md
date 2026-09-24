# Trade Data Discrepancy Analysis — Poland's Mirror Trade Statistics (2024)

## Business question

When Poland reports exporting €X to Germany, and Germany reports importing €Y from Poland
for the same year and product category, X and Y almost never match exactly. This project
investigates why — comparing how the same trade flow gets reported differently by the two
countries involved ("mirror statistics"), and whether that gap is a data quality problem or
a structural, expected feature of how trade statistics are compiled.

The analytical angle draws directly on the author's background in customs and logistics
coordination.

## Data sources

| Source | Coverage | Endpoint | Auth |
|---|---|---|---|
| UN Comtrade | Poland vs. Germany, Netherlands, UK, USA — total + HS2-chapter level, both directions | `comtradeapi.un.org/data/v1/get/C/A/HS` | API key required (free registration) |
| Eurostat / Comext | Poland vs. Germany, Netherlands only (intra-EU) — CN8 product level, aggregated to HS2 | `ec.europa.eu/eurostat/api/comext/dissemination/sdmx/2.1/data/DS-045409` | None |

- **Reference year:** 2024
- **Access date:** 11–17 September 2026
- **HS/CN hierarchy:** Comtrade queried at HS2-chapter level directly; Eurostat queried at
  CN8 (8-digit) product level and aggregated to HS2 by taking the first 2 digits, to make the
  two sources comparable at the same granularity.
- **Currency:** Comtrade reports in USD; Eurostat/Comext reports in EUR. Eurostat figures are
  converted to USD at a fixed rate of 1.0824 USD/EUR (2024 annual average) wherever the two sources are
  compared directly. Source: [ECB Data Portal, annual USD/EUR reference rate](https://data.ecb.europa.eu/data/datasets/EXR/EXR.A.USD.EUR.SP00.A).
- **Country scope:** Germany and Netherlands (intra-EU, Intrastat-based reporting) chosen for
  depth, contrasted against UK and USA (extra-EU, customs-declaration-based reporting). UK was
  not extended to HS2-level depth, since post-Brexit it shares USA's reporting mechanism —
  a UK breakdown would duplicate the USA analysis without testing a new hypothesis.

## Methodology

Full detail is in the "Methodology & Limitations" section of the notebook. In short: for every
mirror pair, `gap` is the signed exporter-minus-importer difference; `discrepancy_pct` expresses
that as a percentage of the average of the two figures. Chapter-level rankings apply a $100M
minimum value floor before sorting by percentage, to avoid small-value chapters producing
misleadingly large percentage swings.

## Key findings

1. Mirror discrepancies range from ~1.4% (UK) to ~45.6% (Poland→USA) — no partner or direction
   is consistently near zero.
2. Germany and Netherlands both show large chapter-level gaps despite modest aggregate
   discrepancies; Netherlands additionally shows a pronounced direction asymmetry not present
   for Germany.
3. Electrical machinery (HS85) and vehicles (HS87) are the largest absolute-dollar contributors
   across partners. Apparel, footwear, and toys (HS61/62/64/95) show the most extreme relative
   Comtrade-versus-Eurostat import-side divergence; the platforms use different partner-country
   concepts for intra-EU imports (origin versus consignment), a documented mechanism consistent
   with this pattern but not tested at shipment level here.
4. Comtrade and Eurostat report Poland's own export declarations almost identically (under ~3%
   apart), but diverge substantially on the partner's import-side figure for the same flow —
   showing that the import-side outputs are not directly comparable without accounting for their
   different partner-country concepts, and without establishing which one is more accurate.

## Repository structure

```
.
├── README.md
├── requirements.txt
├── .env                                    # COMTRADE_KEY — not committed, see .gitignore
├── trade_data_comtrade_eurostat.ipynb      # main analysis notebook
└── hs2_*.csv                               # per-partner/flow HS2 checkpoint files (Comtrade)
```

## Reproducing this analysis

1. `pip install -r requirements.txt`
2. Create a `.env` file with `COMTRADE_KEY=your_key_here` (free key from the UN Comtrade
   developer portal — no key needed for the Eurostat/Comext side).
3. Run the notebook top to bottom. HS2-chapter pulls checkpoint to CSV per reporter/partner/
   flow as they go — if the Comtrade daily quota is hit mid-run, re-running the same cell
   resumes from the last saved chapter rather than losing progress or re-spending quota on
   chapters already fetched.
4. Comtrade's free-tier key has a daily call-volume quota; a full run across all four partners
   at HS2-chapter granularity can approach that limit in a single day.

## Limitations

See the "Methodology & Limitations" section in the notebook for the full discussion: valuation
basis (FOB vs. CIF), Intrastat exemption thresholds, reporting timing differences, and
re-export/country-attribution effects (notably for the Netherlands, given Rotterdam's role as a
re-export hub) are all plausible contributors to the discrepancies observed here, and are not
individually isolated or tested in this analysis.
