# Sentinel Triage — Project Outline
An AI-powered SOC analyst (Security Operations Center analyst) assistant that ingests security event logs, clusters related events, classifies threats, and produces analyst-ready incident summaries with MITRE ATT&CK technique mapping.


## Project Snapshot




Framing
Security / SOC analyst angle
Time commitment
2–4 hours per week (evenings)
Target ship date
8–10 weeks (thorough, with polish)
Repo name
sentinel-triage
Tech stack
Python, Anthropic API (Claude Sonnet), Docker (optional)



## Guiding Principles
Every stage ends with something you could screenshot and show someone. No stage exists only to set up the next one. This is the single biggest anti-procrastination pattern for portfolio projects: each week has a visible payoff.

Depth beats breadth. One meaningful integration, one rigorous eval, one well-written README will outperform three half-finished features every time.

The eval harness is non-negotiable. It's the sentence that separates this project from every other student LLM project. Do not skip Stage 5.


## The Overall Shape
Eight stages across nine weeks, with a buffer week baked in for real life (illness, midterms, commitments). Each stage produces something concrete and demo-able — if you have to pause after Stage 4 for two weeks, what you already have is still a coherent artifact.


### Stage 0 — Foundation
Week 1 · ~3 hours

Purpose: Set up the environment so you never have to fight tooling later. Boring but load-bearing.

Entry criteria: You have an Anthropic API key and Python installed.

Work:

Create the GitHub repo sentinel-triage (public, MIT license, real README stub — not the auto-generated one)
Set up virtual environment, .env file, .gitignore, and requirements.txt
Write a one-paragraph project vision at the top of the README — what the tool does, who it's for, why it matters. This becomes your north star for scope decisions.
Sketch a 3-sentence "non-goals" section: what this tool will not do (e.g., "not a production SIEM replacement," "does not detect novel zero-days"). Non-goals prevent scope creep more effectively than any project management technique.
Create a docs/ folder with an empty ARCHITECTURE.md and EVAL.md. Just placeholders — future you will thank present you.

Exit criteria: You can git clone your repo, activate the venv, and it's ready to code in. README exists with vision and non-goals.

Anti-procrastination note: This is the stage where perfectionism kills projects. Do not spend three hours picking a logo or agonizing over the README structure. The vision paragraph should be rough draft quality — you'll rewrite it in Stage 7 anyway.


### Stage 1 — Synthetic Log Generator
Week 2 · ~4 hours

Purpose: You need realistic-looking security logs to work against, and generating them yourself teaches you what security event categories actually look like. Educational work that produces reusable fixtures.

Entry criteria: Stage 0 complete.

Work:

Write generate_logs.py — produces synthetic auth logs, network logs, and application logs. Model them after common formats: Linux /var/log/auth.log, Windows Security Event logs, or nginx access logs. Pick one to start.
Include scenarios: normal baseline traffic, brute-force login attempts, credential stuffing, privilege escalation (sudo abuse), unusual off-hours access, and lateral movement patterns.
Each scenario is a function that emits a burst of realistic log lines with configurable intensity. Sprinkle them into benign traffic to create realistic "needle in haystack" fixtures.
Save at least three fixture files in fixtures/: benign.log (control), brute_force.log, and mixed_incident.log (multiple scenarios in one file).

Exit criteria: You can run python generate_logs.py --scenario brute_force --output fixtures/brute_force.log and get 5,000 lines of realistic auth logs with an embedded attack pattern.

Learning payoff: Writing the generator makes you internalize what these attacks look like in logs. That knowledge alone is interview gold — most students who claim security interest can't describe what a brute-force pattern looks like on the wire.


### Stage 2 — Core Parsing and Clustering Engine
Week 3 · ~4 hours

Purpose: The non-LLM half of the pipeline. Extract signals from noise before you spend API tokens.

Entry criteria: You have fixtures to work against from Stage 1.

Work:

