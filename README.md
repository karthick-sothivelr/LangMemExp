# LangMemExp

A hands-on exploration of [LangMem](https://langchain-ai.github.io/langmem/) — LangChain's memory management library for LLM agents. This repository contains a series of Jupyter notebooks that progressively cover different memory patterns, from basic background extraction to multi-prompt optimization.

## Prerequisites

- Python 3.13+
- [uv](https://github.com/astral-sh/uv) package manager
- A Google API key with access to Gemini models

## Setup

```bash
# Clone the repository
git clone https://github.com/karthick-sothivelr/LangMemExp.git
cd LangMemExp

# Install dependencies
uv sync

# Configure environment variables
cp .env.example .env  # then fill in your values
```

### Environment Variables

Create a `.env` file in the project root:

```env
GOOGLE_API_KEY=your_google_api_key_here
GOOGLE_GENAI_USE_VERTEXAI=false  # set to true if using Vertex AI
```

## Running Notebooks

```bash
uv run jupyter notebook
```

## Key Libraries

| Library | Purpose |
|---|---|
| `langmem` | Memory extraction and storage from conversations |
| `langgraph` | Agent graph orchestration (`@entrypoint`, `InMemoryStore`) |
| `langchain-google-genai` | Gemini models (`gemini-2.5-flash`, `gemini-embedding-001`) |

## Notebooks

### Long-Term Memory

| Notebook | Topic | Summary |
|---|---|---|
| [1_background_memory_management.ipynb](1_background_memory_management.ipynb) | Background Memory | The LLM responds immediately while a `ReflectionExecutor` asynchronously extracts and stores memories. Low latency, but new memories aren't available until the next turn. |
| [2_1_hotpath_memory_management.ipynb](2_1_hotpath_memory_management.ipynb) | Hot-Path Memory | Memory extraction runs synchronously before the response, so newly learned facts are immediately available in the same conversation turn. Higher latency, but richer personalisation. |
| [2_2_hotpath_memory_langmem_doc.ipynb](2_2_hotpath_memory_langmem_doc.ipynb) | Agent-Controlled Memory | Uses `create_manage_memory_tool` and `create_search_memory_tool` so the agent explicitly decides what to remember via ReAct-style tool calls, with a `@dynamic_prompt` injecting recalled memories before each LLM call. |
| [3_TypesMemoryExample.ipynb](3_TypesMemoryExample.ipynb) | Memory Schemas | Explores different memory representations: free-form semantic collections, structured Pydantic profiles, knowledge triples (subject–predicate–object), episodic memories (situation–thought–action–result), and procedural/system-prompt memories. |
| [4_memory_manager_agent.ipynb](4_memory_manager_agent.ipynb) | Memory Manager Agent | A dual-agent architecture where a user-facing LLM handles conversation quality while a separate ReAct memory manager agent reasons about which memories to create, update, or delete. |

### Short-Term Memory

| Notebook | Topic | Summary |
|---|---|---|
| [8_SummarizeMessages.ipynb](8_SummarizeMessages.ipynb) | Message Summarization | Three approaches to managing long conversations: inline `summarize_messages()`, a dedicated `SummarizationNode`, and summarization embedded in a ReAct agent loop. Covers the lossy-compression gotcha where critical facts can be dropped from summaries. |

### Prompt Optimization

| Notebook | Topic | Summary |
|---|---|---|
| [5_SinglePromptOptimizer.ipynb](5_SinglePromptOptimizer.ipynb) | Single Prompt Optimizer | Uses `create_prompt_optimizer` to automatically improve a system prompt from conversation trajectories. Covers three strategies: `prompt_memory` (fast, 1 call), `metaprompt` (balanced, up to 5 calls), and `gradient` (thorough, up to 10 calls with iterative failure analysis). |
| [6_MultiPromptOptimizer.ipynb](6_MultiPromptOptimizer.ipynb) | Multi-Prompt Optimizer | Extends optimization to multi-stage pipelines with `create_multi_prompt_optimizer`, which traces failures back to the specific prompt responsible — preventing updates to working prompts from making things worse. |

### Namespacing & Multi-Tenancy

| Notebook | Topic | Summary |
|---|---|---|
| [7_ConfigureDynamicNameSpaces.ipynb](7_ConfigureDynamicNameSpaces.ipynb) | Dynamic Namespaces | Organises memories in hierarchical namespaces for multi-user and multi-org scenarios (e.g. `("memories", "{user_id}")`, `("memories", "{org_id}", "{user_id}")`). Namespace templates are populated from LangGraph's runtime config at invoke time. |

## Core Architecture

```
User message
     │
     ▼
LangGraph @entrypoint  ──►  InMemoryStore (vector DB)
     │                              ▲
     │                              │
     ▼                    ReflectionExecutor / MemoryManager
  LLM response  ◄──  recalled memories injected into system prompt
```

1. A LangGraph entrypoint wraps an LLM chat function with a shared `InMemoryStore`.
2. `create_memory_store_manager` extracts and stores memories from conversations.
3. `ReflectionExecutor` defers memory processing so that if new messages arrive quickly, the pending task is cancelled and rescheduled with accumulated context.
