# pylinex AI Assistant

Get accurate, runnable code for **global 21cm signal extraction** with any AI tool — Claude Code, GitHub Copilot, ChatGPT, Gemini, or any chat interface.

The core asset is a 944-line API reference covering the full pylinex ecosystem: **pylinex**, **distpy**, **perses**, and **ares**. Each integration method below loads that reference into your AI of choice.

---

## Choose your AI tool

| Tool | Instructions |
|------|-------------|
| Claude Code | [docs/install-claude-code.md](docs/install-claude-code.md) |
| GitHub Copilot | [docs/install-copilot.md](docs/install-copilot.md) |
| ChatGPT | [docs/install-chatgpt.md](docs/install-chatgpt.md) |
| Gemini / any other chat AI | [docs/install-other.md](docs/install-other.md) |

Every guide includes both the Python library install and the AI-specific setup — start to finish.

---

## What the libraries do

- **pylinex** — basis representations, models, fitters, likelihoods, MCMC
- **distpy** — probability distributions, transforms, MCMC proposals
- **perses** — foreground, beam, and instrument models
- **ares** — physics-based 21cm signal generation

---

## Usage examples

Once set up, describe what you want in plain English:

> "Fit my spectrum using a 6-term PolynomialBasis with the Fitter class."

> "Run MetaFitter over 3–10 terms using BPIC to select the best fit automatically."

> "Use BasisSum to fit foreground and signal jointly, then extract the signal component."

> "Build an Extractor with a TrainedBasis foreground and a MetaFitter for the signal."

---

## Repository structure

```
pylinex-claude-plugin/
├── skills/pylinex/SKILL.md         — 944-line API reference (source of truth)
├── .github/copilot-instructions.md — same API reference, for GitHub Copilot
├── starter_prompt.md               — preamble + API reference, paste into any chat
├── docs/
│   ├── install-claude-code.md
│   ├── install-copilot.md
│   ├── install-chatgpt.md
│   └── install-other.md
├── environment.yml                 — one-command conda install for all four libraries
└── README.md
```

---

## Requirements

- [Conda](https://docs.conda.io/en/latest/miniconda.html) for managing the Python environment
- An account on your AI tool of choice
- Optional: `pip install emcee` — required for MCMC ensemble sampling workflows
