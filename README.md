# The Eval Bootcamp

A hands-on AI evals course for product managers — taught inside Claude Code, not a classroom.

No slides. No videos. You clone this repo, open Claude Code, and it becomes your AI evals tutor: teaching concepts one at a time, guiding you through exercises with real datasets, and evaluating your product decisions against a scoring rubric.

## Who Is This For?

Product managers working on or transitioning into AI products. You don't need a machine learning background — just willingness to work with data and think critically about system quality.

## What You'll Learn

- **How to read an AI system** — understand LLM pipelines, trace production logs, and identify which stage is responsible when things go wrong
- **How to measure AI reliability** — pass@k, reliable@k, and the consistency gap that determines real ship/hold decisions
- **How to map and triage failure surfaces** — a structured framework for finding every way your AI system can break
- **How to build automated quality checks** — code-based graders, LLM-as-judge, and layering strategies for production
- **How to build ground truth you can trust** — golden datasets, contamination detection, and dataset lifecycle management
- **How to connect evals to product decisions** — not just "what's the accuracy?" but "what should we do about it?"

## Quick Start

**Prerequisites:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and configured.

```bash
git clone https://github.com/sanjeevrao1992/the-eval-bootcamp.git
cd the-eval-bootcamp
claude
```

That's it. Claude reads `CLAUDE.md` on startup and introduces itself as your tutor. It will check your progress and guide you from wherever you left off.

## Course Structure

21 days, one lesson per day. Each lesson is ~25–40 min and follows the same structure:

1. **Concepts** — Industry-agnostic frameworks any PM can apply. Taught one concept at a time with concrete examples.
2. **Exercise** — You analyze a real (synthetic) dataset from an AI-powered menu verification system at a food delivery company. You direct the analysis; Claude does the computation; you interpret the results.
3. **PM Decision Point** — You write a recommendation using what you calculated. Claude evaluates your response against a scoring rubric.

Your progress is saved locally (`progress/progress.json`, gitignored) so you pick up where you left off.

### Week 1 — You Just Inherited an AI Product

| Day | Lesson | Key Skills |
|-----|--------|------------|
| D1 | What Does This System Actually Do? | Pipeline stages, non-determinism, reading traces |
| D2 | Mapping Every Way Your AI System Can Fail | Evaluation surface map, failure layers, coverage gaps |
| D3 | Error Analysis | Open coding, axial coding, triage framework |

### Week 2 — Building Your Evaluation Toolkit

| Day | Lesson | Key Skills |
|-----|--------|------------|
| D4 | Thinking in Distributions | Shape before depth, pass@k, reliable@k, the gap |
| D5 | The Three Types of Quality Checks | Code-based, model-based, human graders; layering strategy |
| D6 | LLM-as-Judge | Calibration trap, Critique Shadowing, failure modes, meta-evaluation |

### Week 3 — Measuring What Matters

| Day | Lesson | Key Skills |
|-----|--------|------------|
| D7 | Ground Truth and Golden Datasets | Three sources of ground truth, contamination, dataset lifecycle |
| D8–D9 | *(coming soon)* | RAG evaluation, hallucination detection |

### Weeks 4–7 — *(coming soon)*
Metric design, release criteria, eval-driven development, observability, red teaming, and more.

## What's in the Repo

```
lessons/        Lesson content — concepts, exercises, decision points (D1.md through D7.md)
exercises/      CSV datasets you'll analyze during exercises
tutor/          Session protocol and scoring rubrics (Claude's tutor instructions)
progress/       Your local progress — gitignored, never leaves your machine
CLAUDE.md       Course configuration — Claude reads this on startup
```

## License

[CC BY-NC-SA 4.0](LICENSE) — Free to use and adapt for non-commercial purposes with attribution.
