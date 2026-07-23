# Sentinel Triage

**An AI-powered SOC analyst assistant that clusters security event logs, classifies threats, and generates analyst-ready incident summaries with MITRE ATT&CK technique mapping.**

![Status](https://img.shields.io/badge/status-in%20development-yellow)
![Python](https://img.shields.io/badge/python-3.11+-blue)
![License](https://img.shields.io/badge/license-MIT-green)

> ⚠️ **Work in progress.** This project is being built in public as a portfolio artifact. See the [Roadmap Tracker](#roadmap) for current status.

## Vision

Sentinel Triage is a command-line tool that helps security operations analysts cut through log noise during incident response. It ingests raw event logs (auth logs, network logs, application logs), clusters similar events using signature-based hashing, and uses Claude (Anthropic's LLM) to produce structured triage reports — threat classification, MITRE ATT&CK technique mapping, severity ratings, and suggested analyst next steps.

The goal is a tool a tier-1 SOC analyst could actually use to accelerate the "first five minutes" of an alert investigation.

## Motivation

Security operations centers face a well-documented triage problem: analysts are buried in alerts, most of which are false positives or duplicates of the same underlying issue. Mean time to detect (MTTD) and mean time to respond (MTTR) both suffer, and analyst burnout is a serious industry-wide concern.

Sentinel Triage explores a hypothesis: an LLM, prompted with pre-clustered event signatures and a well-scoped output schema, can produce triage summaries that meaningfully reduce the cognitive load on tier-1 analysts — without hallucinating threats where none exist. The project measures this hypothesis with a hand-labeled evaluation harness rather than relying on demo-day vibes.

## Non-Goals

To keep scope honest, Sentinel Triage explicitly is **not**:

- A production SIEM replacement (no real-time streaming, no long-term storage, no user management)
- A zero-day detection system (it identifies known attack patterns, not novel exploits)
- An autonomous response tool (it triages and recommends; humans decide)

## Planned Architecture

Raw logs -> parser -> Signature Clusterer -> Claude API -> Structured report ( -> Slack / GitHub / stdout)

The clustering layer exists specifically to reduce LLM token usage on repetitive event streams - a design decision driven by both cost and signal-to-noise concerns. Full architecture notes live in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) (coming in Stage 7).

## Roadmap

Built in eight stages over roughly nine weeks. See [docs/PLAN.md](docs/PLAN.md) for the full plan.

- [x] **Stage 0** — Foundation (repo, README, docs scaffolding)
- [ ] **Stage 1** — Synthetic log generator
- [ ] **Stage 2** — Parser + signature clusterer
- [ ] **Stage 3** — Claude integration and prompt engineering
- [ ] **Stage 4** — CLI wrapper and output formatting
- [ ] **Stage 5** — Evaluation harness (hand-labeled scenarios)
- [ ] **Stage 6** — Integration (Slack webhook or SIEM output)
- [ ] **Stage 7** — Documentation and polish
- [ ] **Stage 8** — Ship and promote

## Tech Stack

- **Language:** Python 3.11+
- **LLM:** Anthropic Claude Sonnet
- **Testing:** pytest (planned)
- **Formatting:** ruff, black

## Licence

MIT - see [LICENCE](LICENCE)


## About

Built by [Domenic (Dom) Jernigan](https://github.com/Domenicj1) as a portfolio project exploring the intersection of AI and security operations. Feedback welcome via GitHub issues.



