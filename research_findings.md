# Did Streaming Break How Songs Are Built?

## A Multi-Method Analysis of Structural Changes in Music Composition Across the Streaming Era

---

## 1 — Introduction

### 1.1 Research Context

The music industry underwent a tectonic shift between 2010 and 2018. Physical sales and digital downloads gave way to algorithmic streaming platforms — Spotify, Apple Music, YouTube Music — where discovery is governed by recommendation engines, playlist placement, and attention metrics. This shift fundamentally changed the economic incentives for artists and producers: revenue became a function of per-stream payouts, rewarding replay-ability, playlist compatibility, and hook-driven intros over album-length artistic expression.

Industry analysts and music critics have long argued that these incentives are homogenizing popular music — making it shorter, louder, more formulaic. But is this intuition empirically supported? This study uses a dataset of over 1 million Spotify tracks spanning 2000–2023 to test whether the streaming era measurably changed how songs are composed.

### 1.2 Research Questions

This study is organized as a tree of connected research questions:

**Main Question (Student's t-test):** Did the shift to algorithmic streaming (post-2018) structurally change song composition — specifically duration and tempo — compared to the pre-streaming era (2000–2009)?

**Branch 1 — The Loudness-Valence Trade-off (Pearson's Correlation):** Has the systemic engineering of track loudness degraded the emotional positivity (valence) of music?

**Branch 2 — Tempo Convergence (Variance Comparison):** Has popular music converged toward a narrower tempo range over time, consistent with algorithmic standardization?

### 1.3 Hypotheses

**H1 (Duration):** Tracks released post-2018 have a significantly lower mean `duration_ms` than tracks released between 2000–2009.

**H2 (Tempo):** Tracks released post-2018 have a significantly higher mean `tempo` than tracks from 2000–2009.

**H3 (Loudness-Valence):** There is a significant negative correlation between loudness and valence — as tracks are compressed to be louder, their emotional positivity decreases.

**H4 (Tempo Convergence):** Post-2015 tracks show lower tempo variance than earlier decades, indicating convergence toward a narrower BPM range.

### 1.4 Previous Research

The "Loudness War" has been extensively documented in audio engineering literature — mastering levels have increased steadily since the 1990s as producers compress dynamic range to make tracks sound louder on radio and streaming (Deruty & Tardieu, 2014). Studies using the Million Song Dataset have shown trends toward decreasing timbral diversity and increasing loudness over decades (Serra et al., 2012). Research on Spotify's recommendation algorithm suggests that features like danceability, energy, and track duration influence playlist placement (Anderson et al., 2020). The hypothesis that streaming shortens songs has been supported by observational analyses showing average track length declining from ~4 minutes to ~3:30 since 2017 (Spotify Insights, various reports).

### 1.5 Statistical Techniques Utilized

| Technique | Purpose | Application |
|-----------|---------|-------------|
| **Descriptive Statistics** | Summarize data centrality and dispersion | Mean, median, SD, IQR for all key variables |
| **IQR Outlier Detection** | Identify and treat anomalous values | Cap extreme values in duration, tempo, loudness |
| **Z-score Standardization** | Normalize variables for comparison | Standardize variables used in correlation analysis |
| **Shapiro-Wilk Test** | Check normality assumption | Applied to random subsamples (n=5,000) per group |
| **Kolmogorov-Smirnov Test** | Supplementary normality check | Applied alongside Shapiro-Wilk |
| **Levene's Test** | Check homogeneity of variances | Applied before t-tests and for variance comparison |
| **Welch's t-test** | Compare means of two independent groups | Compare duration and tempo across eras |
| **Cohen's d** | Quantify practical effect size | Supplement p-values with meaningful magnitude |
| **Pearson's r** | Measure linear relationship | Loudness vs. valence correlation |
| **Fisher Z-transformation** | Compare two correlations | Test if correlation changed across eras |
| **Spearman's rho** | Test monotonic trend | Year vs. tempo variance relationship |

---

## 2 — Method

### 2.1 Dataset Description

- **Source:** Spotify audio features dataset (`spotify_data_edited.csv`)
- **Total observations:** 1,048,575 tracks
- **Columns:** 19 variables (3 categorical, 16 numeric)
- **Temporal range:** 2000–2009 and 2012–2023 (natural gap at 2010–2011)
- **Genres:** 82 unique genre labels
- **Missing values:** 0 (no imputation required)
- **Duplicate rows:** 0
- **Duplicate track IDs:** 0

**Key variables used in this study:**

| Variable | Type | Range | Description |
|----------|------|-------|-------------|
| `duration_ms` | Continuous | 2,073–6,000,495 ms | Track length in milliseconds |
| `tempo` | Continuous | 0–249.9 BPM | Estimated tempo |
| `loudness` | Continuous | -56.1–6.2 dB | Overall loudness (LUFS approximation) |
| `valence` | Continuous | 0.0–1.0 | Musical positiveness (0=sad, 1=happy) |
| `popularity` | Discrete | 0–100 | Spotify popularity score |
| `year` | Discrete | 2000–2023 | Release year |

**Era groupings:**

| Group | Years | Tracks |
|-------|-------|--------|
| Pre-streaming | 2000–2009 | 425,924 |
| Transition (excluded from t-test) | 2012–2017 | 309,313 |
| Streaming era | 2018–2023 | 313,338 |

### 2.2 Descriptive Statistics Before Preprocessing

| Variable | Mean | Median | SD | Min | Q1 | Q3 | IQR |
|----------|------|--------|----|-----|----|----|-----|
| popularity | 18.75 | 16.00 | 16.07 | 0 | 5.0 | 29.0 | 24.0 |
| danceability | 0.538 | 0.551 | 0.185 | 0.0 | 0.413 | 0.678 | 0.265 |
| energy | 0.639 | 0.692 | 0.271 | 0.0 | 0.452 | 0.872 | 0.420 |
| loudness | -9.00 | -7.46 | 5.71 | -56.1 | -10.84 | -5.29 | 5.56 |
| valence | 0.454 | 0.436 | 0.268 | 0.0 | 0.225 | 0.672 | 0.447 |
| tempo | 121.31 | 121.66 | 29.79 | 0.0 | 98.64 | 139.89 | 41.25 |
| duration_ms | 247,981 | 224,827 | 144,641 | 2,073 | 180,392 | 285,173 | 104,782 |

### 2.3 Preprocessing Pipeline

**Step 1 — Missing Value Check:**
No missing values were found across any of the 19 columns. No imputation was required.

**Step 2 — Outlier Detection and Treatment (IQR Method):**

Outliers were identified using the IQR fencing rule: Lower Bound = Q1 - 1.5 x IQR, Upper Bound = Q3 + 1.5 x IQR. Rather than removing outliers (which would reduce sample size), we applied **winsorization** — capping values at the fence boundaries.

| Variable | Lower Bound | Upper Bound | Outliers Detected | % of Data |
|----------|-------------|-------------|-------------------|-----------|
| duration_ms | 23,219 ms | 442,345 ms | 54,834 | 5.23% |
| tempo | 36.76 BPM | 201.77 BPM | 4,951 | 0.47% |
| loudness | -19.18 dB | 3.05 dB | 68,835 | 6.56% |
| valence | -0.45 | 1.34 | 0 | 0.00% |

Notably, `loudness` had the highest outlier rate (6.56%), consistent with extreme dynamic compression in certain genres (e.g., black-metal, death-metal, ambient).

**Step 3 — Standardization:**
Z-score standardization was applied to `loudness`, `valence`, `tempo`, and `duration_ms` for use in correlation analysis. After standardization, each variable had mean = 0 and SD = 1.

### 2.4 Descriptive Statistics After Preprocessing

| Variable | Mean | Median | SD | Min | Q1 | Q3 | IQR |
|----------|------|--------|----|-----|----|----|-----|
| loudness | -8.62 | -7.46 | 4.55 | -19.18 | -10.84 | -5.29 | 5.56 |
| tempo | 121.33 | 121.66 | 29.61 | 36.76 | 98.64 | 139.89 | 41.25 |
| duration_ms | 238,864 | 224,827 | 89,518 | 23,219 | 180,392 | 285,173 | 104,782 |
| valence | 0.454 | 0.436 | 0.268 | 0.0 | 0.225 | 0.672 | 0.447 |

Key changes after preprocessing: `duration_ms` mean dropped from 247,981 to 238,864 (extreme long tracks were capped). Standard deviation dropped from 144,641 to 89,518, indicating reduced spread from outlier treatment. `loudness` SD dropped from 5.71 to 4.55 as extreme quiet tracks were capped.

---

## 3 — Results

### 3.1 Main Analysis: The Streaming Attention Span (Welch's t-test)

#### 3.1.1 Assumption Checks

**Normality (Shapiro-Wilk on n=5,000 random subsamples):**

| Variable | Group | W | p-value |
|----------|-------|---|---------|
| duration_ms | Pre-streaming | 0.9652 | 4.31e-33 |
| duration_ms | Streaming era | 0.9469 | 3.66e-39 |
| tempo | Pre-streaming | 0.9852 | 1.76e-22 |
| tempo | Streaming era | 0.9900 | 2.24e-18 |

All Shapiro-Wilk tests rejected normality (p < 0.001). This is expected and typical for large datasets — with n > 400,000 per group, even trivial deviations from perfect normality will be detected. By the **Central Limit Theorem**, the sampling distribution of the mean is approximately normal for these sample sizes, making the t-test robust despite non-normal raw data.

**Supplementary KS tests** confirmed non-normality (all p < 0.001).

**Homoscedasticity (Levene's Test):**

| Variable | F | p-value | Interpretation |
|----------|---|---------|----------------|
| duration_ms | 8,271.33 | < 2.2e-308 | Variances significantly differ |
| tempo | 4.09 | 4.30e-02 | Variances significantly differ |

Both variables showed unequal variances, so **Welch's t-test** (which does not assume equal variances) was used throughout.

#### 3.1.2 T-test Results

**Test 1: Song Duration**

| Metric | Pre-Streaming (2000–2009) | Streaming Era (2018–2023) |
|--------|---------------------------|---------------------------|
| n | 425,924 | 313,338 |
| Mean | 249,304 ms (4:09) | 221,489 ms (3:41) |
| Median | 235,427 ms (3:55) | 208,221 ms (3:28) |
| SD | 94,358 ms | 81,964 ms |

| Statistic | Value |
|-----------|-------|
| Welch's t | 135.17 |
| p-value (one-tailed) | < 2.2e-308 |
| Cohen's d | 0.315 (small effect) |
| Mean difference | -27,815 ms (-11.16%) |

**Result: H1 SUPPORTED.** Songs in the streaming era are significantly shorter — by an average of 28 seconds (11.16% reduction). The effect size is small but practically meaningful: the equivalent of losing nearly half a verse from the average track.

**Test 2: Tempo**

| Metric | Pre-Streaming (2000–2009) | Streaming Era (2018–2023) |
|--------|---------------------------|---------------------------|
| n | 425,924 | 313,338 |
| Mean | 120.70 BPM | 121.71 BPM |
| Median | 120.07 BPM | 122.06 BPM |
| SD | 29.59 BPM | 29.78 BPM |

| Statistic | Value |
|-----------|-------|
| Welch's t | -14.45 |
| p-value (one-tailed) | 1.28e-47 |
| Cohen's d | 0.034 (negligible effect) |
| Mean difference | +1.01 BPM (+0.84%) |

**Result: H2 STATISTICALLY SUPPORTED but PRACTICALLY NEGLIGIBLE.** While the difference is statistically significant (p < 0.001), the actual difference is only 1 BPM — imperceptible to any listener. Cohen's d = 0.034 confirms a trivially small effect. This is a textbook example of how massive sample sizes can make meaningless differences "significant." We conclude there is **no practically meaningful tempo increase** in the streaming era.

### 3.2 Branch 1: The Loudness-Valence Trade-off (Pearson's Correlation)

#### 3.2.1 Overall Correlation

| Metric | Value |
|--------|-------|
| Pearson's r | **+0.277** |
| p-value | < 2.2e-308 |
| r-squared | 0.077 |
| Interpretation | Weak **positive** correlation |

**Result: H3 REJECTED.** The hypothesis predicted a *negative* correlation — that louder tracks would be less emotionally positive. The data shows the opposite: louder tracks tend to be *more* positive in valence. This makes musical sense — energetic, upbeat songs (high valence) tend to have louder mixes, while quiet, melancholic tracks (low valence) are often softer.

#### 3.2.2 Per-Era Correlation

| Era | Pearson's r | r-squared | n |
|-----|-------------|-----------|---|
| Pre-streaming (2000–2009) | +0.285 | 0.081 | 425,924 |
| Streaming era (2018–2023) | +0.299 | 0.089 | 313,338 |

**Fisher Z-test (comparing correlations across eras):**

| Metric | Value |
|--------|-------|
| Z-statistic | -6.25 |
| p-value | 4.16e-10 |

The correlation has **significantly strengthened** in the streaming era (from r=0.285 to r=0.299), though the absolute difference is small. This suggests that in the streaming era, the coupling between loudness and positivity has slightly tightened — loud-positive and quiet-sad tracks have become even more formulaic.

#### 3.2.3 Yearly Trend

The Pearson r between loudness and valence by year shows fluctuation between r = 0.19 and r = 0.36:

| Year | r | Year | r | Year | r |
|------|---|------|---|------|---|
| 2000 | 0.316 | 2008 | 0.266 | 2017 | 0.286 |
| 2001 | 0.356 | 2009 | 0.194 | 2018 | 0.276 |
| 2002 | 0.299 | 2012 | 0.280 | 2019 | 0.288 |
| 2003 | 0.293 | 2013 | 0.263 | 2020 | 0.292 |
| 2004 | 0.307 | 2014 | 0.263 | 2021 | 0.305 |
| 2005 | 0.266 | 2015 | 0.275 | 2022 | 0.321 |
| 2006 | 0.267 | 2016 | 0.252 | 2023 | 0.319 |
| 2007 | 0.303 | | | | |

The correlation dipped to its lowest point in 2009 (r=0.194) and has been climbing steadily since ~2016, reaching recent highs of r=0.321 in 2022 — the strongest coupling in over a decade.

### 3.3 Branch 2: Tempo Convergence (Variance Comparison)

#### 3.3.1 Tempo Statistics by Era

| Era | Mean (BPM) | Median (BPM) | Variance | SD | IQR |
|-----|------------|--------------|----------|----|-----|
| 2000–2005 | 120.03 | 119.63 | 880.23 | 29.67 | 41.08 |
| 2006–2009 | 121.72 | 122.60 | 866.79 | 29.44 | 40.59 |
| 2012–2015 | 121.90 | 122.03 | 863.46 | 29.38 | 39.99 |
| 2016–2019 | 121.61 | 121.99 | 878.09 | 29.63 | 40.10 |
| 2020–2023 | 121.78 | 122.60 | 889.11 | 29.82 | 40.66 |

#### 3.3.2 Levene's Test (Overall)

| Metric | Value |
|--------|-------|
| Levene's F | 53.33 |
| p-value | 5.15e-45 |

The omnibus Levene's test is **significant** — tempo variances are not equal across all five eras.

#### 3.3.3 Pairwise Levene's Tests

| Comparison | F | p-value | Significant? |
|------------|---|---------|--------------|
| 2000–2005 vs 2006–2009 | 82.39 | 1.12e-19 | Yes |
| 2000–2005 vs 2012–2015 | 189.16 | 4.94e-43 | Yes |
| 2000–2005 vs 2016–2019 | 67.12 | 2.56e-16 | Yes |
| 2000–2005 vs 2020–2023 | 17.91 | 2.32e-05 | Yes |
| 2006–2009 vs 2012–2015 | 13.15 | 2.87e-04 | Yes |
| 2006–2009 vs 2016–2019 | 1.58 | 0.208 | **No** |
| 2012–2015 vs 2016–2019 | 26.61 | 2.50e-07 | Yes |
| 2012–2015 vs 2020–2023 | 76.41 | 2.31e-18 | Yes |
| 2016–2019 vs 2020–2023 | 13.11 | 2.94e-04 | Yes |

#### 3.3.4 Trend Analysis

| Metric | Value |
|--------|-------|
| Spearman's rho (year vs. tempo variance) | +0.203 |
| p-value | 0.366 |

**Result: H4 PARTIALLY REJECTED.** While Levene's test confirms that variances differ significantly across eras, the pattern is **not a monotonic decrease** as hypothesized. Tempo variance actually follows a U-shape: it decreased from 2000–2005 (880) to its minimum at 2012–2015 (863), then **increased again** to 2020–2023 (889). The Spearman trend test is non-significant (p = 0.366), meaning there is no consistent directional convergence over time.

The most interesting finding is that the lowest-variance era was 2012–2015 — the period when streaming was *emerging* but not yet dominant. The most recent era (2020–2023) actually shows the **highest** tempo variance, contradicting the convergence hypothesis entirely.

---

## 4 — Discussion

### 4.1 Summary of Findings

| Hypothesis | Supported? | Key Finding |
|------------|------------|-------------|
| H1: Songs are shorter post-2018 | **Yes** | -28 seconds (-11.16%), d=0.315 |
| H2: Songs are faster post-2018 | Statistically yes, practically **no** | +1 BPM (+0.84%), d=0.034 |
| H3: Louder tracks are less happy | **No** (reversed) | r=+0.277, louder = happier |
| H4: Tempo is converging | **No** | Variance is U-shaped, not declining |

The data paints a nuanced picture. **Streaming has measurably shortened songs** — the single strongest and most practically meaningful finding, consistent with industry observations. However, the other hypothesized effects of algorithmic homogenization largely fail to materialize:

- Tempo has not meaningfully increased, nor has it converged.
- The loudness-valence relationship runs opposite to our prediction, and while it has slightly strengthened in recent years, this suggests genre consolidation rather than emotional degradation.

### 4.2 Interpreting the Duration Decline

The 28-second reduction in average song length aligns with economic incentives in the streaming model. Spotify pays per stream, not per minute — a 3-minute song generates the same revenue as a 5-minute song. Artists and labels therefore have rational incentive to shorten tracks, enabling more songs per album and more potential stream-triggers. The decline in the median (from 3:55 to 3:28) is even more striking than the mean, suggesting the shift is not driven by outliers but reflects a genuine population-level restructuring.

### 4.3 The Loudness-Valence Surprise

Our hypothesis assumed that loudness compression destroys emotional nuance, reducing valence. The positive correlation instead suggests that loudness and valence share a common cause — **energy**. High-energy genres (pop, dance, EDM) tend to be both loud and positive, while low-energy genres (ambient, classical, sad) are quieter and more melancholic. The correlation is not causal — it reflects genre structure. The Fisher Z-test's finding that this coupling is tightening in the streaming era may indicate increasing genre polarization: streaming playlists that cluster "upbeat bangers" separately from "chill sad vibes."

### 4.4 Tempo Variance: No Convergence

The U-shaped variance pattern suggests that the streaming era has, if anything, **diversified** tempo. This may reflect the long-tail effect of streaming platforms: while mainstream hits may cluster around a narrow tempo range, streaming democratizes access to niche genres (extreme metal, ambient, experimental), which widen the tempo distribution. The lowest variance era (2012–2015) coincides with peak "EDM bubble" years, when a specific tempo range (~128 BPM) dominated popular culture.

### 4.5 Reliability of Measures

- **Internal consistency:** Spotify's audio features are algorithmically computed and consistent within the platform, but represent a single measurement system with no competing measures to assess reliability.
- **Test-retest:** The audio features for any given track do not change over time (they are computed once at upload), so test-retest reliability is inherently perfect within Spotify's system.
- **Cross-validation:** Not applicable to this descriptive/correlational design, but would be relevant if building predictive models.

### 4.6 Validity

#### Internal Validity
- **Strengths:** The dataset is massive (>1M tracks), reducing sampling error to near-zero. Welch's t-test accounts for unequal variances. Effect sizes supplement p-values.
- **Threats:** The study is observational and cannot establish causation. The gap at 2010–2011 may introduce cohort effects. Year-of-release is confounded with other cultural shifts (social media, pandemic, genre trends). We cannot isolate streaming as the sole cause of any observed differences.

#### External Validity
- **Strengths:** The dataset spans 82 genres and 22 years, representing a broad cross-section of music.
- **Limitations:** The data comes exclusively from Spotify, which may over-represent Western popular music and under-represent local, regional, or underground scenes. Popularity scores are Spotify-specific and do not reflect total cultural impact (e.g., TikTok virality, radio play, concert attendance).

#### Construct Validity
- **`valence`** is Spotify's algorithmic estimation of musical positiveness. It is not a direct measure of listener emotional experience — a song classified as "happy" by the algorithm may evoke sadness in context (e.g., nostalgic associations). The construct validity of valence as a measure of emotional content is debatable.
- **`loudness`** is an objective acoustic measurement (approximately LUFS), making it the most construct-valid variable in this study.
- **`tempo`** is algorithmically estimated and generally reliable, though it can be confused by half-time/double-time patterns.

### 4.7 Limitations

1. **Correlation, not causation:** We cannot prove streaming *caused* shorter songs. Other factors (TikTok, shorter attention spans, smartphone usage patterns) may be responsible.
2. **Survivorship bias:** The dataset only contains tracks available on Spotify. Delisted or region-locked tracks are absent.
3. **Genre imbalance:** Some genres have more tracks than others, which may skew aggregate statistics.
4. **2010–2011 gap:** The missing years create a discontinuity in longitudinal analysis.
5. **Popularity score dynamics:** Spotify's popularity metric is time-decayed and recency-biased, meaning older tracks appear less popular even if they were once hits.
6. **Ecological fallacy:** Aggregate trends across all genres may not hold within specific genres.

### 4.8 Future Directions

1. **Within-genre analysis:** Repeat the t-test and correlation analyses within specific genres (e.g., pop, hip-hop, rock) to test whether the duration decline is universal or genre-specific.
2. **Causal inference:** Use interrupted time-series analysis or regression discontinuity around key streaming milestones (e.g., Spotify's launch in each market) to better isolate causal effects.
3. **Intro length analysis:** Test whether the first 30 seconds of songs have become more hook-dense (higher energy, faster onset) as a response to skip-based algorithms.
4. **Cross-platform comparison:** Compare Spotify features with data from other platforms (Apple Music, YouTube) to assess external validity.
5. **Listener-level analysis:** Pair audio features with actual listener behavior data (skip rates, completion rates) to test whether shorter songs actually perform better algorithmically.

---

## Visual References

All figures are saved in the `figures/` directory:

| Figure | Description |
|--------|-------------|
| `phase1_initial_distributions.png` | Histograms of key variables before preprocessing |
| `phase2_before_after_boxplots.png` | Box plots showing effect of outlier capping |
| `phase3_era_comparison.png` | Box plots and density histograms comparing eras for duration and tempo |
| `phase3_yearly_trends.png` | Year-by-year mean and median trends for duration and tempo |
| `phase4_loudness_valence_scatter.png` | Scatter plots with regression lines (overall, pre, post) |
| `phase4_yearly_correlation_trend.png` | Pearson r between loudness and valence over time |
| `phase5_tempo_convergence.png` | Tempo box plots by era and variance trend line |
| `phase5_variance_bars.png` | Bar chart of tempo variance per era |

---

## 5 — Conclusion

So, did streaming actually break how songs are built? The short answer is: **partially — but not in the ways we expected.**

The clearest and strongest finding is about **song length**. Tracks in the streaming era (2018–2023) are, on average, 28 seconds shorter than tracks from the pre-streaming era (2000–2009) — a drop of about 11%. That's not a subtle statistical artifact; it's nearly half a verse gone. The median fell from 3 minutes 55 seconds down to 3 minutes 28 seconds. This lines up with the economic reality of streaming: artists get paid per play, not per minute, so shorter songs mean more potential streams in the same listening window. The data backs this up with high confidence (p < 0.001) and a meaningful, if modest, effect size (Cohen's d = 0.315).

When it comes to **tempo**, the story is different. Yes, post-2018 songs are technically "faster" — but only by 1 BPM. That's a difference no human ear could detect, and the effect size is essentially zero (d = 0.034). This is a useful reminder that with a million data points, even completely trivial differences become statistically significant. In practical terms, streaming has not made music faster.

The **loudness-valence** hypothesis gave us a genuine surprise. We predicted that louder tracks would be less emotionally positive — that compressing dynamic range would flatten emotional content. The data showed the exact opposite: louder tracks tend to be *happier* (r = +0.28). This makes intuitive sense once you think about it — upbeat dance tracks and pop hits are both loud and cheerful, while quiet acoustic ballads and ambient pieces tend to be more melancholic. Loudness and valence aren't in opposition; they're both reflections of a song's energy. Interestingly, this positive coupling has gotten slightly stronger in recent years, which may suggest that streaming playlists are increasingly sorting music into neat emotional boxes — "upbeat bangers" in one playlist, "sad chill vibes" in another.

Finally, we tested whether **tempo is converging** — whether streaming algorithms are pushing all music toward the same BPM. The answer is no. Tempo variance actually follows a U-shape: it dipped to its lowest point around 2012–2015 (the peak of the EDM era, when 128 BPM dominated charts), and has since bounced back. The most recent years (2020–2023) show the *highest* tempo variance in the entire dataset. If anything, the streaming long tail — where niche genres from death metal to ambient can find their audience — may be keeping tempo diversity alive.

**In plain terms:** streaming has made songs shorter, but it hasn't made them faster, sadder, or more uniform in tempo. The music industry adapted to streaming economics in the most direct way possible — by trimming track length — while the deeper musical DNA of songs (their emotional tone, their rhythmic variety) has remained remarkably resilient.
