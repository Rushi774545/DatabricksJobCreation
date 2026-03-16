Creating Databricks Jobs for Hourly and Daily Stock Ingestion

This project uses Databricks Workflows (Jobs) to automatically run notebooks that fetch stock data using yfinance and store it in Delta tables inside Unity Catalog (workspace.default).

Two automated jobs are created:

Hourly Stock Job → runs every 1 hour

Daily Stock Job → runs once per day

1. Create Hourly Stock Job
Step 1: Open Workflows

Go to the Databricks Workspace.

In the left sidebar click:

Workflows

Click:

Create Job
Step 2: Configure Job Details

Set the following:

Job Name: Hourly Stock Ingestion
Step 3: Add Task

Click Add Task

Select:

Notebook

Configure the task:

Task Name: hourly_stock_pipeline_task
Notebook Path: /Workspace/hourly_stock_pipeline
Step 4: Select Compute

Choose a compute resource:

Serverless (Recommended)

or

Existing Cluster
Step 5: Configure Schedule

Click:

Add Trigger

Choose:

Scheduled

Set the schedule:

Period: Hours
Every: 1 Hour
Timezone: Asia/Kolkata

This will run the pipeline every hour.

Step 6: Create Job

Click:

Create

The job will now automatically update the table:

workspace.default.hourly_stock_data
2. Create Daily Stock Job
Step 1: Open Workflows

Navigate to:

Workflows → Create Job
Step 2: Configure Job Details

Set:

Job Name: Daily Stock Ingestion
Step 3: Add Task

Select:

Notebook

Configure:

Task Name: daily_stock_pipeline_task
Notebook Path: /Workspace/daily_stock_pipeline
Step 4: Select Compute

Choose:

Serverless

or

Existing Cluster
Step 5: Configure Schedule

Click:

Add Trigger → Scheduled

Set:

Period: Days
Every: 1 Day
Time: 18:00
Timezone: Asia/Kolkata

This will run the pipeline once every day.

Step 6: Create Job

Click:

Create

The job will now automatically update the table:

workspace.default.daily_stock_data
3. Verify Job Execution

To monitor job runs:

Go to:

Workflows

Select the job:

Hourly Stock Ingestion
or
Daily Stock Ingestion

View the Runs tab to check:

Running
Succeeded
Failed
4. Verify Updated Data

Run the following query in a Databricks notebook:

SELECT ticker, COUNT(*) 
FROM workspace.default.hourly_stock_data
GROUP BY ticker;

or

SELECT ticker, COUNT(*) 
FROM workspace.default.daily_stock_data
GROUP BY ticker;
Final Architecture
Databricks Workflows
        │
        ├── Hourly Stock Job
        │        │
        │        └── hourly_stock_pipeline notebook
        │
        ├── Daily Stock Job
        │        │
        │        └── daily_stock_pipeline notebook
        │
        ▼
Unity Catalog
workspace.default
        │
        ├── hourly_stock_data
        └── daily_stock_data
