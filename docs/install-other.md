# Using pylinex with any AI (Gemini, Claude.ai, Perplexity, etc.)

This method works with any chat-based AI. The `starter_prompt.md` file in this repo contains the full pylinex API reference along with a short instruction preamble. Paste it as your first message and the AI will have pylinex knowledge for the rest of that session.

---

## 1. Install the Python libraries

This creates a conda environment called `pylinex_env` with pylinex, distpy, perses, ares, and all their dependencies installed from source:

```bash
conda env create -f https://raw.githubusercontent.com/caden-kunkel/pylinex-ai-assistant/main/environment.yml
conda activate pylinex_env
```

## 2. Load the API reference

Download [`starter_prompt.md`](../starter_prompt.md) from this repo and paste the entire contents as your first message in a new chat. Most modern AI interfaces handle the full length without truncation.

> **Note:** The context only lasts for that session. Start a new chat and paste again when you need a fresh session.
