# Using pylinex with ChatGPT

## Option A — Custom GPT (easiest, no setup)

A dedicated pylinex Custom GPT is available with the full API reference pre-loaded:

**[pylinex assistant — ChatGPT Custom GPT]**
> URL coming soon — check back after initial release.

Just click the link, open a chat, and start asking. No installation required beyond the Python libraries below.

---

## Option B — Paste starter prompt into any chat

1. Open the file [`starter_prompt.md`](../starter_prompt.md) in this repo.
2. Copy everything from the line that says `---` downward (preamble + full API reference).
3. Paste it as your first message in any ChatGPT chat (free or Plus).
4. ChatGPT will use the API reference for the rest of the session.

This works in any ChatGPT interface and in any model (GPT-4o, o1, etc.).

---

## Step 2 — Install the Python libraries

Whichever option you chose above, you'll need the libraries installed locally to run the code ChatGPT writes for you:

```bash
conda env create -f environment.yml
conda activate pylinex_env
```

This installs pylinex, distpy, perses, ares, and all dependencies in one step.

---

## Usage examples

**Fit a spectrum with a polynomial basis:**
> "Fit my spectrum using a 5-term PolynomialBasis with the Fitter class. My frequencies array is in MHz."

**Automatic term selection:**
> "Run a MetaFitter over 3–10 terms using BPIC to find the best fit."

**Multi-component fit:**
> "Use BasisSum to fit foreground + signal jointly. The foreground has 6 terms, the signal model comes from ares."

**Full extraction pipeline:**
> "Walk me through setting up an Extractor with a TrainedBasis foreground and a MetaFitter for the signal."
