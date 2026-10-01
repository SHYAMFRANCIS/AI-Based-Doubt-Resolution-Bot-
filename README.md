# AI-Based Doubt Resolution Bot

![Python](https://img.shields.io/badge/Python-3.x-blue) ![Gemini](https://img.shields.io/badge/LLM-Gemini_1.5_Flash-purple) ![License](https://img.shields.io/badge/License-MIT-green) ![Status](https://img.shields.io/badge/Status-CLI_working,_web_backend_missing-orange)

An academic doubt-resolution assistant powered by Google Gemini (`gemini-1.5-flash`). The working implementation is a **command-line chatbot** (`main.py`) that keeps a local JSON conversation history and answers questions with a Gemini-API-key-help system prompt. Two HTML fragments (`login.html`, `doubt.html`) provide a student login page and a doubt-submission form, but **no backend implementing their `/login` / `/submit_doubt` endpoints exists in this repo** (see [Known gaps](#known-gaps)).

> **Note on repo description:** it mentions FastAPI, FAISS, and SentenceTransformers. None of those appear in the committed code — `main.py` uses only `google-generativeai` plus the standard library. They are treated below as roadmap, not implemented features.

## Features

- CLI Q&A loop over Gemini 1.5 Flash with a focused system prompt (Gemini API-key setup, usage, troubleshooting, best practices)
- Last-5-exchanges context window injected into each prompt
- Persistent history in `conversation_history.json` (timestamped user/bot pairs), loaded at startup and saved after every turn
- Structured logging (`logging` module) for configuration, history, and generation errors
- Graceful degradation: friendly fallback message on generation failure; clean `exit` handling
- Web UI fragments: styled student login form and doubt-submission form wired to `POST /login` and `POST /submit_doubt` (backend not included)

## Stack

| Layer | Technology |
|---|---|
| LLM | Google Gemini 1.5 Flash via `google-generativeai` |
| Language | Python 3.x (`os`, `json`, `logging`, `datetime`) |
| Frontend fragments | Plain HTML + vanilla JS `fetch` (`doubt.html`, `login.html`) |
| Persistence | Local `conversation_history.json` (created at runtime, git-ignored) |
| Ignored at runtime | `__pycache__/`, `*.pyc`, `conversation_history.json` patterns belong in `.gitignore` |

## Repository structure

```text
AI-Based-Doubt-Resolution-Bot-/
├── main.py          # CLI bot: Gemini config, history load/save, prompt builder, REPL loop
├── doubt.html       # HTML fragment: greeting + doubt form → POST /submit_doubt (expects {response})
├── login.html       # Standalone page: student login form → POST /login; RIT branding header
├── .gitignore       # Python / packaging / test ignores
├── LICENSE          # MIT (© 2025 SHYAMFRANCIS)
└── conversation_history.json  # Created at runtime (not committed)
```

## Installation

**Prerequisites:** Python 3.8+, a Gemini API key ([Google AI Studio](https://aistudio.google.com/)).

```bash
git clone https://github.com/SHYAMFRANCIS/AI-Based-Doubt-Resolution-Bot-.git
cd AI-Based-Doubt-Resolution-Bot-
pip install google-generativeai
export GEMINI_API_KEY="your-key-here"   # PowerShell: $env:GEMINI_API_KEY="your-key-here"
```

> ⚠️ **Security:** the committed `main.py` currently hardcodes an API key as a module-level `api_key` string. Rotate that key, delete it from the source, and read it from the environment instead:
> ```python
> GEMINI_API_KEY = os.environ.get("GEMINI_API_KEY")
> ```

## Usage

```bash
python main.py
```

```text
Welcome to the Gemini API Key Doubt Resolution Bot!
Ask your questions about Gemini API key setup, usage, or troubleshooting.
Type 'exit' to quit.

You: How do I create a Gemini API key?
Bot: ...
You: exit
Goodbye!
```

- Empty input is rejected with a prompt to enter a valid question.
- Each exchange is appended to `conversation_history.json` with an ISO timestamp.

## Examples

| You ask | Bot context |
|---|---|
| "Where do I find my Gemini API key?" | System prompt + last 5 turns → concise, beginner-friendly setup steps |
| "Why do I get a 400/403 from the API?" | Troubleshooting guidance grounded in conversation history |
| `exit` | Saves history and quits |

**Web fragments (frontend only):**

```html
<!-- doubt.html posts urlencoded doubt=... and renders JSON field `response` -->
fetch('/submit_doubt', { method: 'POST',
  headers: {'Content-Type': 'application/x-www-form-urlencoded'},
  body: new URLSearchParams({ doubt }) })
```

```html
<!-- login.html -->
<form action="/login" method="POST"> ... username / email / password ... </form>
```

## Configuration

| Setting | Location | Default |
|---|---|---|
| Model | `initialize_model()` in `main.py` | `gemini-1.5-flash` |
| History file | `HISTORY_FILE` | `conversation_history.json` |
| Context window | `generate_response()` | last 5 exchanges |
| System prompt | `generate_response()` | Gemini-API-key specialist, beginner-friendly |
| Log level | `logging.basicConfig` | `INFO` |

## Known gaps

- `doubt.html` / `login.html` reference `POST /submit_doubt` and `POST /login`, but no Flask/FastAPI server implements them — `main.py` is CLI-only (`input()` loop).
- No `requirements.txt`; install is a single `pip install google-generativeai`.
- No FAISS index, SentenceTransformers embeddings, voice input, or multilingual support in code despite the repo description — candidates for future work.

## Contributing

1. Fork and create a feature branch.
2. Move the API key to `GEMINI_API_KEY` env (do not commit keys).
3. Add the missing web backend or `requirements.txt` if in scope.
4. Open a pull request describing the change and how it was tested (`python main.py` smoke test).

## License

MIT — see [LICENSE](LICENSE). © 2025 SHYAMFRANCIS.
