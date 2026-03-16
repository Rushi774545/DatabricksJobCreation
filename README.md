1️⃣ Prepare Your Hourly Notebook

Your notebook should contain the full pipeline code like:

%pip install yfinance

import yfinance as yf
import pandas as pd

tickers = [
"AAPL","MSFT","GOOGL","AMZN","TSLA",
"META","NVDA","JPM","V","NFLX"
]

all_data = []

for ticker in tickers:

    df = yf.download(
        ticker,
        period="1d",
        interval="1h"
    )

    df = df.reset_index()

    # Fix multi index columns
    df.columns = [col[0] if isinstance(col, tuple) else col for col in df.columns]

    df["ticker"] = ticker

    all_data.append(df)

final_df = pd.concat(all_data)

final_df.columns = [
    c.lower().replace(" ", "_") for c in final_df.columns
]

spark_df = spark.createDataFrame(final_df)

spark.sql("USE CATALOG workspace")
spark.sql("USE SCHEMA default")

spark_df.write \
    .format("delta") \
    .mode("append") \
    .saveAsTable("hourly_stock_data")

Save this notebook as:

hourly_stock_pipeline
2️⃣ Go to Databricks Jobs

Left sidebar:

Workflows

Click:

Create Job
3️⃣ Configure Job

Fill these fields.

Job Name
Hourly Stock Ingestion
Task

Choose

Notebook

Select your notebook:

hourly_stock_pipeline
4️⃣ Choose Compute

Select either:

Serverless

OR

Existing Cluster

Recommended for learning:

Serverless
5️⃣ Add Schedule (Important)

Click Add Trigger

Choose:

Scheduled

Set:

Period → Hours
Every → 1 Hour

Example:

Run every 1 hour

Timezone:

Asia/Kolkata
6️⃣ Final Job Configuration

Your job will look like:

Job Name:
Hourly Stock Ingestion

Task:
Notebook → hourly_stock_pipeline

Compute:
Serverless

Schedule:
Every 1 Hour

Click:

Create
7️⃣ What Happens Now

Every 1 hour Databricks will run:

yfinance → pandas → spark → delta table

and update:

workspace.default.hourly_stock_data
8️⃣ Check Job Runs

Go to:

Workflows → Job Runs

You will see logs like:

Succeeded
Running
Failed
9️⃣ Check Updated Data

Run in a notebook:

spark.sql("""
SELECT ticker, COUNT(*) 
FROM workspace.default.hourly_stock_data
GROUP BY ticker
""").show()
🔥 Important Improvement (Recommended)

Right now you are re-downloading the same 1-day data every hour which will create duplicates.

Better approach:

Use MERGE or DropDuplicates.

Example:

spark_df = spark_df.dropDuplicates(["datetime","ticker"])

before writing.

🚀 Next Step (Very Important)

Next you should create:

Daily Stock Job

Schedule:

Run every 24 hours

I can also show you a much more professional architecture used by real data engineers:

Bronze Layer
raw_stock_data

Silver Layer
clean_stock_data

Gold Layer
analytics tables

This will make your Databricks project look production-level for interviews.

If you want, I can also show you how to build the entire stock pipeline architecture.

Daily Notebook

Below is the complete setup for creating a Databricks Job for your DAILY stock data pipeline, similar to the hourly one but scheduled once per day.

1️⃣ Prepare Your Daily Notebook

Your notebook should contain the pipeline code that fetches daily stock prices and stores them in the Delta table daily_stock_data.

%pip install yfinance

import yfinance as yf
import pandas as pd

tickers = [
"AAPL","MSFT","GOOGL","AMZN","TSLA",
"META","NVDA","JPM","V","NFLX"
]

all_data = []

for ticker in tickers:

    df = yf.download(
        ticker,
        period="30d",
        interval="1d"
    )

    df = df.reset_index()

    # Fix multi-index columns
    df.columns = [col[0] if isinstance(col, tuple) else col for col in df.columns]

    df["ticker"] = ticker

    all_data.append(df)

final_df = pd.concat(all_data)

# Clean column names
final_df.columns = [
    c.lower().replace(" ", "_") for c in final_df.columns
]

spark_df = spark.createDataFrame(final_df)

spark.sql("USE CATALOG workspace")
spark.sql("USE SCHEMA default")

# Remove duplicates before writing
spark_df = spark_df.dropDuplicates(["date","ticker"])

spark_df.write \
    .format("delta") \
    .mode("append") \
    .saveAsTable("daily_stock_data")

Save the notebook as:

daily_stock_pipeline
2️⃣ Go to Databricks Jobs

Left sidebar → Workflows

Click:

Create Job
3️⃣ Configure Job
Job Name
Daily Stock Ingestion
Task Type

Select:

Notebook

Choose notebook:

daily_stock_pipeline
4️⃣ Select Compute

Choose one:

Serverless

or

Existing Cluster

For most users:

Serverless
5️⃣ Add Schedule

Click Add Trigger

Select:

Scheduled

Configure:

Period → Days
Every → 1 Day

Example schedule:

Run every day at 18:00
Timezone → Asia/Kolkata

This means your pipeline runs once per day.

6️⃣ Final Job Configuration

Your job should look like this:

Job Name:
Daily Stock Ingestion

Task:
Notebook → daily_stock_pipeline

Compute:
Serverless

Schedule:
Every 1 Day
Timezone: Asia/Kolkata

Click:

Create
7️⃣ What Happens Now

Every 24 hours, Databricks will run:

yfinance → pandas → spark → delta table

and update:

workspace.default.daily_stock_data
8️⃣ Verify Data Updates

Run this query in a notebook:

spark.sql("""
SELECT ticker, COUNT(*) 
FROM workspace.default.daily_stock_data
GROUP BY ticker
""").show()
9️⃣ Final Project Structure

Your Databricks project should now look like:

Databricks Workspace
        │
        ├── hourly_stock_pipeline notebook
        │       ↓
        │   Job → Runs every 1 hour
        │
        ├── daily_stock_pipeline notebook
        │       ↓
        │   Job → Runs every 24 hours
        │
        ▼
Unity Catalog
workspace.default
        │
        ├── hourly_stock_data
        └── daily_stock_data
