# Session notes — Cleaning & validation (WeatherWise lab)

*From cleaning walkthrough notebook + Session 1 slides + Lab 2.*

Cleaning changes data so that it means what the column name says. Validation proves the change worked. They are not the same step, and neither one is optional because the model “might still train.”

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

**What “silent” means:** Pandas will happily average a humidity of 140 and a temperature of −999. There is no exception. The loss still goes down. The model learns the defect as if it were weather.

**Why measure before you fix:** Guessing “there are a few bad dates” is how a second date format survives. Count the mismatches, then fix, then count again and expect zero.

**The three decisions:**

| Decision | When | What you write down |
|----------|------|---------------------|
| **Fix** | The true value is recoverable, such as `s01` → `S01` or °F → °C | The rule |
| **Flag** | You filled a gap and downstream must know it was filled | A boolean column such as `temp_was_imputed` |
| **Drop** | The row cannot be trusted and cannot be repaired, such as a duplicate or an impossible value your contract forbids | The count and the reason |

**Why the smallest fix:** A clever imputation can invent weather. Uppercasing a station id cannot. Prefer the repair that does not add new information.

**JSON report:** A machine-readable log (`validation_report.json`) of what you found and what you did. **JSON** means JavaScript Object Notation, a text format of keys and values. The point is an audit, not the syntax. If the question “what changed?” cannot be answered from the report, the lab is incomplete.

**Imputed:** A value the pipeline filled in, as opposed to a value the sensor measured. Leaving it unmarked teaches the model that the filled number was observed.

---

## 🟢 S3 paths (lab)

| Artifact | Path pattern |
|----------|--------------|
| Raw | `s3://<bucket>/raw/weather/weather_raw.csv` |
| Clean | `s3://<bucket>/clean/weather/weather_clean.csv` |
| Report | `s3://<bucket>/clean/reports/validation_report.json` |

**What a URI is telling you:** `s3://` is the storage service, the next piece is the **bucket** (the container), and the rest is the **key** (the path). The bucket in the lab is yours; the prefixes `raw/` and `clean/` are the zones from the lifecycle notes.

**Why the report sits next to clean and not inside raw:** The report describes a transform. It is not what the station sent. Raw stays untouched so you can rerun.

**Where a path bug hurts:** Writing clean data back onto the raw key. Versioning can save you, but the habit is still wrong. The destination prefix should say what the file is.

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

### What each defect means and how it fools you

| # | What it is | Why a spreadsheet looks fine | What the model does with it |
|---|------------|------------------------------|-----------------------------|
| 1 | The same station under two strings | Both look like ids | Two stations, split history, broken lags |
| 2 | Dates as text in more than one layout | They are still readable to you | Sort order is wrong; “03/04/2024” is 3 April or 4 March depending on the parser |
| 3 | The same observation twice | The numbers agree, so nothing looks broken | That day counts double in the loss and may land in both train and test |
| 4 | A **sentinel**, a code pretending to be a measurement. −999 means “missing,” not a temperature | The column is numeric, so charts still draw | The mean temperature collapses, and −999 becomes a pattern |
| 5 | A unit change that was not written down | 95 looks like a number in a temperature column | The model learns that 95 °C happens on Earth at that station |
| 6 | A value outside physical or contract range | One cell among thousands | The average and the tree splits chase an impossibility |
| 7 | A blank | Easy to see in a small file, easy to miss in a large one | Most models cannot multiply a blank. An imputer fills it; if the fill is silent, missingness itself is erased. Sometimes “sensor down” is the signal. |
| 8 | The target or a feature uses information from after the decision | The column `rain_tomorrow` looks like a proper label | Offline scores jump and live scores do not, because the future is not available yet |
| 9 | No automated gate | The notebook “looks done” | The next run saves a bad file because nobody rechecked |

**ISO** for dates means the ISO 8601 form `YYYY-MM-DD`. One format, unambiguous, sorts correctly as text.

**DD/MM/YYYY** is day, month, year. A parser that assumes month-first will silently swap 5 June and 6 May for every day up to 12.

---

## 🟡 Profile before fixing

Check:

- **dtypes** (`date` as `object` = red flag)
- **min/max** per numeric column
- **missing counts**
- **duplicate** keys
- **distinct** station IDs (10 vs 5 real stations)

**What a profile is:** A description of the file before you change it. It is the “detect” step written down.

**dtype** means data type. `object` in pandas usually means “text or mixed.” A date that is still `object` has not been parsed. An id that is `int` may have lost a leading zero.

**Why min and max:** Humidity max 140 tells you defect 6 immediately. Temperature max 100 tells you to suspect Fahrenheit. Temperature min −999 tells you there is a sentinel. You do not need a model to see these.

**Why distinct station ids:** If the contract says 5 stations and the file has 10 strings, casing or typos created phantom stations. Count before you upper-case, and count after, and expect the contract number.

**How a profile prevents a bad fix:** If you impute before you notice −999, you fill real blanks with a median that was dragged down by the sentinel. Order is part of profiling.

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

**NaN** means “not a number,” the marker for a missing numeric value. Replacing −999 with NaN is what makes missingness visible to an imputer. Leaving −999 makes it look like a real cold day.

**Clip** means pull values back inside a range (141% humidity becomes 100%). Clipping hides the defect and creates a pile of values exactly on the boundary. **Drop** removes the row. The contract decides. Do not clip a target. Do not clip a value that is illegal because the unit is wrong; convert the unit first.

