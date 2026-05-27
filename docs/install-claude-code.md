# Using pylinex with Claude Code

## Step 1 — Install the Python libraries

```bash
conda env create -f environment.yml
conda activate pylinex_env
```

This installs pylinex, distpy, perses, ares, and all dependencies in one step.

## Step 2 — Install the Claude Code plugin

Clone this repo (if you haven't already):

```bash
git clone https://github.com/caden-kunkel/pylinex-assistant.git
```

Then install the plugin from the local clone:

```bash
claude plugin install /path/to/pylinex-assistant
```

Or install directly from GitHub via the plugin marketplace inside Claude Code:

```
/plugin marketplace add caden-kunkel/pylinex-assistant
```

## Step 3 — Activate in a session

The plugin registers a skill called `pylinex`. To load it, just say:

> "Use the pylinex skill"

or

> "Load the pylinex plugin"

Claude will load the full 944-line API reference for pylinex, distpy, perses, and ares into context for that session.

## Usage examples

**Fit a spectrum with a polynomial basis:**
> "Fit my spectrum using a 5-term PolynomialBasis with the Fitter class. My frequencies array is in MHz."

**Automatic term selection:**
> "Run a MetaFitter over 3–10 terms using BPIC to find the best fit."

**Multi-component fit:**
> "Use BasisSum to fit foreground + signal jointly. The foreground has 6 terms, the signal model comes from ares."

**Full extraction pipeline:**
> "Walk me through setting up an Extractor with a TrainedBasis foreground and a MetaFitter for the signal."
