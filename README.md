# Spotify Popularity Regression

What predicts a track's popularity on Spotify? This project analyzes 586,601 tracks (1921 to 2020) using exploratory analysis, OLS regression with release year fixed effects, and hypothesis tests.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/lakshyasajal18/spotify_popularity_regression/blob/main/Spotify_Audio_Features_Analysis.ipynb)

## Headline finding

Energy looks positively related to popularity until release year is controlled for. Once it is, the sign reverses. Newer tracks are both more popular and more energetic, so the raw relationship mostly reflects time, not the audio feature. Once year is controlled, every audio feature's association with popularity is small (about 1.3 points per standard deviation or less on a 0 to 100 scale), while explicit content is far larger (about 8 points).

All results are associations in observational data, not causal effects.

## Data

- Spotify Dataset 1921 to 2020 on Kaggle (`yamaerenay/spotify-dataset-19212020-600k-tracks`, `tracks.csv`): 586,672 tracks, of which 586,601 are used (71 tracks with no name are dropped).
- `release_date` mixes `YYYY-MM-DD`, `YYYY-MM` and `YYYY` formats. The default pandas parser silently turns the non matching rows into missing values (about 24% of the data), so dates are parsed with `format='mixed'`.
- Popularity is a 0 to 100 score based heavily on recent plays, so newer tracks are favored by construction.

## Method

1. **Exploratory analysis:** correlation matrix, audio attribute trends by year, and popularity versus energy, liveness and acousticness.
2. **Regression (OLS, heteroskedasticity robust HC1 standard errors):** popularity on five z-scored audio features (energy, danceability, acousticness, tempo, liveness).
   - Model 1: audio features only.
   - Model 2: adds explicit content and a fixed effect for every release year.
   - Multicollinearity checked with VIF, residuals checked visually.
3. **Group comparisons (tracks released 2010 or later):** Welch's t-test, 95% confidence intervals, Cohen's d, and a Mann-Whitney robustness check, with a Bonferroni adjusted threshold for two tests.

## Results

Coefficients are popularity points per 1 standard deviation increase in the feature.

| Feature | Model 1 (audio only) | Model 2 (+ explicit, year FE) | Model 2 95% CI |
|---|---|---|---|
| energy | +1.39 | -0.33 | -0.39 to -0.28 |
| danceability | +1.69 | +0.65 | +0.61 to +0.69 |
| acousticness | -5.42 | -1.24 | -1.30 to -1.18 |
| tempo | -0.01 (p = 0.60) | -0.10 | -0.14 to -0.07 |
| liveness | -0.92 | -0.58 | -0.62 to -0.54 |
| explicit (0 or 1) | n/a | +8.25 | +8.02 to +8.48 |

- R-squared rises from 0.152 (Model 1) to 0.384 (Model 2). Roughly 60% of the variation in popularity is unexplained.
- All VIFs are 2.2 or lower, so overlap between features is not distorting the estimates.

Group comparisons, tracks released 2010 or later:

| Comparison (A minus B) | Difference | 95% CI | Cohen's d |
|---|---|---|---|
| Explicit vs not explicit | +11.01 points | +10.71 to +11.31 | +0.54 (medium) |
| High vs low energy (median split) | -4.16 points | -4.39 to -3.94 | -0.20 (small) |

These are not randomized experiments, and they do not hold other features or exact year fixed, so they differ from the regression estimates. The regression numbers are the better estimates of each variable's own association.

## Limitations

- Observational data: no causal claims. Restricting group comparisons to 2010 and later reduces but does not remove confounding by release year.
- Popularity is bounded between 0 and 100 and skewed, so OLS is an approximation. Robust standard errors and a rank based test limit reliance on its assumptions.
- Audio features are Spotify's own model outputs, not direct measurements.
- Artist, genre, marketing, playlist placement and lyrics are not in the data. The explicit content estimate may partly reflect genre.
- Years before about 1940 and the highest popularity scores contain few tracks, so patterns there are unreliable.

## Run it

Open the notebook in Colab with the badge above and run all cells. It downloads the data with `kagglehub`.

Requirements: `pandas`, `numpy`, `matplotlib`, `seaborn`, `statsmodels`, `scipy`, `kagglehub`.

## Files

- `Spotify_Audio_Features_Analysis.ipynb`: full analysis with outputs.
