# Water Quality Dashboard

**An interactive multi-sample compliance dashboard for drinking water analysis**

Built from original laboratory data. Eight water samples, six analytical techniques, three regulatory frameworks and one Python web application.

---

## About this project

This dashboard was built from original laboratory data collected across a multi-session analytical chemistry study of eight unknown contaminated water samples. Six complementary analytical techniques were applied across five laboratory sessions. The data existed as scattered results across multiple reports and this project consolidates it into a single automated compliance tool.

The application does what a real laboratory information management system would do: checks results against legal drinking water limits, compares outputs between analytical methods, flags QC issues automatically, validates the dataset through ionic balance analysis, and generates a dated compliance report. Built entirely in Python, deployed publicly, and every architectural decision is documented.

---

## The dataset

Eight water samples (A through H) were each characterised using:

| Technique | Analytes targeted |
|---|---|
| Ion chromatography (IC) | F, Cl, NO3, SO4, Br |
| Ion-selective electrodes (ISE) | F, NO3 |
| Conductimetric titration | SO4 |
| Potentiometric titration (AgNO3) | Cl, Br |
| Flame photometry | Na, K |
| Atomic absorption spectrometry (AAS) | Cu, Zn |

---

## Dashboard features

**Tab 1: Compliance Overview**
Traffic-light compliance matrix across all samples and analytes. Red for exceedance, amber for flagged or uncertain results, green for passing, grey for no applicable limit or below detection. The full survey is visible at a glance.

**Tab 2: Inter-Method Agreement**
Grouped bar charts showing all method results side by side per analyte, with percentage difference annotated. Bland-Altman plots where sufficient paired data exists. The 53% discrepancy between IC and AgNO3 potentiometry for chloride in Sample G illustrates why cross-validation between methods is analytically essential.

**Tab 3: QC Flag Log**
Complete table of all active flags with analyte, method, flag code, and plain-language explanation. The non-Nernstian flag on nitrate ISE includes the electrochemical reasoning: a measured slope of 22.9 mV/decade against a theoretical 59.2 mV/decade.

**Tab 4: Sample Profile**
Radar chart of detected concentrations normalised to their most stringent applicable limit, toggleable by sample. Contamination fingerprinting across the full survey becomes immediately visible.

**Tab 5: Report Generator**
Generates a dated PDF compliance report on demand: sample ID, analytical conditions, compliance summary, QC flag log, method agreement summary, and a recommended value column with documented reasoning for each selection.

---
 
## Ionic balance analysis
 
Water is electrically neutral: the total charge from all cations must equal the total charge from all anions. The dashboard calculates this automatically for every sample, converting all concentrations to milliequivalents per litre (meq/L) and computing the ionic balance error (IBE):
 
```
IBE (%) = (sum cations - sum anions) / (sum cations + sum anions) x 100
```
 
| IBE range | Status | Interpretation |
|---|---|---|
| < +/- 5% | Acceptable | Dataset is internally consistent |
| +/- 5-10% | Investigate | Possible missing ion or measurement uncertainty |
| > +/- 10% | Problematic | Significant error or missing analyte likely |
 
For Sample G, IBE = +10.9%. Zinc at 85 mg/L (2.60 meq/L) contributes 33% of the total cation load: an extraordinary proportion for a trace metal that drives the imbalance and flags the dataset for deeper investigation.
 

---

## QC flag system

Flags are assigned at the data layer, not the visualisation layer. This mirrors how a real LIMS operates: QC information travels with the data, not around it.

| Flag | Meaning |
|---|---|
| `BDL` | Below detection limit |
| `NON_NERNSTIAN` | ISE slope outside 54-60 mV/decade: result unreliable |
| `HIGH_DISC` | Inter-method discrepancy greater than 20% |
| `APPROX` | Approximate value: elevated uncertainty |
| `EXCLUDED` | Data point excluded from calibration |

---

## Regulatory frameworks

Results are checked against three frameworks simultaneously.

**WHO**: Guidelines for Drinking Water Quality, 4th edition (2022 addendum). The global scientific reference. Not legally binding anywhere, but the evidence base on which most national legislation is built.

**EU Directive 2020/2184**: The recast Drinking Water Directive, transposed into member state law by January 2023. The sulfate parametric value of 250 mg/L is notably stricter than the WHO guideline of 500 mg/L. The dashboard surfaces this difference explicitly rather than collapsing the two into a single limit.

**French national law**: Implemented via the Arrete du 11 janvier 2007 and subsequent updates. Regional enforcement is carried out by ARS (Agences Regionales de Sante). Public monitoring data is published in SISE-Eaux, the national drinking water quality database.

---

## Method priority hierarchy

When multiple methods exist for the same analyte, the compliance checker applies a defined priority order to select the recommended value. This hierarchy is recorded in every generated report, so every recommended value is fully traceable.