Write parser.py — reads log lines, extracts structured fields (timestamp, source IP, user, action, status). Support one log format cleanly rather than trying to handle all of them.
Write clusterer.py — groups similar events using signature hashing (strip timestamps, IPs, session IDs; keep the event shape). Return the top N clusters with counts.
Add a --verbose flag that prints cluster breakdowns without any LLM call, so you can debug the classical layer independently.

Exit criteria: Running the tool against mixed_incident.log prints a ranked list of event clusters with counts, and the top clusters visibly correspond to the attacks you seeded. You've validated the pipeline before spending an API cent.

Why this matters: Any competent security engineer will look at your project and immediately ask "how do you avoid burning tokens on repetitive noise?" This stage is your answer.


### Stage 3 — Claude Integration and Prompt Engineering
Weeks 4–5 · ~7 hours

Purpose: The LLM half. This is the stage where you'll iterate the most — budget accordingly.

Entry criteria: Clusterer produces sensible output on your fixtures.

Work:

Write analyzer.py — takes clusters, formats them into a prompt, calls Claude, returns structured output.
Design a prompt that instructs Claude to output specific fields: threat classification, confidence score, MITRE ATT&CK technique IDs (T1110 for brute force, T1078 for valid accounts, etc.), suggested analyst next steps, and related events.
Use JSON mode or ask for structured output so downstream code can parse it reliably. Include a schema in the prompt.
Iterate on the prompt across at least 5 fixture runs. Save your best prompt versions in prompts/ with dated filenames so you can see your progression.
Add token usage tracking to every run — log input/output tokens and estimated cost.

Exit criteria: Running against brute_force.log produces a parseable JSON report identifying T1110 with reasonable confidence, sensible next steps, and cost < $0.02 per run.

Anti-procrastination note: This is where you'll be tempted to keep tweaking the prompt forever. Set a hard rule: after 5 iterations, whatever you have goes into the codebase and you move on. You can revisit in Stage 6.


### Stage 4 — CLI Wrapper and Output Formatting
Week 6 · ~4 hours

Purpose: Make it feel like a real tool, not a research script.

Entry criteria: Analyzer works end-to-end.

Work:

Write main.py (or sentinel.py) — the CLI entry point using argparse or click. Wire the phases together: parse → cluster → analyze → format.
Support two output modes: --format human (colored terminal output with sections) and --format json (machine-readable for downstream integration).
Add a --dry-run flag that runs everything except the Claude call and prints what would be sent. Invaluable for demos where you don't want to burn credits.
Add clean error handling: what happens when the log file is empty, malformed, or when the API call fails? Log gracefully rather than crashing.

Exit criteria: You can run sentinel fixtures/mixed_incident.log --format human and get a colored, well-structured incident report. Someone else could pick up your terminal and use the tool without asking you questions.


### Stage 5 — Evaluation Harness ⭐
Week 7 · ~4 hours

Purpose: This is the stage that separates your project from every other student LLM project. Do not skip it.

Entry criteria: CLI works reliably on your fixtures.

Work:

Create eval/scenarios.json — a list of 20 hand-labeled scenarios. Each has: fixture file, expected threat classification, expected MITRE technique ID, expected severity, and any notes.
Write eval/run_eval.py — iterates through scenarios, runs the tool on each, compares output to expected values, produces a score card.
Report on at least three metrics:
Classification accuracy — did it identify the right threat type?
Technique-mapping accuracy — did it hit the right MITRE ID?
False positive rate — did it flag benign fixtures as threats?
Save eval results as eval/results-YYYY-MM-DD.json so you have a version history of your tool's performance.
Document methodology in docs/EVAL.md: how scenarios were constructed, what the labels mean, known limitations.

Exit criteria: You can quote a real accuracy number, backed by a reproducible eval you documented. Something like "87% classification accuracy and 4% false-positive rate across 20 hand-labeled scenarios."

