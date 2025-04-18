# Structured-Credit-News-ETL-Dashboard

## Project Overview
This project automates the extraction and summarization of structured credit-related news using Python and deploys it through a user-friendly Streamlit dashboard.

## Features
- Real-time news collection from NewsAPI
- NLP-based summarization with spaCy
- Storage in Google BigQuery
- Streamlit UI for manual ETL trigger
- Execution log for tracking usage

## Tech Stack
- Python
- Streamlit
- spaCy (NLP)
- Google BigQuery
- NewsAPI

## Data Flow
`NewsAPI → spaCy → Pandas → BigQuery → Streamlit UI`

## Output Example
![streamlit demo](./streamlit.png)

## 🧾 How to Run

```bash
pip install -r requirements.txt
streamlit run etl_app.py
