# AI Evals for PMs

An interactive, hands-on course that teaches product managers how to evaluate AI systems — by actually doing it.

No slides. No videos. You open Claude Code, and it becomes your AI evals tutor: teaching concepts one at a time, guiding you through exercises with real datasets, and evaluating your product decisions.

## Who Is This For?

Product managers working on or transitioning into AI products. You don't need a machine learning background — just willingness to work with data and think critically about system quality.

## What You'll Learn

- **How to read an AI system** — understand LLM pipelines, trace production logs, and identify which stage is responsible when things go wrong
- **How to measure AI reliability** — pass@k, reliable@k, and the consistency gap that determines ship/hold decisions
- **How to map failure surfaces** — a structured framework for identifying every way your AI system can fail
- **How to connect evals to product decisions** — not just "what's the accuracy?" but "what should we do about it?"

## Quick Start

**Prerequisites:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and configured.

```bash
git clone https://github.com/sanjeevrao1992/ai-evals-for-pms.git
cd ai-evals-for-pms
claude
```

That's it. Claude will introduce itself as your tutor and guide you through the first lesson.

## Course Outline

The course is structured as a 7-day journey. Each day simulates a real PM scenario — you've just inherited an AI product and need to evaluate it.

| Day | Lesson | Topic | Duration |
|-----|--------|-------|----------|
| 1 | D1-L1 | What Does This System Actually Do? | ~25 min |

More lessons coming soon.

## How It Works

Each lesson follows a three-part structure:

1. **Concepts** — Industry-agnostic frameworks any PM can apply. Claude teaches one concept at a time, checking your understanding before moving on.
2. **Exercise** — You analyze a real (synthetic) dataset from an AI-powered menu verification system at a food delivery company. You direct the analysis; Claude does the computation. You interpret the results.
3. **PM Decision Point** — You write a memo, recommendation, or artifact using what you calculated. Claude evaluates your response.

Your progress is saved locally so you can pick up where you left off.

## What's in the Repo

```
lessons/        Lesson content (concepts + exercises + decision points)
exercises/      CSV datasets you'll analyze during exercises
tutor/          Session protocol and scoring rubrics (Claude's instructions)
progress/       Your local progress (gitignored)
```

## License

[CC BY-NC-SA 4.0](LICENSE) — Share and adapt for non-commercial purposes with attribution.
