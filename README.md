# Aviation Accidents Analysis

A data analysis project examining U.S. aviation accident data (1948–2023) to identify aircraft makes and models with strong safety profiles, for an airline/aircraft insurer client.

## Project Goal

The client wants to know which aircraft makes/models exhibit:
- Low rates of total destruction in an accident
- Low likelihood of fatal or serious passenger injury when an accident occurs

...plus any general conditions (weather, engine type, etc.) that affect accident severity. Recommendations are split into **small aircraft** and **large aircraft (20+ passengers)**, and are restricted to makes/models plausibly still in active service (accidents from 1983 onward, assuming a ~40-year aircraft lifespan).

## Repository Structure

```
├── data/
│   ├── AviationData.csv            # Raw source data (NTSB accident records, 1948-2023)
│   └── AviationData_Cleaned.csv    # Cleaned/derived dataset produced by the cleaning notebook
├── Aviation_Accidents_Cleaning.ipynb        # Data loading, inspection, cleaning, feature engineering
├── Aviation_Accidents_Data_Analysis.ipynb   # EDA, visualizations, and recommendations
└── README.md
```

Run the cleaning notebook first — it produces `AviationData_Cleaned.csv`, which the analysis notebook depends on.

## Data Source

`AviationData.csv` — U.S. aviation accident/incident records, one row per event, ~88,889 rows and 31 columns, including aircraft make/model, injury counts, damage severity, weather conditions, engine details, and flight phase/purpose.

## Cleaning Notebook — What Was Done and Why

| Step | Decision | Rationale |
|---|---|---|
| **Aircraft.Category** | Keep only `Airplane` | Client is only interested in fixed-wing airplanes, not helicopters/gliders/balloons. **Caveat:** this field is missing for ~64% of rows, and missingness is heavily concentrated pre-2008 (not random) — the field simply wasn't consistently recorded until then. This means the filtered dataset skews toward more recent accidents. |
| **Amateur.Built** | Keep only `No` | Client wants professional builds only. Only 102 NaNs — dropped. |
| **Event.Date** | Keep accidents from 1983 onward | Assumes a 40-year max aircraft lifespan; older accidents involve aircraft unlikely to still be active. |
| **Injury columns** (Fatal/Serious/Minor/Uninjured) | Dropped 47 rows with all 4 columns NaN (no info at all); filled remaining NaNs with 0 (assumed "none of this injury type," not "unknown") | Enables estimating `Total.Passengers` per flight. |
| **Total.Passengers = 0** | Dropped 904 rows | Injury *fraction* is undefined (0/0) with no passengers recorded. |
| **Derived: Injury.Fraction** | `(Fatal + Serious) / Total.Passengers` | Normalizes injury severity across aircraft of very different sizes — a raw injury count isn't comparable between a 4-seat Cessna and a 300-seat airliner. |
| **Aircraft.damage** | Dropped `Unknown`/NaN (~6% of rows) | Can't derive a destroyed/not-destroyed flag without a known outcome. |
| **Derived: Is.Destroyed** | Binary flag: 1 if `Aircraft.damage == 'Destroyed'`, else 0 | `Substantial` and `Minor` damage are both treated as "not destroyed" — a simplification worth noting, since substantial damage is still a serious outcome. |
| **Make** | Standardized casing/whitespace, stripped corporate suffixes (`INC`, `CORP`, `CORPORATION`, etc.), fixed known aliases (e.g. `DEHAVILLAND` → `DE HAVILLAND`) | Same manufacturer was previously split across multiple string variants (e.g. `CIRRUS` vs `CIRRUS DESIGN CORP`), which would have understated its true sample size. |
| **Make threshold** | Kept only makes with ≥50 records | Ensures any make-level comparison is statistically reasonable. |
| **Model** | Dropped 11 NaNs; standardized casing/whitespace | — |
| **Derived: Make.Model** | Concatenation of cleaned `Make` + `Model` | Raw `Model` codes are **not unique across manufacturers** (e.g. `"500"` appears under 3+ different makes) — a combined identifier is needed to represent a specific airplane type. |
| **Engine.Type / Weather.Condition** | Consolidated inconsistent "unknown" labels (`UNK`, `Unk` → `Unknown`) | Cosmetic cleanup; NaNs left as-is (not required to impute). |
| **Number.of.Engines** | Dropped 3 rows with `0` engines | Implausible for a powered airplane — treated as data-entry error. |
| **Purpose.of.flight** | Merged `Air Race/show` into `Air Race show` | Same category, inconsistent formatting. |
| **Broad.phase.of.flight** | Left in dataset despite 85% missingness | Explicit decision, not an oversight — this field is a candidate "factor" for later analysis, so it's kept but excluded from robust factor comparisons given the sample-size problem. |
| **Columns dropped** | `Schedule`, `Air.carrier`, `Airport.Code`, `Airport.Name`, `Report.Status`, `Latitude`, `Longitude`, `FAR.Description`, `Publication.Date` | Either too sparse (>30-90% missing) or not relevant to the safety analysis (administrative/location fields). |

