# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A personal study-notes repository for the **AWS Certified Solutions Architect – Associate (SAA-C03)** exam. It is a fork of [ChathurangaVKD/AWS-Certified-Solutions-Architect-Associate-SAA-C03](https://github.com/ChathurangaVKD/AWS-Certified-Solutions-Architect-Associate-SAA-C03) (MIT licensed), personalized and extended by the repo owner. There is no application code — every file is Markdown study content, plus a static HTML/Jekyll site for browsing it.

**Git remotes:**
- `origin` → the owner's fork (`karthikeya7/AWS-SAA-C03-Study-Notes-And-Question-Dump`) — push changes here.
- `upstream` → the original author's repo — reference only, do not push here.

Work happens directly on `origin/main` (no PR workflow needed — it's the owner's own fork). After any content edit, **commit and push to `origin main`** unless told otherwise.

## Repository structure

Each numbered module folder (`01-AWS-Fundamentals/` through `13-Cost-Optimization/`, plus `14-Practice/`) follows the same file pattern:

| File | Purpose |
|------|---------|
| `README.md` | The main, detailed reference notes for the module — this is where deep-dive explanations, tables, and mnemonics get added |
| `FAST-LEARN.md` | A condensed 60–90 min cram version of the same module |
| `ULTRA-FAST-LEARN.md` | An even more condensed cram version |
| `PRACTICE-QUESTIONS.md` | Multiple-choice practice questions with collapsible answers |
| `DIAGRAMS.md` | Mermaid diagrams for the module |

Other top-level content:
- `exam-reviews/` — reviews of full practice exams taken, including `REINFORCEMENT-QUESTIONS-ALL-TESTS.md` and `STUDY-PROGRESS-TRACKER.md`.
- `docs/study-guides/` — cross-module guides (flashcards, quick reference, roadmap, ultra-fast learning index).
- `scripts/` — Python utilities for validating/fixing Mermaid diagram syntax across the repo (see below).
- `index.html` + `_config.yml` — a Jekyll-based interactive study site (GitHub Pages), with a client-side search box over module links. Internal links in `index.html`, `_config.yml`, and `README.md` must point at the `origin` fork, not `upstream`.

## Content conventions

- **Collapsible answers use native Obsidian foldable callouts**, not raw `<details>` HTML — the latter renders unreliably in Obsidian (nested Markdown/code fences inside `<details>` often fail to render). Pattern:
  ```markdown
  > [!success]- Show Answer
  > **Answer: B**
  >
  > **Explanation:**
  > - point one
  ```
  Every line of the answer body is prefixed with `>`, including blank lines (as bare `>`).
- **Explanations should teach the "why," not just state the "what."** House style favors real-world analogies (car rental for EC2 pricing, restaurant manager for Auto Scaling Groups, office badge access for Lambda+VPC) over dry definitions — these are deliberately added to aid retention, not filler.
- **Mnemonics are used liberally** for anything with an enumerable list (e.g. EC2 instance family letters C-R-M-T-I-D-H → "Crazy Rabbits Munch Tiny Insects During Hunts"). When adding new list-heavy content, consider adding one.
- **Tables commonly include a "Plain English" or analogy column** alongside the technical description — this repo is optimized for a learner who wants intuition, not just facts.
- **Practice question answers must be internally consistent.** When an explanation's own reasoning contradicts the stated answer letter, or when two options are both technically defensible (e.g. an identity-based vs. resource-based policy achieving the same result), treat that as a bug to fix — correct the answer key or rewrite the distractor so exactly one option is correct, don't just add a caveat.
- **Cross-reference other modules before adding new content**, to avoid duplicating coverage (e.g. EBS lives in `04-Storage`, Security Groups in `06-Networking`, Dedicated Hosts in `13-Cost-Optimization` — don't re-explain these in `03-Compute`).

## Diagram validation scripts

```bash
# Validate all Mermaid diagrams for syntax issues (exit 0 = pass, 1 = errors)
python3 scripts/validate_mermaid.py

# Auto-fix common Mermaid syntax issues (review with git diff before committing)
python3 scripts/fix_mermaid.py
```

No external Python dependencies required (standard library only). Run validate → fix → validate again when editing any `DIAGRAMS.md` file.

## License and attribution

MIT licensed (inherited from upstream). The `README.md` attribution note crediting the original author must be preserved — do not remove it when editing the README.