1. Ion chromatography (IC): multi-analyte, minimal interferences, reference method
2. Potentiometric titration (AgNO3): reliable for halides
3. Conductimetric titration: reliable for sulfate
4. Flame photometry: good for Na and K, matrix-sensitive
5. AAS: excellent accuracy, single-element
6. ISE:  useful but interference-prone, applied only when no higher-priority result exists and no QC flag is active
7. Flagged ISE results are never used as a recommended value regardless of priority

---

## Project structure
 
```
water-quality-dashboard/
|-- README.md
|-- requirements.txt
|-- app.py
|-- data/
|   |-- raw/
|   |   `-- sample_results.csv
|   `-- limits/
|       |-- who_limits.csv
|       |-- eu_dwi_limits.csv
|       `-- french_limits.csv
|-- src/
|   |-- __init__.py
|   |-- data_loader.py
|   |-- compliance_checker.py
|   |-- ionic_balance.py
|   |-- method_comparator.py
|   |-- qc_engine.py
|   `-- report_generator.py
|-- notebooks/
|   `-- exploratory_analysis.ipynb
|-- tests/
|   |-- test_data_loader.py
|   |-- test_compliance_checker.py
|   `-- test_qc_engine.py
`-- reports/
    `-- (auto-generated PDF output)
```
 
---

## Tech stack

| Library | Purpose |
|---|---|
| pandas | Data loading, cleaning, merging |
| Plotly | Interactive charts |
| Dash | Web application framework |
| scipy | Statistical tests, Bland-Altman, Lin's CCC |
| fpdf2 | PDF report generation |
| pytest | Testing |

---

## Build milestones

| Step | Description | Status |
|---|---|---|
| 1 | Repository scaffold, requirements.txt, folder structure | ✅ Complete |
| 2 | Data files: sample_results.csv and all three limits CSVs | ✅ Complete |
| 3 | data_loader.py — ingest, validate, BDL handling, derive Ca/Mg from hardness | ⬜ Next |
| 4 | pytest tests for data_loader.py | ⬜ Not started |
| 5 | compliance_checker.py — full compliance matrix with method priority | ⬜ Not started |
| 6 | qc_engine.py — all flags including HIGH_IBE | ⬜ Not started |
| 7 | ionic_balance.py — meq/L conversion, IBE, interpretation | ⬜ Not started |
| 8 | method_comparator.py — percentage difference, Bland-Altman, Lin's CCC | ⬜ Not started |
| 9 | Dash app skeleton and Tab 1 (compliance matrix) | ⬜ Not started |
| 10 | Tabs 2 through 5 | ⬜ Not started |
| 11 | report_generator.py — PDF output from button click | ⬜ Not started |
| 12 | README final polish, methodology section, screenshot | ⬜ Not started |
| 13 | Deployment to Render — public URL | ⬜ Not started |
---

## Status log

| Date | Update |
|---|---|
| May 2026 | Project scoped. Full dataset confirmed for Sample G. Ionic balance calculated: IBE +10.9%. Data collection template distributed to collaborating groups. Repository initialised. |
| May 2026 | Steps 1 and 2 complete. Folder scaffold built. All four data files created: sample_results.csv, who_limits.csv, eu_dwi_limits.csv, french_limits.csv. Next: data_loader.py. |

---

## Data contributors

| Sample | Contributors | Status |
|---|---|---|
| G | Berducou V., McCallum E., Raymond M. | Complete |
| A | TBC | Pending |
| B | TBC | Pending |
| C | TBC | Pending |
| D | TBC | Pending |
| E | TBC | Pending |
| F | TBC | Pending |
| H | TBC | Pending |

---

## References

1. World Health Organisation. *Guidelines for Drinking Water Quality*, 4th edition + 2022 addendum. Geneva: WHO Press. https://www.who.int/publications/i/item/9789240045064

2. European Parliament. Directive 2020/2184/EC on the quality of water intended for human consumption (recast). *Official Journal of the European Union*, 2020. https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32020L2184

3. Ministere de la Sante. Arrete du 11 janvier 2007 relatif aux limites et references de qualite des eaux brutes et des eaux destinees a la consommation humaine. Journal Officiel de la Republique Francaise, 2007.

4. ISO 10304-1:2007. Water quality — Determination of dissolved anions by liquid chromatography of ions. Geneva: ISO.

5. ISO 10523:2008. Water quality — Determination of pH. Geneva: ISO.

6. ISO 9963-1:1994. Water quality — Determination of alkalinity. Geneva: ISO.

7. ISO 6058:1984. Water quality — Determination of calcium content. Geneva: ISO.

8. ISO 9964-1:1993. Water quality — Determination of sodium and potassium by flame photometry. Geneva: ISO.

9. ISO 8288:1986. Water quality — Determination of cobalt, nickel, copper, zinc, cadmium and lead by flame atomic absorption spectrometry. Geneva: ISO.

---

## Author

**Emma McCallum**


Environmental Data Science | Analytical Chemistry | Research


[github.com/ecmccallum](https://github.com/ecmccallum)

---

