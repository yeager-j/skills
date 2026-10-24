---
name: implement-linear-ticket
description: Implement Linear tickets end to end. Use when asked to implement a Linear ticket or continue its PR after review feedback or CI failure.
when_to_use: When user requests implementation of a Linear ticket.
---

# Implement Linear Ticket

1. Read the Linear ticket, every comment, and its parent ticket chain before touching code.
2. Decide whether subagent exploration adds value. Read files directly cited by the ticket first.
3. State the ticket's end-goal. Do not just re-state acceptance criteria. Think big-picture; why does the ticket exist and what does closing it look like?
4. Synthesize the evidence, then begin implementation. A ticket's proposed plan or acceptance section is input, not the contract. Do not implement until the seams, API, invariants, invalid cases, acceptance matrix, and verification commands are decided.
5. Run the repository's full test suite and require it to pass before publishing the PR; do not publish while it is failing.
6. Use LARP to review the diff with `larp review --message "Review the diff"`. Verify findings and fix before opening a PR. Do not run if the task was just doc edits or other non-code changes.
7. Commit, push, and open a ready (non-draft) PR. Put Linear's magic words in the PR body, in addition to a summary of the changes, for every ticket being closed, using the exact form `Closes <ticket id>` (for example, `Closes SDK-10`). For multiple tickets, use a single sentence such as `Closes SDK-10, SDK-11, and SDK-12`. Completion requires the PR to exist.

Create a new branch from the Linear ticket's prescribed branch name.

Use the `/watch-prs` Skill to await Codex's review of the PR. If it requests any changes, evaluate them first before implementing and updating the PR.

## Planning

When planning, use the `/planning` Skill.

## Implementation

When starting implementation, use the `/code-style` Skill.

## Submitting a PR

You can use the `/watch-prs` Skill to get notified when Codex completes its review. Run the script in the background without polling; it'll exit when the review is complete.

Verify and fix all valid findings, then respond to and close Codex's inline comments.

## LARP vs Codex Reviews

When you submit a PR, OpenAI's code review model hunts for defects in the code, but it does not think big-picture or question assumptions. If you want the *architecture* or *assumptions* validated, reach for `/larp-agent`. If a task is straightforward and contains little ambiguity, you can skip LARP. The user pays for Claude and Codex via subscription, so a LARP review does not cost anything.