# Using pylinex with GitHub Copilot

## Step 1 — Install the Python libraries

```bash
conda env create -f environment.yml
conda activate pylinex_env
```

This installs pylinex, distpy, perses, ares, and all dependencies in one step.

## Step 2 — Add Copilot instructions to your project

Copy the Copilot instructions file into your working repository:

```bash
cp /path/to/pylinex-AI-Assistant/.github/copilot-instructions.md  \
   /path/to/your-project/.github/copilot-instructions.md
```

GitHub Copilot reads `.github/copilot-instructions.md` automatically for any repository that contains it. No other configuration is needed — the full pylinex API reference will be available to Copilot in every file you open in that repo.

> **Tip:** Commit `.github/copilot-instructions.md` to your project repo so teammates get it automatically when they clone.

## Step 3 — Start coding

Open any Python file in your project and ask Copilot inline. Copilot will use the API reference to suggest correct pylinex code.

Example prompts (inline or in Copilot Chat):

- "Fit this spectrum with a 6-term PolynomialBasis using the Fitter class"
- "Run MetaFitter over 3–10 terms using BPIC"
- "Build a BasisSum with a foreground basis and a signal basis, then extract the signal"
- "Create a TrainedBasis from my training set and pair it with a RepeatExpander"

## Usage examples

**Fit a single spectrum:**
```python
# Ask Copilot: "Fit my spectrum using PolynomialBasis and Fitter"
```

**Automatic term selection:**
```python
# Ask Copilot: "Use MetaFitter to pick the best number of terms by BPIC"
```

**Full extraction pipeline:**
```python
# Ask Copilot: "Set up an Extractor with a trained foreground basis"
```
