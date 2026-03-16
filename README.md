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
- [Project Structure](#project-structure)
- [Future Improvements](#future-improvements)

---

## Overview

This project implements a **4-stage NLP pipeline** that collects live news headlines from three major Indian news outlets, performs sentiment analysis on each headline using TextBlob, stores the annotated records in a local SQLite database, and generates visualizations to surface sentiment patterns across sources.

The entire pipeline runs as a single self-contained Jupyter notebook, designed to be executed on **Google Colab** with no local setup required.

---

## Pipeline Architecture

```
┌─────────────────────┐
│   1. Data Collection │  RSS (feedparser) + HTML Scrape (BeautifulSoup)
└────────┬────────────┘
         ▼
┌─────────────────────┐
│  2. Sentiment Analysis│  TextBlob → polarity score + subjectivity score
└────────┬────────────┘
         ▼
┌─────────────────────┐
│   3. SQLite Storage  │  Persist records with timestamps
└────────┬────────────┘
         ▼
┌─────────────────────┐
│   4. Visualizations  │  Bar charts · Polarity histogram · Word cloud
└─────────────────────┘
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
| `requests` + `BeautifulSoup` | HTML scraping |
| `TextBlob` + `NLTK` | Sentiment analysis (polarity & subjectivity) |
| `pandas` | Data wrangling |
| `sqlite3` | Persistent local storage |
| `matplotlib` | Bar charts and polarity histogram |
| `wordcloud` | Word cloud generation |

---

## Getting Started

### Option 1 — Google Colab (Recommended)

1. Open `news_sentiment_pipeline.ipynb` in [Google Colab](https://colab.research.google.com/).
2. Run all cells (`Runtime → Run all`).
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

| Section | Description |
|---|---|
| **0. Setup** | Install packages and download NLTK corpora |
| **1. Data Collection** | Scrape RSS feeds (NDTV, TOI) and HTML (NewsLive) |
| **2. Sentiment Analysis** | Apply TextBlob to score each headline |
| **3. SQLite Storage** | Persist annotated records to `news_sentiment.db` |
| **4. Visualizations** | Sentiment breakdown, polarity distribution, word cloud |
| **5. Summary Table** | Per-source stats: avg polarity, avg subjectivity, counts |

---

## Sample Results

Results from a single run (58 headlines collected):

| Source | Headlines | Avg Polarity | Avg Subjectivity | Positive | Neutral | Negative |
|---|---|---|---|---|---|---|
| NDTV | 20 | 0.0436 | 0.2731 | 5 | 12 | 3 |
| Times of India | 20 | 0.0660 | — | 5 | 13 | 2 |
| NewsLive | 18 | 0.0329 | 0.2253 | 4 | 11 | 3 |
| **Total** | **58** | **0.0483** | — | **14** | **36** | **7** |

As expected, Indian news headlines are predominantly **neutral** in tone — factual reporting naturally skews polarity scores toward zero.

---

## Sentiment Scoring Logic

TextBlob produces a **polarity score** in the range `[-1.0, +1.0]`. The following thresholds are applied:

| Polarity Range | Label |
|---|---|
| `polarity > 0.05` | Positive |
| `polarity < -0.05` | Negative |
| `-0.05 ≤ polarity ≤ 0.05` | Neutral |

Thresholds are kept tight (±0.05) because news headlines are factual by nature and tend to cluster near zero.

---

## Project Structure

```
News-Sentiment-Pipeline/
│
├── news_sentiment_pipeline.ipynb   # Main pipeline notebook
├── news_sentiment.db               # SQLite database (auto-generated on run)
└── README.md
```

---

## Future Improvements

- [ ] Schedule periodic runs with `cron` or GitHub Actions for time-series tracking
- [ ] Add more news sources (The Hindu, Indian Express, Hindustan Times)
- [ ] Replace TextBlob with a fine-tuned BERT model for improved accuracy on news text
- [ ] Export results to CSV / Google Sheets for further analysis
- [ ] Build a simple Streamlit dashboard for interactive exploration

---

## License

This project is licensed under the [MIT License](LICENSE).
