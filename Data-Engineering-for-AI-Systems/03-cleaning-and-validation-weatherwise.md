# Session notes — Cleaning & validation (WeatherWise lab)

*From cleaning walkthrough notebook + Session 1 slides + Lab 2.*

---

## 🟢 Mindset: defects are silent

~**1 in 3 rows** had issues in the course raw file — **nothing crashed**. Bad data **quietly** hurts models.

**Routine per defect:**

1. **Detect** (measure, don’t guess)  
2. **Decide** fix / flag / drop  
3. **Fix** (smallest change that works)  
4. **Verify** (re-run check → count → 0)  
5. **Log** (audit in JSON report)  

**Principles:**

- Never edit **raw** in S3 — work on a copy  
- **Order matters** (e.g. unit fix **before** range check)  
- Flag imputed values so they aren’t mistaken as real measurements  
- **Cleaning fixes; validation proves**

---

## 🟢 S3 paths (lab)

| Artifact | Path pattern |
|----------|--------------|
| Raw | `s3://<bucket>/raw/weather/weather_raw.csv` |
| Clean | `s3://<bucket>/clean/weather/weather_clean.csv` |
| Report | `s3://<bucket>/clean/reports/validation_report.json` |

---

## 🟡 Nine defect types (course curriculum)

| # | Defect | Symptom |
|---|--------|---------|
| 1 | Inconsistent station IDs | `S01` vs `s01` |
| 2 | Two date formats | `DD/MM/YYYY` vs ISO |
| 3 | Duplicate rows | Same station+date twice |
| 4 | Sentinel values | `-999` = missing |
| 5 | Hidden unit change | Fahrenheit in Celsius column |
| 6 | Impossible values | Humidity 140% |
| 7 | Missing values | Blanks in temp/wind |
| 8 | Target leakage | `rain_tomorrow` built using future info |
| 9 | Validation gate | Automated rules before save |

---

## 🟡 Profile before fixing

Check:

- **dtypes** (`date` as `object` = red flag)  
- **min/max** per numeric column  
- **missing counts**  
- **duplicate** keys  
- **distinct** station IDs (10 vs 5 real stations)  

---

## 🟡 Example fixes (logic, not memorizing code)

| Issue | Fix idea |
|-------|----------|
| Station case | `str.upper()` |
| Dates | Parse to single ISO format |
| Duplicates | Drop on `(station_id, date)` |
| -999 sentinel | Replace with NaN, then impute/flag |
| °F in °C column | Convert when values implausible for Celsius |
| Range violations | Clip or drop per data contract |
| Missing | Impute with median/interpolate + `was_imputed` flag |
| Target | Build `rain_tomorrow` only from **next day** label rules, no peeking |

**Checkpoint from lab:** **7/7** validation checks pass; **~5,466** clean rows.

---

## 🟡 Validation as code (Glue DQ example)

Rules encode the **data contract**:

- `station_id` complete, primary key with `date`  
- `humidity_pct` between 0 and 100  
- `temp_c` realistic range  
- `completeness` thresholds  
- `rain_tomorrow` ∈ {0, 1}  

**Alternatives:** Great Expectations, Deequ, pandera.

**Fail fast:** stop pipeline when rules break.

---

## 🟡 Why unit fix before range check

If Fahrenheit sits in a Celsius column, range checks mislead you. Convert units **first**, then validate ranges.

---

## 🔴 Building `rain_tomorrow` without leakage

**Leakage** = using information from the future when predicting the future.

Safe approach:

- Sort by station + date  
- Define tomorrow’s rain from **next row’s** measured rain (per station)  
- Drop last row per station where “tomorrow” doesn’t exist  

---

## Lab 2 study steps (if notebook overwhelmed you)

1. Set `BUCKET` only — run config cell  
2. Load raw → print `shape`, `head`, `dtypes`  
3. Run profile cells — write defects in your own words  
4. Run cleaning cells **one at a time** — note `log` dict growing  
5. Run validation — aim for 7/7  
6. Upload clean CSV + JSON report to S3  

---

## Checkpoint

Name two defects that look fine in a spreadsheet but break ML.

**Next:** [04-labeling-storage-versioning-search.md](04-labeling-storage-versioning-search.md)
