# News-Sentiment-Pipeline

> End-to-end Indian news sentiment pipeline: scrapes live headlines from NDTV, Times of India & NewsLive TV via RSS feeds and BeautifulSoup, scores polarity & subjectivity with TextBlob NLP, persists timestamped records in SQLite, and visualizes sentiment distributions and word clouds across sources.

---

## Table of Contents

- [Overview](#overview)
- [Pipeline Architecture](#pipeline-architecture)
- [Data Sources](#data-sources)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Notebook Structure](#notebook-structure)
- [Sample Results](#sample-results)
- [Sentiment Scoring Logic](#sentiment-scoring-logic)
- [SQLite Storage](#sqlite-storage)
- [Project Structure](#project-structure)
- [Future Improvements](#future-improvements)

---

## Overview

This project implements a **4-stage NLP pipeline** that collects live news headlines from three major Indian news outlets, performs sentiment analysis on each headline using TextBlob, stores the annotated records in a local SQLite database, and generates visualizations to surface sentiment patterns across sources.

The entire pipeline runs as a single self-contained Jupyter notebook, designed to be executed on **Google Colab** with no local setup required.

---

## Pipeline Architecture

```
┌──────────────────────┐
│   1. Data Collection  │  RSS (feedparser) + HTML scrape (BeautifulSoup)
└─────────┬────────────┘
          ▼
┌──────────────────────┐
│  2. Sentiment Analysis│  TextBlob → polarity score + subjectivity score
└─────────┬────────────┘
          ▼
┌──────────────────────┐
│   3. SQLite Storage   │  Persist timestamped records — re-runnable, appendable
└─────────┬────────────┘
          ▼
┌──────────────────────┐
│   4. Visualizations   │  Stacked bar · Polarity histogram · Word cloud
└──────────────────────┘
```

---

## Data Sources

| Source | Method | Feed / URL |
|---|---|---|
| [NDTV](https://www.ndtv.com) | RSS via `feedparser` | `feeds.feedburner.com/ndtvnews-top-stories` |
| [Times of India](https://timesofindia.indiatimes.com) | RSS via `feedparser` | `timesofindia.indiatimes.com/rssfeedstopstories.cms` |
| [NewsLive TV](https://newslivetv.com) | HTML scrape via `BeautifulSoup` | `newslivetv.com` |

Up to **20 headlines per source** are collected per run. Duplicate headlines are dropped before analysis.

---

## Tech Stack

| Library | Purpose |
|---|---|
| `feedparser` | Parse RSS/Atom feeds |
| `requests` + `BeautifulSoup` | HTML scraping for NewsLive |
| `TextBlob` + `NLTK` | Sentiment scoring (polarity & subjectivity) |
| `pandas` | Data wrangling and summary stats |
| `sqlite3` | Persistent local storage with run history |
| `matplotlib` | Stacked bar chart and polarity histogram |
| `wordcloud` | Most frequent headline words |

---

## Getting Started

### Option 1 — Google Colab (Recommended)

1. Open `news_sentiment_pipeline.ipynb` in [Google Colab](https://colab.research.google.com/).
2. Run all cells — `Runtime → Run all`.
3. The first cell installs all dependencies automatically.

### Option 2 — Local

```bash
# Clone the repository
git clone https://github.com/abhinab44/News-Sentiment-Pipeline.git
cd News-Sentiment-Pipeline

# Install dependencies
pip install feedparser textblob wordcloud requests beautifulsoup4 pandas matplotlib
python -m textblob.download_corpora lite

# Launch the notebook
jupyter notebook news_sentiment_pipeline.ipynb
```

> **Python version:** 3.10+

---

## Notebook Structure

| Cell | Section | Description |
|---|---|---|
| 0 | Header | Pipeline overview and data sources |
| 1 | Setup | Install packages, download NLTK corpora |
| 2 | Imports | Load all libraries |
| 3–5 | **1. Data Collection** | Scrape NDTV & TOI via RSS; scrape NewsLive via BeautifulSoup; deduplicate |
| 6–7 | **2. Sentiment Analysis** | Apply TextBlob polarity & subjectivity; classify each headline |
| 8–10 | **3. SQLite Storage** | Persist annotated records to `news_sentiment.db`; query latest run |
| 11–14 | **4. Visualizations** | Stacked sentiment bar, polarity histogram, word cloud |
| 15 | **5. Summary Table** | Per-source stats: avg polarity, avg subjectivity, sentiment counts |

---

## Sample Results

Results from a single run — **57 unique headlines** collected (58 raw, 1 duplicate removed):

### Per-Source Breakdown

| Source | Headlines | Avg Polarity | Avg Subjectivity | Positive | Neutral | Negative |
|---|---|---|---|---|---|---|
| NDTV | 20 | 0.0436 | 0.2253 | 5 | 12 | 3 |
| Times of India | 20 | 0.0660 | 0.2445 | 5 | 13 | 2 |
| NewsLive | 17 | 0.0329 | 0.2731 | 4 | 11 | 2 |
| **Total** | **57** | **0.0483** | — | **14** | **36** | **7** |

### Overall Sentiment Distribution

| Sentiment | Count | Share |
|---|---|---|
| Neutral | 36 | 63.2% |
| Positive | 14 | 24.6% |
| Negative | 7 | 12.3% |

As expected, Indian news headlines are predominantly **neutral** in tone — factual reporting naturally skews polarity scores toward zero. Times of India shows the highest average polarity (0.0660), while NDTV scores lowest (0.0436). All three sources cluster heavily near zero in the polarity histogram.

### Sample Headlines

| Source | Headline | Polarity | Sentiment |
|---|---|---|---|
| NDTV | "Military Dictatorship, Clerical Facade": Farewell... | −0.100 | Negative |
| NDTV | Poll Body Appoints Ex-Bureaucrat Manjeet Singh... | +0.357 | Positive |
| Times of India | 'Distorted picture of India': MEA slams US report | −0.150 | Negative |
| Times of India | The invisible arc of power: How ballistic miss... | 0.000 | Neutral |
| NewsLive | As the Middle East War entered... | 0.000 | Neutral |

---

## Sentiment Scoring Logic

TextBlob produces a **polarity score** in the range `[−1.0, +1.0]`. The following thresholds are applied:

| Polarity Range | Label |
|---|---|
| `polarity > 0.05` | Positive |
| `polarity < −0.05` | Negative |
| `−0.05 ≤ polarity ≤ 0.05` | Neutral |

Thresholds are kept tight (±0.05) because news headlines are factual by nature and tend to cluster near zero. A wider threshold (e.g. ±0.1) would shift more borderline headlines into the neutral bucket.

TextBlob also returns a **subjectivity score** in `[0.0, 1.0]` — 0 = fully objective, 1 = fully subjective. Across all sources, average subjectivity ranges from 0.22 to 0.27, confirming the headlines are largely factual.

---

## SQLite Storage

Results are persisted to `news_sentiment.db` using Python's built-in `sqlite3`. The table schema is:

```sql
CREATE TABLE IF NOT EXISTS articles (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    headline        TEXT NOT NULL,
    source          TEXT NOT NULL,
    polarity        REAL,
    subjectivity    REAL,
    sentiment_label TEXT,
    scraped_at      TEXT
);
```

- The table uses `CREATE IF NOT EXISTS` — **the notebook is safely re-runnable**; each run appends a new batch of timestamped records.
- After 2 runs, the DB contains **114 records** — enabling trend analysis across sessions.
- A SQL query filters to the latest `scraped_at` timestamp for per-run reporting.

---

## Project Structure

```
News-Sentiment-Pipeline/
│
├── news_sentiment_pipeline.ipynb   # Main pipeline notebook
├── news_sentiment.db               # SQLite database (auto-generated on first run)
└── README.md
```

---

## Future Improvements

- [ ] Schedule periodic runs with `cron` or GitHub Actions for time-series sentiment tracking
- [ ] Add more sources: The Hindu, Indian Express, Hindustan Times
- [ ] Replace TextBlob with a fine-tuned BERT/RoBERTa model for improved accuracy on news text
- [ ] Add subjectivity breakdown charts alongside polarity visualizations
- [ ] Export results to CSV or Google Sheets for downstream analysis
- [ ] Build a Streamlit dashboard for interactive, filterable exploration

---

## License

This project is licensed under the [MIT License](LICENSE).

---

*Built with Python 3.10 · TextBlob · feedparser · BeautifulSoup · pandas · SQLite · Matplotlib · WordCloud*
