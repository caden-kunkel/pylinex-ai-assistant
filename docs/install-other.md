# Using pylinex with any AI (Gemini, Claude.ai, Perplexity, etc.)

This method works with any chat-based AI that accepts long messages — Gemini, Claude.ai, Perplexity, Mistral, or anything else.

## Step 1 — Install the Python libraries

```bash
conda env create -f environment.yml
conda activate pylinex_env
```

This installs pylinex, distpy, perses, ares, and all dependencies in one step.

## Step 2 — Paste the starter prompt

1. Open [`starter_prompt.md`](../starter_prompt.md) in this repo.
2. Copy everything from the line that says `---` downward (preamble + full API reference).
3. Paste it as your **first message** in a new chat with your AI of choice.
4. The AI will use the API reference for the rest of that session.

> **Note:** The starter prompt is ~1,000 lines. Most modern AI chat interfaces handle this without truncation. If your AI has a short context window, consider using the [ChatGPT Custom GPT](install-chatgpt.md) option instead, which keeps the reference in a knowledge file.

## Step 3 — Start asking

After pasting the starter prompt, just describe what you want:

**Fit a spectrum:**
> "Fit my spectrum using a 5-term PolynomialBasis with the Fitter class."

**Automatic term selection:**
> "Run a MetaFitter over 3–10 terms using BPIC."

**Multi-component fit:**
> "Use BasisSum to fit foreground + signal jointly. Foreground is 6 terms, signal comes from ares."

**Full extraction pipeline:**
> "Set up an Extractor with a TrainedBasis foreground and a MetaFitter for the signal."

## Tips

- Start a fresh chat when switching to a different project — the starter prompt should be the first message in each session.
- If the AI seems to forget the API details mid-conversation, paste the starter prompt again in a new chat.
- For longer working sessions, Claude Code or the ChatGPT Custom GPT (which keep the reference persistent) will give you a better experience.
