# Survey data analysis in R

A small, reproducible learning example from [Ghostwriter Akademie](https://ghostwriter-akademie.ch/), Winterthur, Switzerland. All observations below are synthetic. No client or participant data is included.

## What this guide covers

- Keep raw data separate from cleaned data.
- Check identifiers, missing values and questionnaire ranges.
- Describe an ordinal response with counts and percentages.
- Summarise a numerical variable without silently dropping missing values.
- Export a reproducible table and chart using base R.

## Run the example

Copy the following code into a new R script and run it in an empty working folder. It uses base R and creates a local `output` folder. It does not upload data or install packages.

```r
# Synthetic demonstration data, not research findings.
survey <- data.frame(
  id = 1:12,
  study_hours = c(2, 4, 3, NA, 6, 1, 5, 4, 2, 7, 3, 5),
  satisfaction = c(2, 3, 4, 3, 5, 2, NA, 4, 3, 5, 4, 1)
)

# Stop if the data violate the documented schema.
stopifnot(!anyNA(survey$id), !anyDuplicated(survey$id))
stopifnot(is.numeric(survey$study_hours))
stopifnot(all(is.na(survey$study_hours) |
              (is.finite(survey$study_hours) & survey$study_hours >= 0)))
stopifnot(all(is.na(survey$satisfaction) |
              survey$satisfaction %in% 1:5))

missingness <- data.frame(
  variable = names(survey),
  missing_n = colSums(is.na(survey))
)

# Ordinal response: retain every possible category, even with zero counts.
valid <- sum(!is.na(survey$satisfaction))
counts <- table(factor(survey$satisfaction, levels = 1:5))
satisfaction_table <- data.frame(
  response = names(counts),
  count = as.integer(counts),
  percent_of_valid = if (valid > 0)
    round(100 * as.integer(counts) / valid, 1) else rep(NA_real_, 5)
)

# Explicitly report the sample size used for the numeric summary.
hours <- survey$study_hours[!is.na(survey$study_hours)]
hours_summary <- data.frame(
  n_total = nrow(survey),
  n_valid = length(hours),
  n_missing = sum(is.na(survey$study_hours)),
  mean = if (length(hours)) mean(hours) else NA_real_,
  median = if (length(hours)) median(hours) else NA_real_,
  sd = if (length(hours) > 1) sd(hours) else NA_real_
)

dir.create("output", showWarnings = FALSE)
write.csv(missingness, "output/missingness.csv", row.names = FALSE)
write.csv(satisfaction_table, "output/satisfaction.csv", row.names = FALSE)
write.csv(hours_summary, "output/study-hours.csv", row.names = FALSE)
png("output/satisfaction.png", width = 1000, height = 600)
barplot(counts, xlab = "Satisfaction response (1 to 5)",
        ylab = "Number of valid responses",
        main = "Synthetic survey: satisfaction", col = "#15304F")
dev.off()
writeLines(capture.output(sessionInfo()), "output/session-info.txt")
print(missingness)
print(satisfaction_table)
print(hours_summary)
```

## Check your output

The example contains 12 rows. Each of the two measured variables has one missing response. There are 11 valid satisfaction responses, with counts 1, 2, 3, 3 and 2 for categories 1 through 5. Percentages use those 11 valid responses as the denominator; rounding can affect their displayed sum. Mean study time is approximately 3.82 hours and the median is 4 hours.

## Adapt it responsibly

Write a codebook before analysing real data: variable name, question wording, response labels, units and missing-value codes. A blank answer is not automatically zero. Reverse-coded questions must be recoded according to the questionnaire documentation before constructing a scale.

Describe single ordinal items primarily with their distribution. Do not treat the numerical distance between response labels as an established measurement property. Choose any scale summary or inferential model based on your design and assumptions.

This tiny convenience example cannot establish a causal effect or support population generalisations. For actual research, document sampling, exclusions, missing-data decisions and limitations. Do not choose analyses only because they produce a desired p-value.

Keep identifying information outside public repositories. Use synthetic examples when asking for help. Preserve an unchanged raw-data copy locally and document every cleaning step in code.

## Reproducibility checklist

- [ ] Research question and variables documented
- [ ] Consent and data access requirements checked
- [ ] Raw data kept separate and unchanged
- [ ] Missing values and invalid entries counted
- [ ] Denominators stated in tables
- [ ] Interpretation matches the study design
- [ ] R session information saved
- [ ] No personal or confidential data published

## About the publisher

[Ghostwriter Akademie](https://ghostwriter-akademie.ch/) offers writing consultation, proofreading, editing and quantitative and qualitative data-analysis support in Switzerland. Academic examples are learning and reference materials and must not be submitted as someone else's assessed work.

[LinkedIn](https://www.linkedin.com/company/146767232) · [X](https://x.com/ghostwriterakad)
