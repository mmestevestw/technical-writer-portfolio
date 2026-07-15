# Case Study: Migrating to a Docs-as-Code Workflow

> **Fill this in with your own details.** This is a structural template — the headings and prompts below are the kind of thing interviewers and hiring managers look for in a case study. Replace the bracketed guidance with your real experience. Don't include your employer's product names, internal tool names, or proprietary architecture — describe the *process*, tools, and outcomes in general terms (e.g., "a Kubernetes-based microservice platform" rather than the actual product name).

## Context

*[1–2 sentences: what kind of product/team was this for, and what was the documentation situation before the change? e.g., "Documentation lived in a wiki, updated inconsistently, often out of sync with releases."]*

## The Problem

*[What was actually broken? Pick 2–3 concrete pain points — e.g., docs drifting from the shipped API, no review process for doc changes, inconsistent terminology across writers, releases going out without doc updates.]*

## What I Did

Describe your actual approach. Suggested structure:

1. **Moved source of truth to Git** — docs authored in Markdown/AsciiDoc, versioned alongside (or parallel to) the codebase, so doc changes could go through the same PR/review process as code.
2. **Automated style enforcement with Vale** — configured a style guide as linting rules, catching terminology inconsistencies, passive voice, and banned words automatically in CI, rather than relying on manual review for style issues.
3. **Generated reference docs from source specs** — for API documentation, scripted generation from OpenAPI/Swagger specs (in Python) so the reference material updated automatically when the spec changed, instead of manually transcribing endpoints.
4. **Set up CI-driven publishing** — doc builds and deployments triggered automatically on merge, removing manual publishing steps.

## Outcome / Impact

*[Be specific and quantify where you can — e.g., "reduced doc review turnaround from X days to Y," "cut terminology inconsistencies flagged in reviews by X%," "onboarding new writers to the style guide dropped from a full day to under an hour."]*

## What I'd Do Differently

*[Optional but valuable — shows reflection. E.g., "I'd introduce the linting rules incrementally rather than all at once, since the initial pass surfaced a large backlog of existing violations that slowed early adoption."]*

---

**Why this format:** Interviewers evaluating a Senior Technical Writer role typically want to see judgment and process, not just tools — this template is structured to surface *why* you made each decision, not just *what* you implemented.
