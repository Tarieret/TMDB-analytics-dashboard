# Film Industry EDA

*Streaming and studio decision-makers often conflate popularity with quality when greenlighting content; this analysis tests whether that assumption holds across 60,000+ films, with direct implications for content investment strategy.*

**Problem:** Determine whether genre, popularity, and release trends in film data reveal actionable patterns, and specifically test two industry assumptions: that popularity predicts quality, and that genre drives audience reception in a statistically meaningful way, across four decades of TMDB data.

**Architecture decision:** Loaded raw CSV data into a local SQLite database and queried with SQL before pulling into R for wrangling and visualization via tidyverse, rather than working directly from CSV in R. This kept large-scale filtering (1.38M rows down to the analysis subset) at the database layer instead of in-memory. Excluded financial data (budget/revenue) entirely despite it being in the raw dataset, since only ~17,000 of 1.38M entries had both fields reliably recorded, an unrepresentative subset not worth analyzing. Used a Welch two-sample t-test rather than a standard t-test for the Action vs. Drama comparison, since the two genre groups don't have equal variance. The R analysis was later expanded into an interactive Tableau dashboard built on the same cleaned data, with a single parameter bridging two otherwise-separate data sources so one decade control filters both a movie-level view and a genre-level view simultaneously.

**What broke:** SQLite stores R date objects as integers (days since 1970-01-01) rather than human-readable strings, so date range filtering had to be handled in R via `lubridate` after querying from SQLite instead of directly in SQL, to avoid silent misinterpretation of dates.

**Metric:** Popularity and rating are nearly uncorrelated (Pearson r = 0.087, p < 0.001), popularity does not predict quality. Drama movies rate significantly higher than Action (6.27 vs. 5.96, p < 0.001) across 35,000+ films, a small but statistically robust gap.

## Interactive Dashboard

**[View the live Tableau dashboard →](https://public.tableau.com/app/profile/tariere.timitimi/viz/tmbd-tableau-dashboard/FilmEDADashboard)**

![Demo: filtering by decade and clicking through genre comparisons](plots/film-eda-full.gif)

Built on the same cleaned dataset as the R analysis above, with three linked views (genre trends over time, popularity vs. rating, and average rating by genre), a shared decade parameter with an "All Decades" option, and click-to-filter genre comparison. Exploring the dashboard surfaced a finding the static R plots didn't call out directly: **Documentary and Music rank as the two highest-rated genres overall**, ahead of higher-volume genres like Drama and Comedy, a pattern that holds consistently across every decade filtered.

## Key Questions
- Which genres are most represented in the film industry?
- How have genre trends shifted across decades?
- Which genres receive the highest audience ratings?
- How have average ratings changed over time?
- Which movies have the highest popularity scores?
- Is there a relationship between a movie's popularity and its rating?
- Do Action and Drama movies receive significantly different ratings?

## Workflow
CSV → SQLite Database → SQL Query → R (tidyverse) → ggplot2 Visualizations → Tableau Dashboard

## Visualizations

| Plot | Description |
|------|-------------|
| `01_releases_per_year.png` | Number of movie releases per year (1980–2023) |
| `02_top_genres.png` | Top 10 genres by total movie count |
| `03_rating_by_genre.png` | Top 10 genres by average audience rating |
| `04_genre_trends_decade.png` | How the top 6 genres have grown or declined by decade |
| `05_rating_by_decade.png` | Average audience rating per decade |
| `06_most_popular.png` | Top 10 most popular movies by TMDB popularity score |

## Preview

![Genre Trends by Decade](plots/04_genre_trends_decade.png)
![Top 10 Genres](plots/02_top_genres.png)
![Most Popular Movies](plots/06_most_popular.png)

## Statistical Analysis

**Pearson Correlation — Popularity vs. Rating**
Tests whether a movie's popularity score is related to its audience rating.

**Welch Two-Sample T-Test — Action vs. Drama Ratings**
Tests whether Action and Drama movies receive statistically different audience ratings.
- H₀: No significant difference in ratings between Action and Drama movies
- H₁: A significant difference exists

## Key Findings

**Popularity vs. Rating (Pearson Correlation, r = 0.087)**
Statistically significant but practically negligible relationship (r = 0.087, p < 0.001). Popularity and quality are largely independent, a movie can be widely watched without being highly rated, and vice versa.

**Action vs. Drama Ratings (Welch T-Test, p < 0.001)**
Drama movies rate statistically higher than Action movies (6.27 vs. 5.96 average rating, p < 0.001). The 0.31-point difference is small but highly significant across 35,000+ films, suggesting audiences consistently reward narrative-driven films over action-oriented ones.

**Documentary and Music rank highest overall (Tableau dashboard)**
Across every decade tested, Documentary and Music consistently rank as the two highest-rated genres by average audience score, ahead of higher-output genres like Drama, Comedy, and Action. Horror consistently ranks lowest across every decade tested, a stable pattern rather than an artifact of any single time period.

## Data & Filtering Decisions

**Source:** [TMDB Movies Dataset on Kaggle](https://www.kaggle.com/datasets/asaniczka/tmdb-movies-dataset-2023-930k-movies), 1.38M rows, 24 columns

**Filters applied:**
- `status = 'Released'`, excludes movies still in production, planned, or cancelled
- `release_year >= 1980 AND release_year <= 2023`, focuses on four decades of modern cinema
- `vote_count >= 10`, removes entries with insufficient community ratings, reducing noise from obscure or low-quality entries

**Note on date filtering:** SQLite stores R date objects as integers (days since 1970-01-01) rather than human-readable strings. Date range filtering was handled in R using `lubridate` after querying from SQLite, rather than directly in SQL.

**Note on financial data:** Budget and revenue fields were excluded from this analysis. Only ~17,000 of 1.38M entries had both budget and revenue recorded, insufficient for meaningful analysis.

## Tech Stack
- **Language:** R
- **Packages:** tidyverse, ggplot2, lubridate, scales, ggrepel, DBI, RSQLite, readr
- **Database:** SQLite (via DBI + RSQLite)
- **Analysis:** Pearson Correlation, Welch Two-Sample T-Test
- **Visualization:** ggplot2 (static), Tableau (interactive dashboard)

## Reflection
A big dataset doesn't mean every column is usable. Deciding what to exclude, and being able to explain why, mattered as much as the statistical findings themselves. Adding an interactive layer on top of a finished statistical analysis also surfaced a pattern (Documentary/Music's consistently high ratings) that the original static plots hadn't highlighted, a reminder that a second visualization pass on the same data isn't purely redundant.
