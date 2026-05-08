# AI Customer Support Agent

An AI-powered customer support automation system built with LangChain and LangGraph. The system reads incoming support emails, classifies them by urgency and topic, searches a knowledge base, decides whether to auto-reply or escalate to a human agent, drafts a response, and schedules a follow-up — all as a deterministic, stateful pipeline.

Three notebook variants are provided, each using a different LLM provider while sharing the same architecture.

---

## Files

| File | Provider | Model |
|---|---|---|
| `ai_customer_support_agent.ipynb` | Anthropic | claude-3-5-haiku-20241022 |
| `ai_customer_support_agent_openai.ipynb` | OpenAI | gpt-4o-mini |
| `ai_customer_support_agent_gemini.ipynb` | Google | gemini-2.5-flash |

---

## Pipeline Architecture

Every email passes through a five-node LangGraph pipeline in fixed order:

```
START
  |
  v
[classify]     — LLM classifies urgency (Low/Medium/High) and topic
  |
  v
[search_kb]    — TF-IDF cosine similarity search over 11 KB articles
  |
  v
[escalation]   — 7 deterministic rules produce auto-reply or escalate
  |
  v
[draft]        — LLM drafts the outgoing response with tone instruction
  |
  v
[followup]     — Lookup matrix schedules a follow-up or skips it
  |
  v
END
```

State is a `TypedDict` (`SupportAgentState`) passed through the graph. Each node receives the full state and returns only the keys it updates.

---

## Classification

The `classify` node calls the LLM via a `ChatPromptTemplate` chain with `with_structured_output()` bound to a Pydantic model. Output is always one of the following values.

**Urgency**

| Value | Criteria |
|---|---|
| High | Financial errors, service outages, data loss, security breaches |
| Medium | Bugs blocking functionality, persistent API failures, account access issues |
| Low | How-to questions, feature requests, general enquiries |

**Topic**

Account, Billing, Bug, Feature Request, Technical Issue

---

## Escalation Rules

Rules are evaluated top-to-bottom; the first match wins.

| Rule | Condition | Action |
|---|---|---|
| 1 | Urgency = High | Escalate |
| 2 | Topic = Bug and no KB match or KB score < 0.15 | Escalate |
| 3 | Topic = Bug and KB score >= 0.15 | Auto-reply |
| 4 | Topic = Technical Issue and KB score < 0.15 | Escalate |
| 5 | Topic = Feature Request | Auto-reply |
| 6 | Urgency = Low or Medium and KB score >= 0.15 | Auto-reply |
| 7 | Fallback | Escalate |

---

## Follow-up Schedule

| Action | Topic | Hours | Note |
|---|---|---|---|
| Escalate | Billing | 24 h | Confirm refund or resolution |
| Escalate | Bug | 48 h | Report engineering fix status |
| Escalate | Technical Issue | 48 h | Human agent update |
| Escalate | Account | 24 h | Confirm account access restored |
| Auto-reply | Bug | 72 h | Confirm workaround resolved issue |
| Auto-reply | Technical Issue | 72 h | Confirm technical resolution |
| Auto-reply | Account | 72 h | Confirm resolution if no reply |
| Auto-reply | Billing | 48 h | Confirm billing acknowledgement |
| Any | Feature Request | — | No follow-up scheduled |

---

## Knowledge Base

The KB contains 11 articles indexed with TF-IDF (unigrams + bigrams). Articles are retrieved by cosine similarity; results with score below 0.05 are discarded.

| ID | Topic | Title |
|---|---|---|
| kb001 | Account | Password Reset |
| kb002 | Account | Update Account Email |
| kb003 | Billing | Duplicate or Incorrect Charges |
| kb004 | Billing | Cancel Subscription |
| kb005 | Bug | PDF Export Crash |
| kb006 | Technical Issue | 504 Gateway Timeout on API Calls |
| kb007 | Technical Issue | API Rate Limits |
| kb008 | Feature Request | Submitting Feature Requests |
| kb009 | Account | Two-Factor Authentication |
| kb010 | Billing | Subscription Plans and Upgrades |
| kb011 | Technical Issue | Intermittent API Integration Failures |

---

## Setup

All notebooks are designed to run in Google Colab.

### 1. Open in Colab

Upload the notebook to Google Drive and open it in Colab, or use File > Upload notebook directly in Colab.

### 2. Add your API key as a Colab secret

Click the key icon in the left sidebar (Secrets), then add:

| Notebook | Secret name |
|---|---|
| Anthropic | `ANTHROPIC_API_KEY` |
| OpenAI | `OPENAI_API_KEY` |
| Gemini | `GEMINI_API_KEY` |

Enable notebook access for the secret when prompted.

### 3. Run all cells

Use Runtime > Run all. The API key cell reads from Colab secrets automatically. If secrets are unavailable, paste your key directly into the commented line in that cell.

---

## Dependencies

Each notebook installs its own dependencies in the first cell. No manual installation is required.

| Package | Purpose |
|---|---|
| `langchain` | Prompt templates, output parsers, chain composition |
| `langchain-anthropic` / `langchain-openai` / `langchain-google-genai` | LLM provider integration |
| `langgraph` | Stateful multi-node pipeline orchestration |
| `scikit-learn` | TF-IDF vectoriser and cosine similarity for KB search |
| `pydantic` | Structured output schema for email classification |

---

## Requirements

- Python 3.10 or later (Colab default satisfies this)
- A valid API key for the chosen provider
