# Using pylinex with Claude Code

## 1. Install the Python libraries

This installs pylinex, distpy, perses, ares, and all dependencies into a new conda environment:

```bash
conda env create -f https://raw.githubusercontent.com/caden-kunkel/pylinex-ai-assistant/main/environment.yml
conda activate pylinex_env
```

## 2. Install the plugin

Inside Claude Code:

```
/plugin marketplace add caden-kunkel/pylinex-ai-assistant
```

## 3. Activate

In any session, say: *"use the pylinex skill"* — Claude will load the full API reference for pylinex, distpy, perses, and ares.
