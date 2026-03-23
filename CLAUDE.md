# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

This project uses `uv` for package management.

```bash
# Install dependencies
uv sync

# Run main.py
uv run python main.py

# Launch Jupyter for notebook experimentation
uv run jupyter notebook
```

## Architecture

This is an early-stage experimentation project exploring [LangMem](https://langchain-ai.github.io/langmem/) — LangChain's memory management library for LLM agents.

**Key libraries:**
- `langmem` — memory extraction and storage from conversations
- `langgraph` — agent graph orchestration (`@entrypoint`, `InMemoryStore`)
- `langchain-google-genai` — Gemini models (`gemini-2.5-flash`, `gemini-embedding-001`)

**Core pattern (see `experiment.ipynb`):**
1. A `LangGraph` entrypoint wraps an LLM chat function with a shared `InMemoryStore`
2. `create_memory_store_manager` extracts and stores memories from conversations
3. `ReflectionExecutor` defers memory processing (with a configurable delay) so that if new messages arrive, the pending task is cancelled and rescheduled with accumulated context

**Environment:** Configure `GOOGLE_API_KEY` and `GOOGLE_GENAI_USE_VERTEXAI` in `.env` (already gitignored).
