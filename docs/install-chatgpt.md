# Using pylinex with ChatGPT

There are two ways to give ChatGPT pylinex knowledge: a dedicated Custom GPT with the API reference permanently loaded, or a starter prompt you paste at the beginning of any chat.

---

## 1. Install the Python libraries

This creates a conda environment called `pylinex_env` with pylinex, distpy, perses, ares, and all their dependencies installed from source:

```bash
conda env create -f https://raw.githubusercontent.com/caden-kunkel/pylinex-ai-assistant/main/environment.yml
conda activate pylinex_env
```

## 2. Load the API reference into ChatGPT

**Option A — Custom GPT (recommended):** [pylinex-ai-assistant](https://chatgpt.com/g/g-6a17420264308191896c4dfcd8193fc7-pylinex-ai-assistant) — the full API reference is pre-loaded as a knowledge file. Just open the link and start chatting, no setup required.

**Option B — Paste prompt:** Download [`starter_prompt.md`](../starter_prompt.md) from this repo and paste it as your first message in any ChatGPT chat. ChatGPT will use the API reference for the rest of that session. Works with any model (GPT-4o, o1, etc.) and requires no account setup beyond a free ChatGPT login.
