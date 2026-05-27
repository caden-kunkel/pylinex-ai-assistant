# Using pylinex with Claude Code

Claude Code is Anthropic's AI coding assistant that runs in your terminal or desktop app. This plugin teaches it the full pylinex API so it can write correct extraction code without you having to explain the library.

---

## 1. Install the Python libraries

This creates a conda environment called `pylinex_env` with pylinex, distpy, perses, ares, and all their dependencies installed from source:

```bash
conda env create -f https://raw.githubusercontent.com/caden-kunkel/pylinex-ai-assistant/main/environment.yml
conda activate pylinex_env
```

## 2. Install the plugin

Open Claude Code and run this slash command to install the plugin from GitHub:

```
/plugin marketplace add caden-kunkel/pylinex-ai-assistant
```

## 3. Activate in a session

The plugin registers a skill called `pylinex`. At the start of any Claude Code session, say:

> "Use the pylinex skill"

Claude will load the full API reference for all four libraries and use it for the rest of the session.
