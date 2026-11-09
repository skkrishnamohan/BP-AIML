# Class notes — Cleaning the weather file, and proving you did

*The cleaning walkthrough, Session 1 slides, and Lab 2.*

About one row in three in the course’s raw file had something wrong with it. The notebook did not crash. Pandas averaged the bad values happily. The loss would still have gone down. This class is about defects that are silent, and about the difference between repairing a value and proving the repair.

**Cleaning** changes the data so that a column means what its name says. **Validation** proves the change, and refuses to publish the file when the proof fails. They are not the same step. A model that “still trains” is not evidence that either step happened.

---

## 🟢 The routine, copied as five verbs

For every defect:

1. **Detect.** Count it. Do not guess that “there are a few bad dates.”
2. **Decide.** Fix, flag, or drop.
3. **Fix.** The smallest change that restores the meaning. A clever fill can invent weather. Uppercasing a station id cannot.
4. **Verify.** Run the same count again. Expect zero, or expect the number you deliberately left and flagged.
5. **Log.** Write what you found and what you did into the JSON report.

**JSON**, JavaScript Object Notation, is just a text format of keys and values. The format is not the point. If a later person cannot answer “what changed in this file?” from `validation_report.json`, the lab is incomplete.

Three decisions, so they do not blur:

| Decision | When | What you record |
|----------|------|-----------------|
| Fix | The true value is recoverable, such as `s01` to `S01`, or a known unit conversion | The rule, and how many cells it touched |
| Flag | You filled a gap, and downstream must not treat the fill as a measurement | A column such as `temp_was_imputed` |
| Drop | The row cannot be trusted and cannot be repaired | The count and the reason |

**Imputed** means filled in by the pipeline, not measured by the sensor. An unmarked imputation teaches the model that the filled number was observed. Sometimes the fact of missingness is the clue that the sensor was down. Erasing it is not neutral.

Three principles sit above the verbs. Never edit raw on S3. Work on a copy. Order matters: a unit repair comes before a range check. Cleaning fixes, and validation proves. A chart a human glanced at is not a proof.

---

## 🟢 Where the lab files live

| What | Path |
|------|------|
| The untouched intake | `s3://<bucket>/raw/weather/weather_raw.csv` |
| The repaired copy | `s3://<bucket>/clean/weather/weather_clean.csv` |
| The audit | `s3://<bucket>/clean/reports/validation_report.json` |

The `s3://` part names the service. The next piece is the **bucket**, your container. The rest is the **key**, the path inside it. The prefixes `raw/` and `clean/` are the zones from the last class. The report sits beside the clean file because it describes a transform. It is not what the station sent. Writing the clean data back onto the raw key is the path bug. Versioning can save you afterwards. The habit is still wrong.

---

## 🟡 The nine defects, each one as a way the spreadsheet lies

**1. Inconsistent station ids.** `S01` and `s01` are the same station to a person and two stations to a group-by. Histories split. Lag features break. The repair is to uppercase, then count distinct ids and expect the number of real stations. The course file shows the smell as more distinct strings than real stations, on the order of ten strings for five stations. Count before the fix and after it.

**2. Two date formats.** Some cells are `DD/MM/YYYY`, day then month then year. Some are ISO. **ISO 8601** for a date is `YYYY-MM-DD`. One format, unambiguous, and it sorts correctly even as text. The lie: you can read both formats, so the column looks fine. A parser that assumes month-first will swap 5 June and 6 May for every day up to the 12th, silently. A date that is still pandas dtype `object` has not been parsed. `object` usually means text or mixed. Parse to one representation, store ISO or a real datetime in UTC, and count the rows that failed to parse. That count should become zero or should become dropped rows you logged.

**3. Duplicate rows.** The same station on the same date, twice. The numbers may agree, so nothing looks broken. The day counts twice in the loss, and the two copies can land on opposite sides of a split. Drop on the key `(station_id, date)`. If the two copies disagree, you do not silently keep one. That disagreement is its own defect and goes in the log.

**4. Sentinel values.** A **sentinel** is a code pretending to be a measurement. −999 in a temperature column means “missing,” not a temperature. The column is numeric, so every chart draws it and every mean is dragged toward it. Replace −999 with **NaN**, “not a number,” the marker for a missing numeric value. Then impute or drop, and flag. If you impute before you notice the sentinel, the median itself has been poisoned.

