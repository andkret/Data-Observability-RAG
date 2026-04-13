# Pipeline Failure Intelligence

**Build an AI agent that answers natural language questions about your data pipeline failures.**

Your pipelines fail. Logs are scattered across task runs. Root cause analysis is manual,
slow, and happens after the damage is done.

This project builds a LangChain agent that reasons over 90 days of pipeline run history —
combining structured SQL queries against job metadata with semantic vector search over log
text — all stored in a single Oracle AI Database instance. No separate vector store.
No extra infrastructure. One database, one query language, one agent.

---

## What the Agent Can Answer

- *"Which DAG failed most often in the last 30 days, and what error codes appeared?"*
- *"What exactly changed in the schema that caused downstream transforms to fail?"*
- *"Why does the weekly tip analysis job keep getting slower?"*
- *"Are there silent data quality issues that didn't cause failures but should be investigated?"*
- *"Compare Monday vs weekday pipeline durations — is there a pattern?"*
- *"Give me a full reliability report for the borough partition load DAG."*

The agent selects from three tools depending on the question:

| Tool | When used |
|------|-----------|
| `pipeline_sql_tool` | Counts, durations, error codes, date ranges |
| `log_search_tool` | Schema changes, error descriptions, silent warnings in log text |
| `hybrid_pipeline_search` | Combines a DAG/date filter with semantic log search in one Oracle query |

---

## Prerequisites

### Docker Desktop

Install and start [Docker Desktop](https://www.docker.com/products/docker-desktop/).

### Ollama — two models required

Install [Ollama](https://ollama.com/) and pull both models before starting:

```bash
ollama pull mistral          # LLM for agent reasoning (or: ollama pull qwen2.5)
ollama pull nomic-embed-text # Dedicated embedding model for vector search
```

> **Why a separate embedding model?**
> Chat models like `mistral` are trained to generate coherent text, not to produce
> semantically precise vector representations. When used for embeddings, two log messages
> that mean the same thing may land far apart in the vector space, producing poor
> similarity search results. `nomic-embed-text` is a purpose-built embedding model
> (~274 MB) trained specifically for semantic similarity tasks. Always use a dedicated
> embedding model for vector search — never the chat model.

---

## Quickstart

From the `pipeline_failure_intelligence/` directory:

```bash
docker compose up --build
```

Wait until Oracle shows as **healthy**:

```bash
docker ps | grep oracle-26ai
# oracle-26ai   ...   (healthy)
```

Then open Jupyter in your browser:

```
http://localhost:8888
```

The default token is printed in the `jupyter` container logs:

```bash
docker logs jupyter 2>&1 | grep token
```

Open `notebooks/pipeline_failure_intelligence.ipynb` and run cells top to bottom.

---

## Switching to OpenAI

The config cell (Cell 2) is the only cell you ever need to edit. To use OpenAI instead
of Ollama, set these four environment variables before starting the container, or edit
the cell directly:

```python
LLM_BASE_URL    = "https://api.openai.com/v1"
LLM_MODEL       = "gpt-4o"
LLM_API_KEY     = "sk-..."          # your OpenAI API key
EMBEDDING_MODEL = "text-embedding-3-small"
```

All LangChain code is identical — only the config changes.

---

## Project Structure

```
pipeline_failure_intelligence/
  docker-compose.yml          # Oracle AI Database + Jupyter services
  jupyter/
    Dockerfile                # scipy-notebook + LangChain + oracledb
  notebooks/
    pipeline_failure_intelligence.ipynb   # the full demo
  README.md
```

---

## Resources

- [Oracle AI Database 26ai Free Tier](https://www.oracle.com/database/free/)
- [Oracle Developer Resources](https://www.oracle.com/developer/resources/)
- [langchain-oracledb Documentation](https://python.langchain.com/docs/integrations/vectorstores/oracle/)
- [Oracle AI Developer Hub](https://github.com/oracle-devrel/oracle-ai-developer-hub)

---

## License

Copyright (c) 2024 Oracle and/or its affiliates.
Licensed under the [Universal Permissive License v 1.0](https://oss.oracle.com/licenses/upl/).
