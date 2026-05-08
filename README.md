# pylinex Claude Code Plugin

This plugin gives Claude deep knowledge of the **pylinex** ecosystem for global 21cm signal extraction. Once installed, Claude can help you write pylinex/distpy/ares/perses code — generating extraction pipelines, configuring MCMC runs, building models, and more.

## What This Does

When you install this plugin, Claude automatically understands:
- The full pylinex API (basis, model, fitter, loglikelihood, nonlinear, expander, etc.)
- How distpy distributions and transforms work for priors and proposals
- How perses foreground/signal models integrate with pylinex
- How ares simulations generate training sets
- Common workflows from simple polynomial fits to full MCMC extraction

## Installation

### 1. Install Claude Code

Download and install from [claude.ai/code](https://claude.ai/code), then authenticate:

```bash
claude
```

Sign in via the browser prompt, then exit with `/exit`.

### 2. Install the Python libraries

Use the included `environment.yml` to create a conda environment with all dependencies and libraries in one command:

```bash
conda env create -f https://raw.githubusercontent.com/caden-kunkel/pylinex-claude-plugin/main/environment.yml
conda activate pylinex_env
```

> **Note:** After installing perses, you may see a message reminding you to set the `$PERSES` environment variable. Add `export PERSES=/path/to/your/perses` to your `.bashrc` or `.zshrc`.

### 3. Install this Claude Code plugin

Inside Claude Code, run these slash commands:

```
/plugin marketplace add caden-kunkel/pylinex-claude-plugin
/plugin install pylinex@caden-kunkel-pylinex-claude-plugin
```

Or install locally by cloning and running in your terminal:

```bash
git clone https://github.com/caden-kunkel/pylinex-claude-plugin.git
claude plugin install ./pylinex-claude-plugin
```

## Usage

Once installed, just ask Claude to help with 21cm signal extraction tasks:

- "Set up a foreground + signal extraction using a power law times log polynomial for the foreground and a trained basis from my signal training set"
- "Create an MCMC sampler to explore the posterior of my model parameters"
- "Show me how to use MetaFitter to optimize the number of basis vectors"
- "Help me flatten my multi-bin data and set up the right expanders"

Claude will generate correct, runnable pylinex code based on its understanding of the full library API.

## Requirements

- A Claude account (Pro, Max, or Team plan)
- [Conda](https://docs.conda.io/en/latest/miniconda.html) for managing the Python environment
- Optional: `pip install emcee` — required for MCMC ensemble sampling workflows