**5. A hidden unit change.** Fahrenheit values are sitting in a column named Celsius. 95 looks like a number, so the spreadsheet is calm. The model learns that 95 °C occurs at that station. The repair is a conversion, \(C = (F - 32) \times 5/9\), applied to the values you have identified as Fahrenheit, and a log of how many cells moved.

**6. Impossible values.** Humidity of 140%. One cell among thousands. Averages and tree splits chase it. The contract says 0 to 100. Either drop the row or, if the contract allows, clip it. **Clip** means pull the value back to the boundary, so 141 becomes 100. Clipping hides the defect and piles values exactly on the boundary, which can itself become a fake pattern. Do not clip a target. Do not clip a value that is illegal only because the unit is wrong. Convert first.

**7. Missing values.** Blanks in temperature or wind. Most models cannot multiply a blank. A median is safer than a mean because a few leftovers do not drag it. Interpolation estimates a point from its neighbors. In a time series, a neighbor in the future is leakage if it would not have been known yet. Prefer a fill that uses only the past, or mark the gap and let the model see that the sensor was down. Always set `was_imputed`.

**8. Target leakage.** `rain_tomorrow` was built using information from after the decision. The column looks like a proper label. Offline scores jump. Live scores do not, because the future is not available at 6 a.m. The safe construction is its own section below.

**9. No validation gate.** The notebook looks done, and the next run saves a bad file because nobody rechecked. A gate is code that refuses to publish. It is the ninth defect because the absence of a check is itself a defect in the pipeline.

---

## 🟡 Profile first, on purpose

A profile is the detect step written down, before any repair.

Print the shape, rows and columns, and write the row count in the margin. You will need it to explain where rows went. Print dtypes. A date still stored as text is a red flag. Print min and max for every numeric column. Humidity max 140 is defect 6. Temperature max around 100 is a reason to suspect Fahrenheit. Temperature min −999 is a sentinel. Print missing counts, duplicate keys, and distinct station ids.

The order of discovery matters as much as the list. If you fill missing temperatures before you replace −999, the fill value is wrong. If you drop “out of range” temperatures before you convert Fahrenheit, you delete the obviously hot Fahrenheit days and you keep Fahrenheit values that happen to sit inside the Celsius band, such as 40 °F stored as the number 40. The file then looks valid. It is still in the wrong unit. Convert units first. Then check ranges. This generalizes to paise versus rupees, knots versus metres per second, and timestamps with and without offsets. Any hidden unit change is repaired before a range or a z-score. Scaling a mixed-unit column freezes the mistake into the mean.

---

## 🔴 Building `rain_tomorrow` without cheating

The label means: at the end of day D, did it rain on day D+1? Features may use measurements from day D and earlier. The label uses the rain observed on D+1. When you actually predict, D+1 has not happened, so the label is unknown and every feature must already be knowable.

The safe steps:

1. Sort by station and by date. “The next row” is “the next day” only after this sort. An unsorted shift can pair Monday at S01 with Thursday at S02 and call it tomorrow.
2. Define tomorrow’s rain from the next row’s measured rain, within the same station.
3. Drop the last row of each station. That day has no tomorrow inside the file. Filling it with 0 would teach the model that the end of the dataset is always dry.

People still leak after doing those three. They use a column that already contains tomorrow’s rain under another name. They compute a three-day average centered on today, which includes tomorrow. They fit the scaler on the whole frame after the label exists. Pick one station and print three days. Monday’s `rain_tomorrow` equals Tuesday’s measured rain. Monday’s features do not include Tuesday’s humidity.

---

## 🟡 The gate, and the lab’s two numbers

Rules encode the contract:

- `station_id` is complete, and together with `date` it is the primary key. A **primary key** identifies one row. A duplicate key means the key is not doing its job.
- Humidity is between 0 and 100.
- Temperature is inside a realistic Celsius range, after the unit repair.
- Completeness meets the threshold you agreed, such as 95% where the story required it.
- `rain_tomorrow` is either 0 or 1, and the rows where tomorrow does not exist are gone, not filled with a pretend 0.

**Fail fast.** Stop the pipeline when a rule breaks. The training job will not know the humidity rule failed. It will train.

Tools that can express the same rules: **Glue Data Quality** inside AWS Glue, **Great Expectations** as a Python library of data tests, **Deequ** as a quality library on Spark, and **pandera** as a schema check on pandas frames. The tool is replaceable. The habit of a failing gate is not. The gate has to be the only path allowed to write the curated prefix. A person who uploads a CSV “just this once” has bypassed the class.

