# pylinex Claude Code Plugin

This plugin is designed for use with [Claude Code](https://claude.ai/code). Once installed, Claude has deep knowledge of the **pylinex** ecosystem for global 21cm signal extraction — generating extraction pipelines, configuring MCMC runs, building models, and more.

## What This Does

When you install this plugin, Claude automatically understands:
- The full pylinex API (basis, model, fitter, loglikelihood, nonlinear, expander, etc.)
- How distpy distributions and transforms work for priors and proposals
- How perses foreground/signal models integrate with pylinex
- How ares simulations generate training sets
- Common workflows from simple polynomial fits to full MCMC extraction

## Installation

### 1. Install the Python libraries

Create a dedicated conda environment with all dependencies and libraries in one command:

```bash
conda env create -f https://raw.githubusercontent.com/caden-kunkel/pylinex-claude-plugin/main/environment.yml
conda activate pylinex_env
```

> **Note:** After installing perses, you may see a message reminding you to set the `$PERSES` environment variable. Add `export PERSES=/path/to/your/perses` to your `.bashrc` or `.zshrc`.

> **Already have a Python environment?** You can install the four libraries directly instead: `pip install git+https://github.com/caden-kunkel/distpy.git git+https://github.com/caden-kunkel/ares.git git+https://github.com/caden-kunkel/perses.git git+https://github.com/caden-kunkel/pylinex.git`

### 2. Install this Claude Code plugin

```bash
claude plugin install pylinex
```

## Usage

Start a new Claude Code session and ask Claude to help with 21cm signal extraction tasks:

- "Set up a foreground + signal extraction using a power law times log polynomial for the foreground and a trained basis from my signal training set"
- "Create an MCMC sampler to explore the posterior of my model parameters"
- "Show me how to use MetaFitter to optimize the number of basis vectors"
- "Help me flatten my multi-bin data and set up the right expanders"

## Requirements

- A Claude account (Pro, Max, or Team plan)
- Conda (recommended) or an existing Python 3.8+ environment with numpy, scipy, matplotlib, h5py, healpy, sympy, pandas, scikit-learn, and numdifftools
- Optional: `pip install emcee` — required for MCMC ensemble sampling workflows
