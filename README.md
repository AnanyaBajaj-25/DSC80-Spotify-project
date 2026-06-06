# Are Explicit Tracks More Popular?
### A Spotify Audio Features Analysis
**By Ananya Bajaj**
**Project for DSC 80 at UCSD**

## Introduction

In this project, I examined a dataset of Spotify tracks spanning 5 musically distinct 
genres: classical, metal, hip-hop, country, and EDM. The dataset was collected via the 
Spotify Web API and adapted from the 
[Spotify Dataset 1921-2020](https://www.kaggle.com/datasets/yamaerenay/spotify-dataset-19212020-600k-tracks) 
by Yamac Eren Ay on Kaggle.

My central research question is: **Do explicit tracks tend to be more popular than 
non-explicit tracks?**

This question has real implications beyond data science. Explicit content has long been 
a point of cultural debate from parental advisory labels to platform content filters 
today. Explicit tracks are excluded from certain Spotify playlists and filtered for 
younger listeners, which could limit their reach. Yet genres like hip-hop and metal 
treat explicit language as core to their artistic identity. If explicit tracks are 
systematically more popular despite these restrictions, it raises important questions 
about how streaming algorithms shape what gets heard and who benefits.

The dataset was constructed by merging two source files: `music_tracks.csv` 
(track-level audio features and metadata) and `artists.csv` (artist-level 
follower counts), joined on primary artist name.

The cleaned dataset contains **5,505 rows**, where each row represents one track. 
The columns most relevant to my question are:

| Column | Description |
|---|---|
| `track_name` | Name of the track |
| `track_genre` | Genre of the track (classical, metal, hip-hop, country, or EDM) |
| `popularity` | Spotify popularity score from 0–100, based on total plays and recency |
| `popular` | Binary indicator of whether popularity ≥ 70 |
| `danceability` | How suitable the track is for dancing, based on tempo and rhythm (0–1) |
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
to represent a wide range of explicit content rates — from genres with almost no 
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

