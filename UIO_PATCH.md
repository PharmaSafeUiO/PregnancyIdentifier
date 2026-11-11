# PregnancyIdentifier 3.3.2-uiopatched (package version 3.3.2.1)

PregnancyIdentifier 3.3.2 (darwin-eu/PregnancyIdentifier, commit 8e54d94; release `v3.3.2` of this repository)
with three changes, used for the Norwegian linked health registries in the OMOP CDM. Every changed line is marked
"[UiO patch]". Changes by Saeed Hayati and Hedvig M.E. Nordeng.

1. **Gestational age recorded in days.** Version 3.3.2 reads gestational-age values (e.g. "Length of gestation at
   birth", 4260747) from `measurement.value_as_number` as weeks and keeps values of 1–44 only. A CDM that stores
   the gestational age in days (unit 8512) therefore loses every value, and almost all pregnancies are dated by
   the maximum length of their outcome (births 308 days). A value with unit day (8512) is now divided by 7; other
   values are unchanged.
   - `R/initPregnancies.R`: `value_as_number` of the HIP measurement records.
   - `R/ESD.R`, `collectDomainForJoin()` (new argument `unitCol`, used for the measurement table) and the value
     parsing in `getTimingConcepts()`: the numeric value is kept, so that weeks converted from days keep their
     exact days (start = record date − round(weeks × 7)). For whole weeks the result equals version 3.3.2.

2. **Episode windows that end before they start.** `R/ESD.R`, `intervalJoinLocal()`: windows whose start
   (`pmin(merge_pregnancy_start, merge_episode_start)`) lies after `merge_episode_end`, or with a missing date, are
   removed before `data.table::foverlaps()`, which otherwise stops ("All entries in column 'start_date' should be
   <= corresponding entries in column 'merge_episode_end'"). Such a window cannot contain an event, so the result
   is the same as the SQL non-equi join that `intervalJoinLocal()` replaced.

3. **Option, off by default:** `options(PregnancyIdentifier.r3_ga_after_end_days = d)` (`R/ESD.R`,
   `intervalJoinLocal()`). With d > 0, records of gestational age at birth (4260747, 46234792) are matched to an
   episode whose window [start, merge_episode_end + d] contains the record; all other records keep the window of
   version 3.3.2 [start, merge_episode_end]. In a birth-registry-based CDM the gestational age is recorded on the
   delivery date, which can lie after the first outcome record that ends the episode. Default 0 = the window of
   version 3.3.2.
