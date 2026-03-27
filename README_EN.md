# Academic Assistant Bot

**Telegram bot with two AI agents for scientific text processing**

A simple agent operating on fixed rules via prompt engineering — no RAG, no vector databases, no complex decision-making logic.

---

## Problem

Automate routine stages of preparing scientific articles for publication:
- Initial compliance check against journal requirements
- Translation of abstracts into academic English

## Features

### Reviewer Agent
Accepts article text and checks:
- **Structure** — title, authors, abstract, keywords, sections (introduction → methodology → results → conclusions), references
- **Formatting** — citation style, numbering, table/figure captions
- **Content** — research objective, methodology description, alignment of conclusions with stated goals
- **Formal requirements** — length, language consistency

Returns a structured checklist with ✅ / ⚠️ / ❌ ratings and a final recommendation (accept / revise / reject).

### Translation Agent
Translates Russian-language abstracts into academic English:
- Preserves accuracy and scientific style
- Uses established domain-specific terminology
- Provides notes on non-obvious terminological choices

## Architecture

```
┌─────────────────────────────────────────────┐
│                Telegram User                │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────┐
│              Telegram Bot (aiogram 3)        │
│                                              │
│  /review    → reviewer mode                  │
│  /translate → translation mode               │
│  /model     → switch LLM                     │
│                                              │
│  Text & file handling (.txt)                 │
│  Input validation (length, format)           │
│  Long response splitting (4096 chars)        │
└──────────────────┬───────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────┐
│             Agents Layer                     │
│                                              │
│  ┌───────────────┐  ┌─────────────────────┐  │
│  │   Reviewer    │  │   Translator        │  │
│  │               │  │                     │  │
│  │  System       │  │  System prompt      │  │
│  │  prompt with  │  │  with academic      │  │
│  │  checklist    │  │  translation        │  │
│  │  rules        │  │  requirements       │  │
│  └───────┬───────┘  └──────────┬──────────┘  │
│          │                     │              │
│          └──────────┬──────────┘              │
└─────────────────────┼────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────┐
│           OpenRouter API                     │
│                                              │
│  OpenAI-compatible endpoint                  │
│  Available models:                           │
│  Gemini Flash · Gemini Pro · Claude Sonnet   │
│  GPT-4o · GPT-4o Mini · DeepSeek V3         │
│                                              │
│  On-the-fly model switching via /model       │
└──────────────────────────────────────────────┘
```

## Project Structure

```
academic-bot/
├── bot.py              # Telegram bot: commands, message & file handling
├── agents.py           # Agent logic: LLM calls, input validation
├── prompts.py          # System prompts (reviewer + translator)
├── models.py           # Pydantic models for structured I/O
├── config.py           # Configuration via Pydantic Settings + .env
├── requirements.txt    # Dependencies
├── .env.example        # Environment variables template
└── .gitignore
```

## Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Language | Python 3.11+ | Core |
| Telegram API | aiogram 3 | Async bot framework |
| LLM API | OpenRouter + OpenAI SDK | Multi-model access via single API |
| Validation | Pydantic | Config and data typing |
| Configuration | pydantic-settings + dotenv | Secret management via .env |

## Key Technical Decisions

**Prompt engineering as the primary tool.** Each agent's behavior is entirely defined by a system prompt — a detailed instruction with output format, rules, and examples. No additional result processing logic.

**OpenRouter instead of direct API.** One key, one endpoint, OpenAI SDK compatibility — and access to models from Google, Anthropic, OpenAI, DeepSeek. Users can switch models on the fly via `/model`.

**Minimal but sufficient validation.** Pydantic for configuration, text length check before sending to LLM, long response splitting at line boundaries for Telegram.

## Example Output

**Reviewer** — user sends article text, bot returns:

```
### Overall Assessment
The article contains the main structural elements but has
several issues with formatting and content completeness.

### Checklist
| # | Category               | Status | Comment                        |
|---|------------------------|--------|--------------------------------|
| 1 | Title & authors        | ✅     | Present                        |
| 2 | Abstract               | ❌     | Missing                        |
| 3 | Keywords               | ❌     | Missing                        |
| 4 | Section structure      | ⚠️     | No literature review           |
| 5 | References             | ❌     | Missing                        |
| 6 | Research objective     | ✅     | Stated                         |
| 7 | Methodology            | ⚠️     | Briefly described              |

### Recommendation
Major revision required — abstract, keywords, literature
review, and references need to be added.
```

## How to Run

1. Create a bot via [@BotFather](https://t.me/BotFather)
2. Get an API key at [openrouter.ai](https://openrouter.ai)
3. Fill in `.env` (using `.env.example` as template)
4. `pip install -r requirements.txt`
5. `python bot.py`

## Status

✅ Working prototype — bot is deployed and tested on real scientific texts.

---

*Built as an example of a simple single-task AI agent driven by prompt engineering.*
