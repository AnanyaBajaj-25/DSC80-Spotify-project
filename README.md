# Are Explicit Tracks More Popular?
### A Spotify Audio Features Analysis
**By Ananya Bajaj**
**Project for DSC 80 at UCSD**

## Introduction

In this project, I examined a dataset collected via the Spotify Web API, adapted from the 
[Spotify Dataset 1921-2020](https://www.kaggle.com/datasets/yamaerenay/spotify-dataset-19212020-600k-tracks) 
by Yamac Eren Ay on Kaggle. The original dataset spans 114 genres, but I narrowed my 
analysis to 5 musically distinct genres: classical, metal, hip-hop, country, and EDM — 
chosen to represent a wide range of explicit content rates, from classical where explicit 
tracks are nearly nonexistent, to hip-hop and metal where explicit content is common.

My central research question is: **Do explicit tracks tend to be more popular than 
non-explicit tracks?**

This question addresses an important cultural debate, from parental advisory labels 
to platform content filters today. Explicit tracks are intentionally excluded from 
certain curated playlists to limit exposure to profanity, particularly for younger 
listeners. Despite this, genres like hip-hop and metal treat explicit language as 
core to their artistic identity, making these restrictions less effective in practice. 
If explicit tracks consistently end up being more popular despite these limitations, 
it raises an important question about how streaming algorithms shape what gets heard 
and who benefits.

The dataset was constructed by merging two source files: `music_tracks.csv` 
(track-level audio features and metadata) and `artists.csv` (artist-level 
follower counts), joined on primary artist name.

The cleaned dataset contains **5,505 rows**, where each row represents one track. 
The columns most relevant to my question are:

| Column | Description |
|---|---|
| `track_name` | Name of the track |
| `track_genre` | Genre of the track, as labeled by Spotify |
| `popularity` | Spotify popularity score from 0–100, based on total plays and recency |
| `popular` | Binary indicator of whether popularity ≥ 70 |
| `danceability` | How suitable the track is for dancing, based on tempo, rhythm, and beat strength (0–1) |
| `energy` | Perceptual intensity of the track — loud, fast tracks score high (0–1) |
| `loudness` | Overall loudness in decibels (typically −60 to 0 dB) |
| `speechiness` | Presence of spoken words in the track (0–1) |
| `acousticness` | Confidence that the track is acoustic rather than electronic (0–1) |
| `instrumentalness` | Probability the track contains no vocals (0–1) |
| `valence` | Musical positiveness — high values sound happy, low values sound sad (0–1) |
| `tempo` | Estimated beats per minute (BPM) |
| `explicit` | Whether the track contains explicit content (True/False) |
| `release_year` | Year the track was released, extracted from the release date |
| `followers` | Total Spotify followers for the track's primary artist |
| `followers_missing` | Boolean flag indicating whether followers data is missing |

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

To prepare the dataset for analysis, I performed the following cleaning steps:

1. **Merged two datasets**: I merged `music_tracks.csv` with `artists.csv` on the 
primary artist name. Since some tracks list multiple artists separated by semicolons, 
only the first listed artist was used for the merge. A left join was used to preserve 
all tracks even if the artist was not found in `artists.csv`, resulting in some missing 
values in the `followers` column.

2. **Filtered to 5 genres**: I selected classical, metal, hip-hop, country, and EDM 
to represent a wide range of explicit content rates from genres with almost no 
explicit tracks (classical) to genres where explicit content is common (hip-hop, metal).

3. **Extracted release year**: The `release_date` column had inconsistent formats 
(e.g. `2018-08-10`, `1995-04`, `1974`). I extracted the first 4 characters to get 
a consistent integer `release_year` column.

4. **Created `popular` column**: Tracks with a popularity score ≥ 70 were labeled 
as popular (True), all others as non-popular (False). This threshold represents 
approximately the top 13% of tracks in the dataset.

5. **Imputed missing tempo**: One track had a missing `tempo` value, which was filled 
with the median tempo across all tracks.

6. **Created `followers_missing` flag**: A boolean column was added to flag tracks 
where artist follower data was missing, for use in the missingness analysis.

The first few rows of this cleaned DataFrame are shown below, with a portion of 
columns selected.

|    | track_name             | track_genre   |   popularity | explicit   |   followers |   release_year |
|---:|:-----------------------|:--------------|-------------:|:-----------|------------:|---------------:|
|  0 | Zara Zara              | classical     |           58 | False      |      124188 |           2001 |
|  1 | Kajra Re               | classical     |           59 | False      |       11604 |           2005 |
|  2 | Zara Zara - Lofi       | classical     |           54 | False      |      124188 |           1984 |
|  3 | Vaseegara              | classical     |           68 | False      |      124188 |           1972 |
|  4 | Zara Zara - LoFi Chill | classical     |           59 | False      |      124188 |           1987 |

### Univariate Analysis

The plot below shows the distribution of popularity scores across all tracks in the 
dataset. The distribution is heavily skewed toward zero, with a large number of tracks 
having very low popularity scores. This suggests that most tracks in the dataset are 
relatively obscure, with only a small number of mainstream hits reaching high popularity.

<iframe
  src="assets/popularity_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The plot below shows the proportion of explicit vs non-explicit tracks in the dataset. 
Non-explicit tracks make up the large majority, which means the dataset is imbalanced 
with respect to explicit content, an important consideration when interpreting 
comparisons between the two groups.

<iframe
  src="assets/explicit_proportion.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

The box plot below shows the distribution of popularity scores for explicit vs 
non-explicit tracks. Explicit tracks show a slightly higher median popularity, 
suggesting a potential relationship between explicit content and popularity.

<iframe
  src="assets/popularity_by_explicit.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The grouped box plot below breaks down popularity by genre and explicit content. 
The relationship between explicit content and popularity varies considerably across 
genres. Metal shows the clearest advantage for explicit tracks, while hip-hop's 
results are affected by a heavily skewed distribution of explicit track popularity scores.

<iframe
  src="assets/popularity_by_genre_explicit.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

The pivot table below shows the median popularity of explicit vs non-explicit tracks 
for each genre. EDM shows almost identical medians for both groups, while metal shows 
the largest difference, explicit metal tracks have a notably higher median popularity 
than non-explicit ones. Classical has no explicit tracks at all, confirming the 
genre-level imbalance noted earlier.

| track_genre   |   False |   True |
|:--------------|--------:|-------:|
| classical     |       3 |    nan |
| country       |       0 |     43 |
| edm           |      46 |     47 |
| hip-hop       |      59 |      1 |
| metal         |      52 |     65 |


## Assessment of Missingness

### NMAR Analysis

The `followers` column contains missing values that I believe are **NMAR** (Not Missing 
At Random). The missingness is likely related to the value itself. Artists with very 
few or zero followers may not have been fully indexed by Spotify's API at the time the 
data was collected, meaning the most obscure artists are the ones most likely to have 
missing follower data. To make this MAR, I would need additional data such as the 
date each artist's profile was created or whether the artist is still active on Spotify.

### Missingness Dependency

I performed two permutation tests to assess whether the missingness of `followers` 
depends on other columns.

**Test 1: Followers Missingness vs. Popularity**

- **Null Hypothesis:** The missingness of `followers` does not depend on `popularity`.
- **Alternative Hypothesis:** The missingness of `followers` depends on `popularity`.
- **Test Statistic:** Absolute difference in mean popularity between missing and non-missing groups.
- **P-value:** Around 0.837

With a p-value of around 0.837, we fail to reject the null hypothesis. The missingness of 
`followers` does not appear to depend on popularity.

<iframe
  src="assets/missingness_permutation.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Test 2: Followers Missingness vs. Genre**

- **Null Hypothesis:** The missingness of `followers` does not depend on `track_genre`.
- **Alternative Hypothesis:** The missingness of `followers` depends on `track_genre`.
- **Test Statistic:** Total Variation Distance (TVD) between genre distributions.
- **P-value:** Around 0.001

With a p-value of 0.001, we reject the null hypothesis. The missingness of `followers` 
is significantly dependent on genre, suggesting that certain genres are more likely 
to have missing follower data than others.

<iframe
  src="assets/genre_missingness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Since `followers` is used as a feature in our predictive model, understanding its 
missingness is important. It shows that obscure artists with missing follower data 
may be systematically harder to classify as popular or not popular.


## Hypothesis Testing

**Research Question:** Do explicit tracks tend to be more popular than non-explicit tracks?

- **Null Hypothesis:** Explicit and non-explicit tracks have the same mean popularity — 
any observed difference is due to random chance.
- **Alternative Hypothesis:** Explicit tracks have higher mean popularity than 
non-explicit tracks.
- **Test Statistic:** Difference in mean popularity (explicit minus non-explicit). 
Mean difference was chosen because popularity is a numerical variable and we want to 
detect a shift in the center of the distribution between the two groups.
- **Significance Level:** 0.05 — a standard threshold that balances the risk of 
false positives and false negatives.
- **Observed Difference:** 5.063
- **P-value:** < 0.001

A one-tailed permutation test was used because our alternative hypothesis predicts 
a specific direction stating explicit tracks are more popular, not just different. 
A permutation test was chosen over a t-test because the popularity distribution is 
heavily skewed, violating the normality assumption required for a t-test.

With a p-value of < 0.001, we reject the null hypothesis. In 1000 permutations, 
the observed difference of 5.063 was never exceeded by chance, suggesting the 
difference is statistically significant. However, since this is an observational 
dataset and not a randomized controlled trial, we cannot conclude that explicit 
content directly causes higher popularity. Other factors such as genre or artist 
following size may contribute to this pattern.

## Framing a Prediction Problem

**Prediction Problem:** Predict whether a track will be popular (popularity ≥ 70) 
from its audio and metadata features.

**Type:** Binary classification — the response variable `popular` takes two values: 
True (popular) and False (not popular).

**Response Variable:** `popular` — a binary column derived from Spotify's popularity 
score, where tracks with a score ≥ 70 are considered popular. This threshold was 
chosen because it represents approximately the top 13% of tracks in the dataset, 
capturing mainstream hits rather than moderately streamed songs.

**Evaluation Metric:** F1-score was chosen over accuracy because the dataset is 
heavily imbalanced. 87% of tracks are non-popular. A model that simply predicts 
"not popular" for every track would achieve 87% accuracy while being completely 
useless. F1-score balances precision and recall, penalizing models that ignore the 
minority class.

**Features at Time of Prediction:** All features used in the model are known at the 
time a track is released — audio features (`danceability`, `energy`, `loudness`) are 
computed by Spotify's algorithms from the audio file itself, `explicit` and 
`track_genre` are metadata known at release, and `release_year` is known at the time 
of release. Artist `followers` is a borderline case — follower count grows over time, 
but a baseline count is available at release. This is noted as a limitation of the model.

## Baseline Model

**Model:** Decision Tree Classifier

**Features:**
- `explicit` (nominal) — directly tied to the research question; passed through 
as-is since it is already binary (True/False)
- `track_genre` (nominal) — provides genre context; encoded using OneHotEncoder 
since it is a categorical variable with 5 unique values

**Encodings:** OneHotEncoder was applied to `track_genre` to convert genre labels 
into numerical columns the model can interpret. `explicit` was passed through 
as-is since it is already binary. Both transformations were implemented inside a 
single sklearn Pipeline to prevent data leakage.

**Class Imbalance:** `class_weight='balanced'` was used to address the 87% 
non-popular class imbalance. Without this, the model predicted all tracks as 
non-popular, resulting in an F1 score of 0.

**Performance:** The baseline model achieved an **F1 score of 0.332** on the 
held-out test set (20% of the data). This performance is expected to be low.
The model uses only two features and a simple decision tree, providing a 
benchmark for the final model to improve upon.

## Final Model

**Model:** Random Forest Classifier

**Features Added:**
- `danceability` (quantitative) — passed through as raw continuous value. 
Danceable songs are more likely to be played at social events and shared on 
platforms, driving streams and popularity.
- `energy` (quantitative) — passed through as raw continuous value. High energy 
songs tend to dominate playlists and radio, increasing their likelihood of 
accumulating streams.
- `loudness` (quantitative) — passed through as raw continuous value. Louder, 
more produced tracks tend to be more radio-ready, correlating with higher popularity 
due to modern music production trends.
- `release_year` (quantitative, binarized) — binarized at a threshold of 2010 to 
distinguish pre-streaming era tracks from streaming era tracks. Spotify's growth 
after 2010 significantly increased the potential audience for new releases, making 
this a meaningful categorical split.
- `followers` (quantitative) — imputed using median via `SimpleImputer` inside the 
pipeline to prevent data leakage. Artist follower count is a strong predictor of 
popularity since larger artists have built-in audiences that will stream their tracks 
regardless of audio features.

Raw continuous values were kept for audio features since Random Forest splits on 
thresholds rather than distances, making scaling unnecessary and potentially harmful 
by discarding information.

**Hyperparameter Tuning:** GridSearchCV with 5-fold cross validation was used to 
tune three hyperparameters:
- `n_estimators` — number of trees in the forest; more trees reduces variance
- `max_depth` — controls overfitting; too deep memorizes training data
- `min_samples_split` — minimum samples required to split a node; higher values 
prevent learning noise in small groups

**Best Hyperparameters:** `max_depth=10`, `min_samples_split=10`, `n_estimators=100`

**Performance:** The final model achieved an **F1 score of 0.421** on the same 
held-out test set used for the baseline model, representing a **26.8% relative 
improvement** over the baseline F1 of 0.332. The most impactful addition was the 
`followers` feature, which improved F1 from 0.364 to 0.421, reflecting the strong 
relationship between artist following and track popularity.

## Fairness Analysis

**Groups:**
- **Group X:** Explicit tracks
- **Group Y:** Non-explicit tracks

**Evaluation Metric:** Precision — measures how many tracks predicted as popular 
are actually popular. This metric was chosen because it directly captures whether 
the model is making reliable positive predictions for each group.

**Null Hypothesis:** Our model is fair. Its precision for explicit and non-explicit 
tracks are roughly the same, and any differences are due to random chance.

**Alternative Hypothesis:** Our model is unfair. Its precision for non-explicit 
tracks is lower than for explicit tracks.

**Test Statistic:** Difference in precision (explicit minus non-explicit). A 
one-tailed test was used since we observed that explicit tracks had higher precision 
and wanted to test whether this difference was statistically significant.

**Significance Level:** 0.05

**Results:**
- Precision (Explicit): 0.562
- Precision (Non-Explicit): 0.301
- Observed Difference: 0.262
- **P-value:** 0.008

With a p-value of 0.008, which is below the 0.05 significance threshold, we reject 
the null hypothesis. The results suggest that our model may not be fair — it achieves 
notably higher precision for explicit tracks than non-explicit tracks. This disparity 
likely reflects the association between explicit content and popularity that the model 
learned during training, making it better at identifying popular explicit tracks than 
popular non-explicit ones. However, since we are performing a statistical test and not 
a randomized controlled trial, we cannot conclude with certainty that our model is 
unfair — only that the observed difference is unlikely to be due to random chance alone.