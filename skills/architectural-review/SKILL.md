---
name: architectural-review
description: User-invoked workflow that orchestrates a full architectural
  review. The main agent divides the repository into slices, gets user
  approval, launches one architecture-reviewer-sol subagent per slice in
  parallel, merges the findings, and records them as a Linear project with
  tickets. Use only when the user explicitly invokes `$architectural-review`;
  never select this skill implicitly.
---

# Architectural Review

Coordinate the review; do not perform it yourself. The `architecture-reviewer` subagent
carries the rubric (`engineering-principles`) and its own output format. Your jobs are: choose
good slices, write good slice prompts, merge the results, and record them in Linear.

## 1. Survey and slice

Read the repository's own map first: `README.md`, `AGENTS.md` / `CLAUDE.md`, any context map or
ADRs, and the workspace/package layout. Then divide the source into review slices.

A slice is a coherent unit one reviewer can hold in full:

- A small package is one slice. That is fine and common.
- A large package gets multiple slices. Split along its own seams — bounded contexts,
  subsystems, or layers — not by file count.
- Every reviewable source file belongs to exactly one slice. Do not create overlapping slices;
  overlap produces duplicate findings.
- Exclude code that is not hand-written: generated output, vendored snapshots, lockfiles,
  fixtures. Name these exclusions in the plan so the omission is visible and deliberate.

For each slice record: a short name, the directories and files it contains, and a one-line
statement of what the slice is responsible for.

## 2. Get approval

Present the slice plan to the user before launching anything: the slices, what each covers,
what is excluded and why, and how many parallel subagents will run. Each subagent is a
full frontier-model review, so the user must approve the plan first. Do not start reviewers
until the user approves. Apply any changes the user requests to the plan.

## 3. Launch reviewers

Launch one `architecture-reviewer` subagent per slice. Launch them all in parallel in a
single batch; there is no dependency between slices.

Each prompt must give the subagent:

- The slice: its directories and files, and its one-line responsibility.
- The instruction to focus its review on this slice only. It may — and should — read outside
  the slice to gather context (callers, contracts, project instructions), but findings must be
  about the slice it was given.
- Any repository conventions that change how the slice should be judged (for example, which
  files are generated and must be traced to their generator rather than reviewed as
  hand-written).

Do not restate the rubric or the output format. The subagent's own prompt already includes the
`engineering-principles` Skill and the finding format (priority P0–P3, title, summary,
violating principle, code citations).

## 4. Merge findings

Collect every subagent report. Then:

- Deduplicate. Two slices can observe the same root cause from different sides; merge such
  findings into one, keeping both sets of citations.
- Verify spot-checks. If a finding looks doubtful, read the cited code yourself before
  recording it. Do not silently drop a finding; if you reject one, note it and the reason.
- Sort the surviving findings by priority.

Present the report to the user. Give them a briefing—how healthy is the architecture overall and what are the areas of concern. Don't proceed until the user reviews and approves the report.

## 5. Create the Linear project

Create a new Linear project:

- Title: `Architectural Review M/DD` using today's date without leading zeros
  (for example `Architectural Review 8/30`).
- Project label: `Health Check`.
- Description: an overview of the findings — the slices reviewed, the count of findings by
  priority, and the main themes with one line each. Keep it a readable summary, not a copy of
  every ticket.

## 6. File tickets

File one Linear ticket per surviving finding, inside the new project. Follow any
repository-specific ticket-filing rules in the project's instructions; where they exist, they
take precedence over the defaults below.

Each ticket must have:

- A clear title and a summary that states the violating principle and includes the code
  citations from the review.
- A `Complete when` (or `Acceptance Criteria`) section that states the observable condition
  that closes the ticket.
- Labels. Fetch the team's available labels from Linear first and choose from them; do not
  invent labels.
- A priority. Map the reviewer's P0–P3 to Linear Urgent/High/Medium/Low as the starting
  recommendation.
- An estimate, T-shirt sized.

When all tickets are filed, report back: a link to the project, the ticket count by priority,
and any findings you rejected during the merge and why.
