# FIFA World Cup Match Search Engine

> Course project for **Information Retrieval (IR)**, University of Isfahan.

A small information retrieval (IR) system for searching every FIFA World Cup match from **1930 to 2022**. It is built from scratch in Python: no search library such as Lucene or Elasticsearch is used.

You can search with plain keywords, boolean operators, structured field queries, and exact phrases. Results are ranked with TF-IDF, and a built-in evaluation suite scores the system with standard IR metrics.

## Features

- **Document builder.** Each CSV row (one match) becomes a readable text document. Messy semi-structured columns (goals, cards, substitutions, penalty shootouts) are parsed into natural sentences such as scorer, minute, and extra time.
- **Inverted index** with term frequencies and word positions, which enables exact phrase search.
- **Field index** for structured search such as `team:Argentina` or `referee:"Szymon Marciniak"`.
- **Query processor** with `AND`, `OR`, `NOT` (precedence `NOT > AND > OR`), quoted phrases, and field queries that can be mixed with boolean logic.
- **TF-IDF ranking** using sublinear term frequency and smoothed IDF:
  `score(q, d) = Σ log(1 + tf(t, d)) · log((N + 1) / (df(t) + 1))`
- **Evaluation suite** with 10 queries, automatically built relevance judgments, and the metrics Precision@K, Recall@K, F1@K, MAP, and NDCG@K.
- **Text preprocessing:** lowercasing, accent stripping, HTML-entity cleanup, punctuation removal, and stopword removal.

## Project structure

| File | Purpose |
|---|---|
| `main.py` | Command-line entry point (interactive, single query, evaluation) |
| `config.py` | Dataset path, CSV separator, column name mapping |
| `preprocess.py` | Text normalization, tokenization, stopword removal |
| `document_builder.py` | Turns CSV rows into searchable documents with fields and metadata |
| `index_builder.py` | `InvertedIndex` and `FieldIndex` |
| `query_processor.py` | Query parsing and boolean/field/phrase retrieval |
| `ranking.py` | `TFIDFRanker` |
| `evaluation.py` | Relevance judgments and IR metrics |

## Setup

Requires Python 3.9 or newer.

```bash
git clone https://github.com/<your-username>/worldcup-search-engine.git
cd worldcup-search-engine
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### Dataset

Place the dataset file `matches_1930_2022.csv` in the project root, next to `main.py`. It contains 964 matches, one per row. The path and column names can be changed in `config.py`.

## Usage

```bash
python main.py                          # interactive search
python main.py --query "messi"          # run a single query
python main.py --query "team:Argentina AND round:Final" --top_k 5
python main.py --evaluate               # run the full evaluation suite
```

In interactive mode, the commands are `:help`, `:eval`, and `:quit`.

## Query syntax

| Query | Meaning |
|---|---|
| `messi` | Keyword search |
| `mbappe AND goal` | Both conditions must match |
| `penalty OR shootout` | Either condition matches |
| `messi NOT france` | Messi, excluding documents matching France |
| `team:Argentina` | Field search |
| `referee:"Szymon Marciniak"` | Phrase inside a field |
| `"extra time"` | Exact phrase in the full text |
| `team:Argentina AND round:Final` | Fields combined with boolean logic |

**Searchable fields:** `team`, `home_team`, `away_team`, `round`, `stage`, `stadium`, `city`, `referee`, `captain`, `coach`, `player`, `score`, `year`, `host`.

Note: for a bare multi-word query without operators (for example `mbappe goal`), the terms are combined with OR, and TF-IDF ranking pushes documents that match more terms to the top.

## Evaluation

`python main.py --evaluate` runs 10 queries, including `messi`, `penalty shootout`, `round:Final`, `team:Argentina round:Final`, `extra time goal`, `referee:"Szymon Marciniak"`, `own goal`, `round:Semi-finals team:Morocco`, `0-0`, and `red card`.

Relevance judgments are generated programmatically by rule-based checks on the document text and metadata. They act as a *silver standard*, not as human annotations, so the scores show how consistently the engine matches those rules and are not a claim about absolute search quality.

