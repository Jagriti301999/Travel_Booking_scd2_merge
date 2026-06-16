# Travel Bookings Data Ingestion Pipeline With SCD2 Merge 

Tech Stack - Databricks, PySpark, Google Storage, Delta Lake, Databricks Workflows, PyDeequ

Create bookings_fact & customers_dim delta tables

Read current date's (user provided date) bookings & customers csv file in Spark dataframe

Perform Data Quality checks on bookings & customers dataframe using PyDeequ library

Continue Or Stop data pipeline based on Checks failure/success

Run aggregations on bookings dataframe and update final aggregations in bookings_fact table using Merge query

Perform SCD2 Merge query on customers_dim table using customers dataframe to maintain history of changes

Setup scheduled databricks workflow with input key-value parameter to pass date value manually