Career payoff: This is the sentence that lands on your resume and lights up in interviews. When you sit down with Greg or any security professional and they ask "how do you know it works?" — you have an answer, and it's a technically credible one.


### Stage 6 — One Meaningful Integration
Week 8 · ~4 hours

Purpose: Prove the tool can plug into a real workflow. Pick one — depth beats breadth.

Entry criteria: Eval harness passes at a level you're satisfied with.

Pick ONE:

Option A — Slack webhook (highest signal for security roles): Add --notify-slack WEBHOOK_URL that posts a formatted incident summary to a channel. Include a screenshot in your README of the alert firing in a test Slack workspace.

Option B — GitHub Actions (broadest DevOps appeal): Create .github/workflows/security-scan.yml that runs Sentinel on a sample log file in CI and posts findings as a workflow summary or PR comment.

Option C — SIEM-flavored structured output: Output in a security-standard format — CEF (Common Event Format) or STIX 2.1 bundles. More research required but strongly signals SOC domain knowledge.

Exit criteria: The integration works end-to-end and is demonstrable via a screenshot or short GIF in the README.

Recommendation: For security-role hiring, Option A first, Option C second. Option A is faster and produces a screenshot recruiters love. Option C is more technically impressive but takes longer.


### Stage 7 — Documentation and Polish
Week 9 · ~4 hours

Purpose: Turn a working codebase into a portfolio artifact. This stage is 80% of what determines whether recruiters actually engage with the repo.

Entry criteria: All prior stages complete.

Work:

Rewrite the README from scratch with these sections in this order:
Hook (one sentence + a screenshot of a real incident report)
Motivation (why this exists)
Architecture diagram
Quickstart (three commands)
Example run with real output
Eval methodology and results
Limitations
Future work
Add an ARCHITECTURE.md walking through design decisions: why cluster before LLM, why structured output, how you evaluate.
Include the architecture diagram.
Add a CHANGELOG.md documenting version history — signals engineering discipline.
Clean up code: consistent formatting (black or ruff), docstrings on public functions, type hints where they help.
Add a LICENSE file and update README badges.

Exit criteria: A recruiter with no context could land on your repo, read the README in 2 minutes, and understand what the tool does, why it's interesting, and how well it works.


### Stage 8 — Ship and Promote
Week 9–10 · ~2 hours

Purpose: The project doesn't exist for anyone but you until you tell people about it.

Entry criteria: Repo is polished.

Work:

Write the LinkedIn post: 3–4 sentences, one screenshot, link to repo. Post it once, don't overthink it. Tag Anthropic in a comment (not the main post — cleaner) since they engage with student builder content.
Add the project to the top of your GitHub profile README as a pinned repo.
Update your resume with the bullet drafted in earlier planning (Variant 3, adapted for the security framing).
Draft a short talking-points doc for yourself: 30-second version, 2-minute version, 5-minute deep dive. Interviews will ask about it — rehearsed versions remove cognitive load in the moment.
Send it to Greg with a short note: "Wanted to share a project I built exploring AI-assisted alert triage — would love your read on it from an industry perspective when you have a moment."

Exit criteria: The project is public, on your resume, on LinkedIn, and in your mentors' inboxes.


Buffer and Reality
Weeks 9–10 include a slack week on purpose. If you lose a week to midterms, PLA obligations, or life, you don't have to redo the plan — you just consume the buffer. If you don't lose a week, use the extra time in Stage 5 or Stage 7 (both benefit disproportionately from more attention).


The Rhythm to Actually Make This Work
A pattern that helps with 2–4 hour weeks and the tendency to procrastinate:

Start every session by reading the exit criteria for your current stage, then set a timer for 90 minutes.

No planning, no re-reading old code, no tweaking the README — just work toward the exit criteria until the timer goes. If you finish early, stop. If you don't finish, note where you stopped in a scratch file and pick up there next session.

Two 90-minute blocks per week gets you through this plan comfortably. Any more is a bonus.
