# 🤖 AI Data Agent

A multi-agent data engineering system built with LangGraph that routes natural language 
queries to either a SQL Analyst Agent (for database queries) or an ETL Analyst Agent 
(for data extraction and transformation).

Originally built following [this tutorial](https://youtu.be/7yOmi4IX-Rs), then migrated 
to run entirely on OpenAI models and debugged end-to-end.

## What it does

You type a plain-English request — "show me the top 5 users by rating" or "extract data 
from this API and save it as CSV" — and the system:

1. **Routes** the query (SQL vs ETL) using a classifier agent
2. **SQL path**: fetches live database schema → generates SQL → runs a safety check 
   (blocks INSERT/UPDATE/DELETE/DROP) → executes → returns a plain-English answer
3. **ETL path**: extracts data from an API or transforms an existing file using 
   LLM-generated Pandas code, saved as CSV/JSON/Parquet

## Architecture

```
Data Agent (Router)
      │
      ├──► SQL Analyst Agent
      │      Curate question → Fetch schema → Generate SQL → Safety check → Execute → Answer
      │
      └──► ETL Analyst Agent
             Tool-calling agent with extract_load_tool and transform_load_tool
```

## Tech stack

- **LangGraph** — multi-agent orchestration and state management
- **LangChain** — LLM framework
- **OpenAI (GPT-5.6 family)** — all agents run on OpenAI models
- **PostgreSQL** — database backend, schema introspected at query time
- **Pydantic** — structured output validation for routing and safety decisions
- **Pandas** — data transformation

## Setup

```bash
python -m venv .venv
.\.venv\Scripts\Activate.ps1        # Windows
pip install -e .

# Create a .env file with:
# OPENAI_API_KEY=your_key
# host=localhost
# port=5432
# user=postgres
# password=your_password
# database=data_agent_db

python feed_db.py     # loads sample CSVs into Postgres
python main.py
```

## Notes from building this

This project involved debugging several real issues after cloning and adapting it — 
module-level code running on import (causing crashes on unrelated imports), a 
missing/mismatched virtual environment after a Python reinstall, and cleaning up 
LangGraph state so messages aren't duplicated across nodes. Each is a pattern worth 
knowing when building production agent pipelines.
