## testbricks

> The ultimate aim of the project is to build a Proxy mock for the Spark object

The ultimate aim of the project is to build a Proxy mock for the Spark object

## Goals

A common Databricks pipeline uses the following commands to interact with Delta tables - 

1. spark.read.table("<schema_name>.<table_name>")

2. spark.write.table("<schema_name>.<table_name>")

3. spark.sql("SELECT * FROM <schema_name>.<table_name>")


The proxy replaces these functionalities with CSV files and Spark temp views.

## Directory structure

All code is placed inside `src` directory and all modules are placed inside their own folders with `__init__.py` files. The main mock Spark object / proxy object is inside `mock` folder



## Recent Changes
- 001-spark-mock-catalog: Added Python 3.14 + pyspark, pandas, pyarrow, numpy, py4j
- 001-spark-mock-catalog: Added [if applicable, e.g., PostgreSQL, CoreData, files or N/A]

## Active Technologies
- Python 3.14 + pyspark, pandas, pyarrow, numpy, py4j (001-spark-mock-catalog)
- Local files (CSV-compatible data + catalog registry) (001-spark-mock-catalog)

---
> Source: [engineeringmadness/testbricks](https://github.com/engineeringmadness/testbricks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
