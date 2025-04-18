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
![streamlit demo](streamlit.png)

## 🧾 How to Run

```bash
import streamlit as st
import pandas as pd
import spacy
from newsapi import NewsApiClient
from google.cloud import bigquery
from datetime import datetime
from fredapi import Fred

# Set API keys and constants
NEWS_API_KEY = "006b9ceb1b1346f58c32e48343c2a45f"
FRED_API_KEY = "a5e3f3993ed8b708f5851781eaa1a2ac"
GCP_PROJECT_ID = "test-2-452103"
BUCKET_NAME = "credit-news-bucket"
BIGQUERY_DATASET = "credit_insight"
BIGQUERY_TABLE = "news_summary"

# Initialize clients
newsapi = NewsApiClient(api_key=NEWS_API_KEY)
fred = Fred(api_key=FRED_API_KEY)
nlp = spacy.load("en_core_web_sm")

def run_etl_pipeline():
    # Step 1: Collect news articles
    st.write("Collecting news articles...")
    news_data = newsapi.get_everything(q='credit', language='en', page_size=1)
    articles = news_data['articles']

    # Step 2: NLP summarization
    st.write("Analyzing and summarizing articles...")
    results = []
    for article in articles:
        description = article['description'] or ""
        doc = nlp(description)
        summary = " ".join([sent.text for sent in doc.sents][:2])
        results.append({
            "title": article['title'],
            "summary": summary,
            "publishedAt": article['publishedAt']
        })

    df = pd.DataFrame(results)

    # Fix: Ensure datetime format for BigQuery compatibility
    df["publishedAt"] = pd.to_datetime(df["publishedAt"])

    # Step 3: Upload to BigQuery
    st.write("Uploading summary to BigQuery...")
    client = bigquery.Client(project=GCP_PROJECT_ID)
    table_id = f"{GCP_PROJECT_ID}.{BIGQUERY_DATASET}.{BIGQUERY_TABLE}"
    job = client.load_table_from_dataframe(df, table_id)
    job.result()

    st.success("✅ ETL completed! News summary uploaded to BigQuery.")
    st.dataframe(df)

def main():
    st.title("📰 Credit News ETL Dashboard")
    st.markdown("This dashboard collects financial news using NewsAPI, summarizes it with spaCy, and loads the result into Google BigQuery.")

    if st.button("Run ETL"):
        try:
            run_etl_pipeline()
        except Exception as e:
            st.error(f"An error occurred: {e}")

if __name__ == "__main__":
    main()

from datetime import datetime

def log_execution():
    with open("execution_log.csv", "a") as f:
        f.write(f"{datetime.now().isoformat()}\n")

if st.button("Run ETL Now"):
    run_etl_pipeline()
    log_execution()