The lab checkpoint is **7 of 7** checks passing, and about **5,466** clean rows. 5,466 is not a number you force by adding or deleting rows. It is the count this lab’s drops and deduplication produce. If you are far from it, a filter was too aggressive or duplicates remain. Find the step that diverged. A score of 6 of 7 with the file saved anyway is a failed lab. The model would still have fit. That is the point of the opening paragraph.

---

## The lab, if the notebook was a blur

1. Set the bucket name and run only the config cell.
2. Load raw. Print shape, the first rows, and dtypes. Write the row count down.
3. Run the profile cells. Write each defect in a sentence of your own before you read the cleaning cell that fixes it.
4. Run the cleaning cells one at a time. Watch the log dictionary grow by one reason at a time.
5. Run validation. Aim for 7 of 7.
6. Upload the clean CSV and the JSON report. Do not upload them onto the raw key.

---

## 🟡 One station, six rows, repaired in order

Write a tiny file in the margin and do not skip ahead. Station ids and temperatures are enough to see four defects at once.

| As received | What a person sees | What is actually wrong |
|-------------|--------------------|------------------------|
| s01, 01/02/2024, temp 77 | A January or February day | Lowercase id. Day-month versus month-day. 77 is not a Celsius air temperature you should trust without a question. |
| S01, 2024-02-01, temp 25 | Looks like the clean version of the row above | This may be the duplicate of that same day once both are parsed, or it may be a second report. You cannot tell until the key is normalized. |
| S01, 2024-02-02, temp -999 | A very cold day | Sentinel. Not a temperature. |
| S01, 2024-02-03, temp (blank) | A gap | Missing. Do not fill it from 4 February. |
| S02, 2024-02-03, humidity 140 | A humid day | Impossible humidity. |
| S01, 2024-02-04, temp 26 | A normal day | The last day. It has no tomorrow inside the file. |

The order we are not allowed to “optimize”:

1. Uppercase the station id. `s01` becomes `S01`. Distinct stations fall. Now the duplicate key can be seen.
2. Parse every date to ISO. `01/02/2024` becomes 2024-02-01 if the contract says day-first. If you guess month-first you have silently moved the row to 1 January. Write the assumption in the log.
3. Only now drop duplicate `(station_id, date)` keys. Doing it before parsing would keep both the text date and the ISO date, because they do not look like duplicates yet.
4. Replace −999 with NaN. The blank and the sentinel are now the same kind of hole. Flag them if you fill them. Do not fill 3 February from 4 February.
5. Ask the unit question about 77 before you range-check. If the contract says “values above 55 in this column during the known bad week are Fahrenheit,” convert 77 °F to about 25 °C, then range-check. If you range-check first and the legal band is −5 to 50, you delete 77 and you never recover the day. If 77 had been 40 °F, a Celsius range check would have kept 40, which is a cool day in Fahrenheit and a hot day in Celsius. That is the row the reversed order cannot see.
6. Humidity 140 fails the contract. Drop or clip, as the contract says, and count it. Do not average it into a “profile” you then forget.
7. Build `rain_tomorrow` only after the sort, and drop 4 February for S01 because tomorrow is not in the file.

Then run the same counts again. Duplicate keys: 0. Unparseable dates: 0. Sentinels left in the temperature column: 0. Humidity outside 0–100: 0. That second count is validation. The first count was only a profile.

The lab’s 5,466 rows is this kind of accounting on the real file. If your tiny example “ends” with the same number of rows you started with, you skipped the drop rules and called it cleaning.

## Checkpoint

Name two defects that look ordinary in a spreadsheet and say, for each, what the model will learn if you leave it.

A sentinel of −999 looks like a very cold day and drags every average and every split toward a code that means “missing.” Fahrenheit values that happen to fall inside the Celsius range look plausible and train the wrong physical relationship, while the obviously impossible Fahrenheit values may be the only ones a range check deletes. You could equally have picked duplicate station-days, which double-count, or mixed date strings, which sort in the wrong order, or `s01` versus `S01`, which splits one station’s history. Any two, with the consequence, is the answer.

**Next class:** [04-labeling-storage-versioning-search.md](04-labeling-storage-versioning-search.md)