**Median vs mean for imputation:** A median resists a few extreme leftovers. A mean is dragged toward them. Neither is a measurement. Flag the fill.

**Interpolate** means estimate a missing point from its neighbors. In time series, neighbors in the future are leakage if they would not have been known yet. Prefer a forward fill from the past, or mark the gap and let the model see that the sensor was down.

**Why ~5,466 is a checkpoint and not a magic constant:** It is the row count after this lab’s drops and deduplication. If you are far from it, a filter was too aggressive or duplicates remain. Do not add or delete rows to force the number. Find which defect step diverged.

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

**What a validation gate is:** Code that refuses to publish `clean/` when a rule fails. A human reading a chart is not a gate. A gate has a pass/fail result and stops the next step.

**Primary key:** The column or columns that identify one row. Here, one station on one date is one observation. A duplicate key means the key is not doing its job.

**Completeness:** The share of non-missing values. The story in the scoping notes asked for at least 95% in places. Put the number you agreed in the rule, not a number you discover after looking at the score.

| Tool | What it is |
|------|------------|
| **Glue Data Quality** | Rules that run as part of AWS Glue |
| **Great Expectations** | A Python library of data tests and reports |
| **Deequ** | A data-quality library on Spark, originally from Amazon |
| **pandera** | A Python library that checks pandas (and similar) frames against a schema |

**Why fail fast:** A downstream training job will not know the humidity rule failed. It will train. Stopping is cheaper than a model built on 140% humidity.

**Where gates get bypassed:** Someone writes the clean file from the notebook “just this once” and skips the check. The gate has to be the only path that is allowed to write the curated prefix.

---

## 🟡 Why unit fix before range check

If Fahrenheit sits in a Celsius column, range checks mislead you. Convert units **first**, then validate ranges.

**What goes wrong if you reverse them:** 98.6 °F is a plausible human temperature and an implausible air temperature in a Celsius column that you expected between, say, −20 and 50. If you drop “out of range” values first, you delete hot Fahrenheit days and keep Fahrenheit values that happen to fall inside the Celsius band, such as 40 °F stored as 40. The file looks valid and is still wrong.

**How the repair should work:** Detect values that cannot be Celsius (or a recorded cutover date when a station changed units). Convert \(C = (F - 32) \times 5/9\). Then run the range check. Log how many values were converted.

**Where this generalizes:** Currency in paise versus rupees, wind in knots versus metres per second, timestamps with and without offsets. Any hidden unit change should be repaired before a range or a z-score. Scaling a mixed-unit column freezes the mistake into the mean.

---

## 🔴 Building `rain_tomorrow` without leakage

**Leakage** = using information from the future when predicting the future.

Safe approach:

- Sort by station + date
- Define tomorrow’s rain from **next row’s** measured rain (per station)
- Drop last row per station where “tomorrow” doesn’t exist

**What the label means:** At the end of day D, “did it rain on day D+1?” The features may use measurements from day D and earlier. The label uses the rain observed on D+1. When you actually predict, D+1 has not happened, so the label is unknown and the features must already be knowable.

**Why you sort before the shift:** “Next row” is only “next day” if the frame is sorted by station and date. An unsorted shift pairs S01’s Monday with S02’s Thursday and calls it tomorrow.

**Why you drop the last row:** The last day at each station has no tomorrow inside the file. Filling it with 0 would teach the model that the end of the dataset is always dry.

**How people leak anyway:** Computing the label from a column that already contains tomorrow’s rain under another name. Using a three-day average centered on today, which includes tomorrow. Fitting the scaler on the whole frame after the label exists, which is a softer leak but still a leak.

**Where to check:** Pick one station, print three consecutive days, and confirm that `rain_tomorrow` on Monday equals measured rain on Tuesday, and that Monday’s features do not include Tuesday’s humidity.

---

## Lab 2 study steps (if notebook overwhelmed you)

1. Set `BUCKET` only — run config cell
2. Load raw → print `shape`, `head`, `dtypes`
3. Run profile cells — write defects in your own words
4. Run cleaning cells **one at a time** — note `log` dict growing
5. Run validation — aim for 7/7
6. Upload clean CSV + JSON report to S3

**Why this order matches the routine:** Steps 2 and 3 are detect. Step 4 is decide and fix, one defect at a time so you can see the log grow. Step 5 is verify. Step 6 publishes the copy, not the raw file.

**What `shape` tells you:** `(rows, columns)`. Write it down before drops so you can explain where the rows went.

**What 7/7 means:** Seven encoded contract checks all passed. A 6/7 with a saved file is a failed lab even if the model would still fit.

---

## Checkpoint

Name two defects that look fine in a spreadsheet but break ML.

**Answer:** Any two of these, with the reason. Sentinel −999 looks like a very cold day and drags every average. Fahrenheit values that sit inside the Celsius range look plausible and train the wrong relationship. Duplicate station-days look consistent and double-count. A `rain_tomorrow` column looks like a proper target and may contain the future. Mixed date strings look readable and sort in the wrong order. Phantom station ids (`s01` versus `S01`) look like two valid codes and split one history.

**Next:** [04-labeling-storage-versioning-search.md](04-labeling-storage-versioning-search.md)
