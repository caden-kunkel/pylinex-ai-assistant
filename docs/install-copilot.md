# Using pylinex with GitHub Copilot

## 1. Install the Python libraries

```bash
conda env create -f https://raw.githubusercontent.com/caden-kunkel/pylinex-ai-assistant/main/environment.yml
conda activate pylinex_env
```

## 2. Add Copilot instructions to your project

Clone this repo, then copy the instructions file into your working repository:

```bash
git clone https://github.com/caden-kunkel/pylinex-ai-assistant.git
cp pylinex-ai-assistant/.github/copilot-instructions.md /path/to/your-project/.github/copilot-instructions.md
```

Copilot reads `.github/copilot-instructions.md` automatically — no other config needed. Commit it so teammates get it too.
