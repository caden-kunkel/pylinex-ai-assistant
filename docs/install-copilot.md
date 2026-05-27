# Using pylinex with GitHub Copilot

GitHub Copilot reads a file called `.github/copilot-instructions.md` from any repo you're working in and uses it as background context for all suggestions. This repo includes that file pre-filled with the full pylinex API reference — copy it into your project once and Copilot will have pylinex knowledge whenever you're working in that repo.

---

## 1. Install the Python libraries

This creates a conda environment called `pylinex_env` with pylinex, distpy, perses, ares, and all their dependencies installed from source:

```bash
conda env create -f https://raw.githubusercontent.com/caden-kunkel/pylinex-ai-assistant/main/environment.yml
conda activate pylinex_env
```

## 2. Add the instructions file to your project

Clone this repo, then copy the Copilot instructions file into your working repository:

```bash
git clone https://github.com/caden-kunkel/pylinex-ai-assistant.git
cp pylinex-ai-assistant/.github/copilot-instructions.md /path/to/your-project/.github/copilot-instructions.md
```

Copilot picks it up automatically — no other configuration needed. Commit it to your repo so teammates get it too when they clone.
