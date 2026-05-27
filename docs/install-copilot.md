# Using pylinex with GitHub Copilot

```bash
conda env create -f environment.yml
conda activate pylinex_env
```

Copy the instructions file into your project repo:

```bash
cp /path/to/pylinex-ai-assistant/.github/copilot-instructions.md \
   /path/to/your-project/.github/copilot-instructions.md
```

Copilot picks it up automatically — no other config needed. Commit it so teammates get it too.
