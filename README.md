# Assignment 4: Visualizing Patient Vitals vs. Physician Assessments in R

This week I worked with a small (made-up) hospital dataset of 10 patients: visit frequency, blood pressure, two doctors' assessments, and a final decision. Goal: clean it, handle a missing value, and use boxplots and histograms to see how blood pressure relates to what the doctors decided.

**Blog post:** https://adrielusf.blogspot.com/2026/09/assignment-4-visualizing-and.html

## Data Preparation

Categorical values were coded numerically (FirstAssess: bad = 1, good = 0; SecondAssess and FinalDecision: low = 0, high = 1).

```r
Frequency     <- c(0.6, 0.3, 0.4, 0.4, 0.2, 0.6, 0.3, 0.4, 0.9, 0.2)
BloodPressure <- c(103, 87, 32, 42, 59, 109, 78, 205, 135, 176)
FirstAssess   <- c(1, 1, 1, 1, 0, 0, 0, 0, NA, 1)
SecondAssess  <- c(0, 0, 1, 1, 0, 0, 1, 1, 1, 1)
FinalDecision <- c(0, 1, 0, 1, 0, 1, 0, 1, 1, 1)

df_hosp <- data.frame(Frequency, BloodPressure, FirstAssess,
                      SecondAssess, FinalDecision)
summary(df_hosp)             # shows 1 NA in FirstAssess
df_hosp <- na.omit(df_hosp)  # 10 rows -> 9 rows
```

## Boxplots

```r
png("plots/bp_by_final_decision.png", width = 800, height = 600)
boxplot(BloodPressure ~ FinalDecision, data = df_hosp,
        names = c("Low", "High"), ylab = "Blood Pressure",
        main = "BP by Final Decision")
dev.off()
```

<img width="800" height="600" alt="bp_by_first_assess" src="https://github.com/user-attachments/assets/75f202cd-2ecb-4d94-9649-1dbeb1f108ab" />

<img width="800" height="600" alt="bp_by_second_assess (1)" src="https://github.com/user-attachments/assets/3757378a-1055-4b85-b9b0-e4f3ad3288e5" />

<img width="800" height="600" alt="bp_by_final_decision (1)" src="https://github.com/user-attachments/assets/42a4849a-6ab1-4e0c-affa-d554fa177278" />


## Histograms

```r
hist(df_hosp$Frequency, breaks = seq(0, 1, by = 0.1),
     xlab = "Visit Frequency", main = "Histogram of Visit Frequency")
hist(df_hosp$BloodPressure, breaks = 8,
     xlab = "Blood Pressure", main = "Histogram of Blood Pressure")
```

<img width="800" height="600" alt="hist_frequency" src="https://github.com/user-attachments/assets/db3fd0b5-b242-4803-954f-8c4c99809313" />

<img width="800" height="600" alt="hist_blood_pressure" src="https://github.com/user-attachments/assets/fe528fb4-1f6d-43a1-a611-e2a05c25f8e5" />


## Discussion

Blood pressure lines up with the final decision far better than with either individual doctor's assessment. The first doctor's "good" and "bad" groups barely differ (medians of 93.5 vs. 87), and the "bad" group spans almost the whole range, from 32 up to 176, so that assessment doesn't seem to be driven by blood pressure at all. The second doctor's "high" group actually has a *lower* median (78) than the "low" group (95), but a much wider spread (32 to 205), while the "low" group is tightly clustered between 59 and 109. The final decision shows the cleanest split: every "low" patient had a blood pressure of 103 or below (median 68.5), while the "high" group had a median of 109 and contains the two highest readings, 176 and 205.

The blood pressure histogram is right-skewed, with most readings between roughly 50 and 110 and a long tail to the right; 205 is flagged as an outlier by R's boxplot rule for the overall distribution. The low end is just as notable: readings of 32 and 42 would be dangerously low in a real patient, which suggests either data-entry problems or that this synthetic data wasn't built with realistic ranges. Visit frequency clusters between 0.2 and 0.4, with only two patients at 0.6. Clinically, the takeaway would be that the final decision seems to weigh blood pressure more heavily than either first-line assessment. But with only 9 usable patients, no units (systolic? diastolic? mmHg?), no statistical testing, and fabricated values, none of this supports a real conclusion. One or two patients can move a median dramatically at this sample size.

Handling the missing value mattered more than I expected. `na.omit()` removes the *entire* row, so patient 9 was dropped from every plot, not just the First Assessment boxplot where the NA actually lives. That patient had a blood pressure of 135, the highest visit frequency (0.9), and "high" ratings on both the second assessment and final decision. Dropping them removed the only 0.9 from the frequency histogram, pulled the second assessment's "high" median down from 106.5 to 78 (which is what flips that group below the "low" group), and pulled the final decision's "high" median from 122 to 109. A better approach would be to drop the NA only for the analysis that needs FirstAssess (e.g. `subset(df_raw, !is.na(FirstAssess))`) and keep all 10 patients for the other plots. Listwise deletion is simple, but on a tiny dataset it throws away real information.
