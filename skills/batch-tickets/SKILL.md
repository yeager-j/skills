---
name: batch-tickets
description: Work on a large number of Linear tickets at once.
---
# Batch Tickets

You are given a set of Linear tickets to work on. Use subagents to accomplish these tasks, acting as a coordinator. By default, use the `implementor` or `implementor-lite` subagent unless the user specifies a different model for subagents.

## Workflow

1. Fetch all the provided Linear tickets. Check each ticket for comments as well.
2. Decide whether exploration adds value. If so, use the `/larp-explore` Skill.
3. Once you have a complete understanding of the ticket series, decide which tickets should be batched together.
4. Present the batch list to the user, who will either approve or suggest changes.
5. Once the user approves, send the first task to a subagent, providing facts uncovered during exploration and the ticket details.
6. When the subagent is finished, review the changes and make sure each of the task's acceptance criteria are met.
7. Open a PR.

When submitting work, use the GitHub PR stacking API (`gh stack`). Once every PR on the stack is complete and green, use `watch-prs` to check for Codex reviews.

## How to Batch Tickets

Two or more tickets can be batched if any of these are true:
1. They touch the same files
2. They share an implementation seam
3. They are all small and mechanical