**Final cleaned dataset:** ~16,589 rows.

## Analysis Notebook — What Was Done and Key Findings

### Size Split
Aircraft split into **Small** (<20 total passengers, n=16,114) and **Large** (≥20, n=475) using `Total.Passengers`. Large aircraft accidents are rare and come from only **8 distinct manufacturers**, so "top 15" comparisons were replaced with a minimum sample floor (n≥5) for that group — a structural limit of the data, not an analysis gap.

### Make-Level Recommendations

**Small aircraft:** Maule, Bellanca, Aeronca, Champion, and Aviat Aircraft rank well on *both* injury fraction and destruction rate, each with 74–218 supporting accidents. **Cessna** is the standout — despite having the largest sample by far (6,942 accidents), it still lands in the bottom 15 for both metrics, making it the most statistically dependable recommendation in this group.

**Large aircraft:** Bombardier, Boeing, and McDonnell Douglas consistently occupy the lower-risk end on both metrics; Airbus and De Havilland rank at the higher-risk end (De Havilland's result rests on just 9 accidents — treat as suggestive only).

### Model-Level Recommendations

**Large:** Only 4 models clear the n≥10 threshold: Boeing 777, Bombardier CL-600-2B19, Boeing 767, Boeing 737. Boeing 777 shows 0% injury fraction across 16 accidents — a strong but small-sample result.

**Small:** 29 qualifying models found within the top-10 lowest-injury-fraction makes. Diamond DA 20 C1 and Maule M-5-210C show 0% injury fraction (n=11 each); models like Cessna 180H, Cessna 195, and Piper PA-32-301 offer a better balance of low risk and larger sample size (20-37 accidents).

### Contributing Factors

**Weather Condition** — the strongest signal in the dataset. Accidents in IMC (instrument/poor-visibility conditions) show a **65% injury fraction and 37% destruction rate**, vs. **24% and 7%** in VMC (clear conditions) — roughly a 3-5x difference, well-supported by sample size (869 vs 14,132 accidents).

**Engine Type** — a more nuanced, trade-off finding: Turbofan (jet) aircraft have the lowest injury rate (14%), but piston/reciprocating engines have the lowest destruction rate (8%). Turboprops perform worst on both metrics (33% injury, 19% destroyed), likely reflecting the type of flying turboprops typically do (e.g. remote or rugged operations) rather than the engine technology itself being riskier — a reminder that these are correlational findings, not causal ones.

## Key Limitations to Keep in Mind

- `Aircraft.Category` missingness is concentrated pre-2008, so the cleaned dataset skews toward more recent accidents.
- Large-aircraft findings rest on much smaller samples (8 total makes) than small-aircraft findings, and should be presented with more caution.
- Some "0% injury fraction" results are based on samples as small as n=11 — promising, but should be framed as "no incidents recorded in this sample," not "zero risk."
- Weather and engine type findings are associational, not causal — other confounding factors (e.g. mission type, pilot experience, terrain) are not controlled for here.

## Requirements

```
pandas
numpy
matplotlib
seaborn
```